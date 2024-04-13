<script>
	import * as d3 from 'd3';
	import Axis from './Axis.svelte';

	export let dataset;
	export let xFeature;
	export let yFeature;
	export let filteredIndices = d3.range(0, dataset.length);
	// export let colorFeature;
	export let color;
	// export let highlightedPlayer;
	// export let onbrush;
	export let buckets;

	/* Copied directly from: https://stackoverflow.com/a/45804710/6573510 */
	function filterOutliers(someArray) {
		if(someArray.length < 4)
			return someArray;

		let values, q1, q3, iqr, maxValue, minValue;

		values = someArray.slice().sort( (a, b) => a - b);//copy array fast and sort

		if((values.length / 4) % 1 === 0){//find quartiles
			q1 = 1/2 * (values[(values.length / 4)] + values[(values.length / 4) + 1]);
			q3 = 1/2 * (values[(values.length * (3 / 4))] + values[(values.length * (3 / 4)) + 1]);
		} else {
			q1 = values[Math.floor(values.length / 4 + 1)];
			q3 = values[Math.ceil(values.length * (3 / 4) + 1)];
		}

		iqr = q3 - q1;
		maxValue = q3 + iqr * 1.5;
		minValue = q1 - iqr * 1.5;

		return values.filter((x) => (x >= minValue) && (x <= maxValue));
	}

	// dimensions

	let borderBoxSize;

	$: width = borderBoxSize
		? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
		: 400;

	$: height = borderBoxSize
		? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
		: 400;

	const margin = { top: 35, right: 20, bottom: 50, left: 60 };
	const maxElevCloudFrac = 0.15; // max percentage of space for cloud
	const maxElevBoxFrac = 0.05; // max percentage of space for boxplot

	$: innerHeight = height - margin.top - margin.bottom;
	$: innerWidth = width - margin.left - margin.right;
	$: cloudSize = d3.min([innerHeight, innerWidth].map(v => Math.round(v * maxElevCloudFrac)));
	$: boxSize = d3.min([innerHeight, innerWidth].map(v => Math.round(v * maxElevBoxFrac)));
	$: scatterplotSize =
		{ height: innerHeight - cloudSize - boxSize,
		  width: innerWidth - cloudSize - boxSize};

	$: scatterMargins =
		{ left: margin.left, bottom : margin.bottom,
		  top: margin.top + cloudSize + boxSize,
		  right: margin.right + cloudSize + boxSize};
	$: topCloudMargin =
		{ top: margin.top, left: margin.left,
		  bottom: margin.bottom + scatterplotSize.height + boxSize,
		  right: margin.right + cloudSize + boxSize};
	$: topBoxMargin =
		{ left: margin.left,
		  top: margin.top + cloudSize,
		  bottom: margin.bottom + scatterplotSize.height,
		  right: margin.right + cloudSize + boxSize};
	$: rightCloudMargin =
		{ right: margin.right, bottom: margin.bottom,
		  left: margin.left + scatterplotSize.width + boxSize,
		  top: margin.top + cloudSize + boxSize};
	$: rightBoxMargin =
		{ bottom: margin.bottom,
		  right: margin.right + cloudSize, 
		  left: margin.left + scatterplotSize.width,
		  top: margin.top + cloudSize + boxSize};

	// Get the values that will be displayed
	let filteredDataset = filteredIndices.map((i) => dataset[i]);
	let xvalues = filteredDataset.map(d => d[xFeature]);
	let yvalues = filteredDataset.map(d => d[yFeature]);

	// X values
	// Compute histogram of values mapping value in middle of bucket value to count
	// Figure out size of buckets based on non-outliers and number of buckets
	let [xvisLowerBound, xvisUpperBound] = d3.extent(filterOutliers(xvalues));
	let [xlowerBound, xupperBound] = d3.extent(xvalues);
	let xbucketSize = (xvisUpperBound - xvisLowerBound) / (buckets - 2);
	let xhist = d3
		.bin()
		.thresholds(d3.range(xlowerBound - xbucketSize, xupperBound + xbucketSize + 1, xbucketSize))
		(xvalues)
		.map(bin => [(bin.x0 + bin.x1) / 2.0, bin.length]);
	// Add one more bucket to start and end
	xhist.unshift([xhist[0][0] - xbucketSize,0])
	xhist.push([xhist[xhist.length - 1][0] + xbucketSize,0]);
	let xcountExtent = d3.extent(xhist.map(d => d[1]));

	$: xvalueScaleRange = [xvisLowerBound, xvisUpperBound];
	$: xvaluesScale = d3
		.scaleLinear()
		.domain(xvalueScaleRange)
		.range([scatterMargins.left, width - scatterMargins.right]);
	$: xvaluesScaleInv = d3
		.scaleLinear()
		.domain([scatterMargins.left, width - scatterMargins.right])
		.range(xvalueScaleRange);
	$: xcountScale = d3
		.scaleLinear()
		.domain(xcountExtent)
		.range([height - topCloudMargin.bottom, topCloudMargin.top]);
	$: xarea = d3.area().curve(d3.curveBasis)
		.y0(([bucket,count]) => xcountScale(0))
		.y1(([bucket,count]) => xcountScale(count))
		.x(([bucket,count]) => xvaluesScale(bucket))
	$: xboxScale = d3
		.scaleLinear()
		.domain([0,1])
		.range([height - topBoxMargin.bottom, topBoxMargin.top]);
	$: xQ10 = d3.quantile(xvalues, 0.1);
	$: xQ25 = d3.quantile(xvalues, 0.25);
	$: xQ50 = d3.quantile(xvalues, 0.5);
	$: xQ75 = d3.quantile(xvalues, 0.75);
	$: xQ90 = d3.quantile(xvalues, 0.9);

	// Y values
	// Compute histogram of values mapping value in middle of bucket value to count
	// Figure out size of buckets based on non-outliers and number of buckets
	let [yvisLowerBound, yvisUpperBound] = d3.extent(filterOutliers(yvalues));
	let [ylowerBound, yupperBound] = d3.extent(yvalues);
	let ybucketSize = (yvisUpperBound - yvisLowerBound) / (buckets - 2);
	let yhist = d3
		.bin()
		.thresholds(d3.range(ylowerBound - ybucketSize, yupperBound + ybucketSize + 1, ybucketSize))
		(yvalues)
		.map(bin => [(bin.x0 + bin.x1) / 2.0, bin.length]);
	// Add one more bucket to start and end
	yhist.unshift([yhist[0][0] - ybucketSize,0])
	yhist.push([yhist[yhist.length - 1][0] + ybucketSize,0]);
	let ycountExtent = d3.extent(yhist.map(d => d[1]));

	$: yvalueScaleRange = [yvisLowerBound, yvisUpperBound];
	$: yvaluesScale = d3
		.scaleLinear()
		.domain(yvalueScaleRange)
		.range([height - scatterMargins.bottom, scatterMargins.top]);
	$: yvaluesScaleInv = d3
		.scaleLinear()
		.domain([height - scatterMargins.bottom, scatterMargins.top])
		.range(yvalueScaleRange);
	$: ycountScale = d3
		.scaleLinear()
		.domain(ycountExtent)
		.range([rightCloudMargin.left, width - rightCloudMargin.right]);
	$: yarea = d3.area().curve(d3.curveBasis)
		.x0(([bucket,count]) => ycountScale(0))
		.x1(([bucket,count]) => ycountScale(count))
		.y(([bucket,count]) => yvaluesScale(bucket));
	$: yboxScale = d3
		.scaleLinear()
		.domain([0,1])
		.range([rightBoxMargin.left, width - rightBoxMargin.right]);
	$: yQ10 = d3.quantile(yvalues, 0.1);
	$: yQ25 = d3.quantile(yvalues, 0.25);
	$: yQ50 = d3.quantile(yvalues, 0.5);
	$: yQ75 = d3.quantile(yvalues, 0.75);
	$: yQ90 = d3.quantile(yvalues, 0.9);

	let raincloud;
	function onDrag(event) {
		// CHANGE X
		let dVal = (xvaluesScaleInv(event.dx) - xvaluesScaleInv(0)) * 8 /* scalar */;
		if (xvalueScaleRange[0] - dVal < xlowerBound) {
			xvalueScaleRange = [xlowerBound, xlowerBound + (xvisUpperBound - xvisLowerBound)];
    	} else if (xvalueScaleRange[1] - dVal > xupperBound) {
			xvalueScaleRange = [xupperBound - (xvisUpperBound - xvisLowerBound), xupperBound];
    	} else {
			xvalueScaleRange = xvalueScaleRange.map(v => v - dVal);
		}

		// CHANGE Y
		dVal = (yvaluesScaleInv(event.dy) - yvaluesScaleInv(0)) * 8 /* scalar */;
		if (yvalueScaleRange[0] - dVal < ylowerBound) {
 			  yvalueScaleRange = [ylowerBound, ylowerBound + (yvisUpperBound - yvisLowerBound)];
    	} else if (yvalueScaleRange[1] - dVal > yupperBound) {
    		  yvalueScaleRange = [yupperBound - (yvisUpperBound - yvisLowerBound), yupperBound];
    	} else {
			yvalueScaleRange = yvalueScaleRange.map(v => v - dVal);
		}
	}
	const drag = d3.drag().on('drag', onDrag)
	$: d3.select(raincloud).call(drag)
</script>

<div class="rainscatterplot" bind:this={raincloud} bind:borderBoxSize>
	<svg {height} {width} >
		<!-- circles -->
		<g class="circles">
			{#each dataset as d}
				<!-- <circle cx={x(d[xFeature])} cy={y(d[yFeature])} fill={color(d[colorFeature])} r={3} /> -->
				<circle cx={xvaluesScale(d[xFeature])} cy={yvaluesScale(d[yFeature])} fill={color} r={3} />
			{/each}
		</g>

		<g class="clouds">
			/* TODO : use luminance to encode distance from mean (kind of like choropleth map)
			remember to check charts and channels! */
			<defs>
				<linearGradient id="xgradient">
				  <stop offset="0%" stop-color="#ef8a62" />
				  <stop offset="5%" stop-color="#000000" />
				  <stop offset="100%" stop-color="#67a9cf" />
				</linearGradient>
			 </defs>
			<defs>
				<linearGradient id="ygradient" gradientTransform="rotate(-90 1 0)">
				  <stop offset="0%" stop-color="#ef8a62" />
				  <stop offset="5%" stop-color="#000000" />
				  <stop offset="100%" stop-color="#67a9cf" />
				</linearGradient>
			</defs>
			<path d={xarea(xhist)} fill="url(#xgradient)" stroke-width=0.01em stroke=black/>
			<path d={yarea(yhist)} fill="url(#ygradient)" stroke-width=0.01em stroke=black/>
		</g>

		<g class="boxplot">
			<rect x={xvaluesScale(xQ25)} y={xboxScale(0.8)} height={Math.abs(xboxScale(0.8) - xboxScale(0.2))} width={Math.abs(xvaluesScale(xQ75) - xvaluesScale(xQ25))} stroke-width=0.1em stroke=black fill=transparent/>
			<line x1={xvaluesScale(xQ50)} x2={xvaluesScale(xQ50)} y1={xboxScale(0.2)} y2={xboxScale(0.8)} stroke-width=0.1em stroke=black />
			<line x1={xvaluesScale(xQ10)} x2={xvaluesScale(xQ25)} y1={xboxScale(0.5)} y2={xboxScale(0.5)} stroke-width=0.11em stroke=black />
			<line x1={xvaluesScale(xQ75)} x2={xvaluesScale(xQ90)} y1={xboxScale(0.5)} y2={xboxScale(0.5)} stroke-width=0.1em stroke=black />

			<rect y={yvaluesScale(yQ75)} x={yboxScale(0.2)} width={Math.abs(yboxScale(0.8) - yboxScale(0.2))} height={Math.abs(yvaluesScale(yQ75) - yvaluesScale(yQ25))} stroke=black fill=transparent/>
			<line y1={yvaluesScale(yQ50)} y2={yvaluesScale(yQ50)} x1={yboxScale(0.2)} x2={yboxScale(0.8)} stroke=black />
			<line y1={yvaluesScale(yQ10)} y2={yvaluesScale(yQ25)} x1={yboxScale(0.5)} x2={yboxScale(0.5)} stroke=black />
			<line y1={yvaluesScale(yQ75)} y2={yvaluesScale(yQ90)} x1={yboxScale(0.5)} x2={yboxScale(0.5)} stroke=black />
		</g>

		<!-- axes -->
		<g class="axes">
			<Axis orientation="bottom" scale={xvaluesScale} {width} {height} {margin} label={xFeature} />
			<Axis orientation="left" scale={yvaluesScale} {width} {height} {margin} label={yFeature} />
		</g>
	</svg>
</div>


<style>
	.rainscatterplot {
		/* take up extra horizontal space in the parent */
		flex: 1;
		/* be as tall as the parent div */
		height: 100%;
	}

	/* animate circles to their new location */
/*	circle {
		transition:
			cx 250ms,
			cy 250ms;
	}
*/
</style>
