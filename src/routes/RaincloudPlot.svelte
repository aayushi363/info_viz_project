<!-- Developed based on https://gist.github.com/vijithassar/c60dafea4431f292660d6f5e0487e470 -->

<script>
	import * as d3 from 'd3';
	import Axis from './Axis.svelte';
	import { onMount } from 'svelte';

	export let dataset;
	export let feature;
	export let filteredIndices;
	export let buckets;
	export let orientation = "up";
	export let axisLabel;
	export let color;

	// let cloudColorList = [ /*"#ffffff",*/ "#f0f0f0", "#d9d9d9", "#bdbdbd", "#969696", "#737373"/*, "#525252","#252525", '#000000'*/].reverse();
	// let rainColorList = [ /*"#f7fbff", "#deebf7", "#c6dbef", "#9ecae1", "#6baed6", "#4292c6", "#2171b5",*/ "#08519c"/*, "#08306b" */].reverse();

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

	// $: width = borderBoxSize
	// 	? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
	// 	: 400;

	// $: height = borderBoxSize
	// 	? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
	// 	: 200;

	$: width = borderBoxSize
		?borderBoxSize[0].inlineSize : 400;

	$: height = borderBoxSize
		? borderBoxSize[0].blockSize : 200;

	const margin = { top: 20, right: 20, bottom: 30, left: 20 };
	const elevCloudFrac = 0.6; // percentage of space for cloud
	const elevBoxFrac = 0.1; // percentage of space for boxplot
	const elevRainFrac = 1.0 - elevCloudFrac - elevBoxFrac; // percentage of space for rain
	// const rainExtraMargin = {up : 0, down: 0};
	const boxplotHeightFrac = 0.05; // height of boxplot relative to container

	$: innerHeight = height - margin.top - margin.bottom;
	$: innerWidth = width - margin.left - margin.right;

	$: cloudMargin = orientation === "up"
		? { right: margin.right, left: margin.left,
			top: margin.top + 0,
			bottom: margin.bottom + Math.round(innerHeight * (1.0 - elevCloudFrac)) }
		: { top: margin.top, bottom: margin.bottom,
			right: margin.right + 0,
			left: margin.left + Math.round(innerWidth * (1.0 - elevCloudFrac)) }
	$: boxMargin = orientation === "up"
		? { right: margin.right, left: margin.left,
			top: margin.top + Math.round(innerHeight * elevCloudFrac),
			bottom: margin.bottom + Math.round(innerHeight * elevRainFrac) }
		: { top: margin.top, bottom: margin.bottom,
			right: margin.right + Math.round(innerWidth * elevCloudFrac),
			left: margin.left + Math.round(innerWidth * elevRainFrac) };
	$: rainMargin = orientation === "up"
		? { right: margin.right, left: margin.left,
			top: margin.top + Math.round(innerHeight * (1 - elevRainFrac)),
			bottom: margin.bottom + 0 }
		: { top: margin.top, bottom: margin.bottom,
			right: margin.right + Math.round(innerWidth * (1 - elevRainFrac)),
			left: margin.left + 0 };

	// Get the values that will be displayed
	let filteredDataset = filteredIndices.map((i) => dataset[i]);
	let values = filteredDataset.map(d => d[feature]);
	let Q50 = d3.quantile(values, 0.5);

	// Compute histogram of values mapping value in middle of bucket value to count
	// Figure out size of buckets based on non-outliers and number of buckets
	let [visLowerBound, visUpperBound] = d3.extent(filterOutliers(values));
	let [lowerBound, upperBound] = d3.extent(values);
	let bucketSize = (visUpperBound - visLowerBound) / (buckets - 2);
	// Compute the buckets so that the mean is at the middle of a bucket.
	// I'm not able to get the cloud coloring correct, so this is the next best thing.
	// by splitting the cloud into two, I can color each piece to achieve.
	// let histDomain = d3.range(
	// 	Q50 - 0.5 * bucketSize > lowerBound
	// 	? Q50 - (Math.ceil((Q50 - 0.5 * bucketSize - lowerBound) / bucketSize) + 0.5) * bucketSize
	// 	: 2 * Q50 - lowerBound - bucketSize,
	// 	upperBound + bucketSize + 1, bucketSize);
	let hist = d3
		.bin()
		.thresholds(d3.range(lowerBound - bucketSize, upperBound + bucketSize + 1, bucketSize))
		(values)
	// if (! (histDomain, Q50 - 0.5 * bucketSize > lowerBound)) {
	// 	$: console.log("HIST", axisLabel, hist, "Q50", Q50, "lowerBound", lowerBound, "bucketSize", bucketSize, "histDomain", histDomain);
	// }
		.map(bin => [(bin.x0 + bin.x1) / 2.0, bin.length]);
	// Add one more bucket to start and end
	hist.unshift([hist[0][0] - bucketSize,0])
	hist.push([hist[hist.length - 1][0] + bucketSize,0]);

	$: histExtent = d3.extent(hist.map(d => d[0]));
	$: console.log('HIST', axisLabel, histExtent)

	$: valueScaleRange = [visLowerBound, visUpperBound];

	// This scale is shared by all plots
	$: valuesScale = d3
		.scaleLinear()
		.domain(valueScaleRange)
		.range(orientation === "up"
			? [margin.left, width - margin.right]
			: [height - margin.bottom, margin.top]);

	$: valuesScaleInv = d3
		.scaleLinear()
		.domain(orientation === "up"
			? [margin.left, width - margin.right]
			: [height - margin.bottom, margin.top])
		.range(valueScaleRange);

	// Used for creating clouds
	$: countScale = d3
		.scaleLinear()
		// The domain is the sum of counts so height corresponds with percentage
		.domain([0,d3.sum(hist.map(d => d[1]))])
		.range(orientation === "up"
				? [height - cloudMargin.bottom, cloudMargin.top]
				: [cloudMargin.left, width - cloudMargin.right]);
	
	// Actually create cloud
	$: area = d3.area().curve(d3.curveBasis);
	$: if (orientation === "up") {
		area = area
		.y0(([bucket,count]) => countScale(0))
		.y1(([bucket,count]) => countScale(count))
      	.x(([bucket,count]) => valuesScale(bucket))
	} else {
		area = area
		.x0(([bucket,count]) => countScale(0))
		.x1(([bucket,count]) => countScale(count))
      	.y(([bucket,count]) => valuesScale(bucket))
	}

	// Used for creating rain
	$: elevationScale = d3
		.scaleLinear()
		.domain([0,1])
		.range(orientation === "up"
				? [height - rainMargin.bottom, rainMargin.top]
				: [rainMargin.left, width - rainMargin.right]);

	// Used for creating boxplot
	$: boxScale = d3
		.scaleLinear()
		.domain([0,1])
		.range(orientation === "up"
				? [height - boxMargin.bottom, boxMargin.top]
				: [boxMargin.left, width - boxMargin.right]);
	$: Q10 = d3.quantile(values, 0.1);
	$: Q25 = d3.quantile(values, 0.25);
	$: Q75 = d3.quantile(values, 0.75);
	$: Q90 = d3.quantile(values, 0.9);

	let raincloud;
	function onDrag(event) {
		let dPos = orientation === "up" ? event.dx : event.dy;
		let dVal = (valuesScaleInv(dPos) - valuesScaleInv(0)) * 8 /* scalar */;
		if (valueScaleRange[0] - dVal < lowerBound) {
 			  valueScaleRange = [lowerBound, lowerBound + (visUpperBound - visLowerBound)];
    	} else if (valueScaleRange[1] - dVal > upperBound) {
    		  valueScaleRange = [upperBound - (visUpperBound - visLowerBound), upperBound];
    	} else {
			valueScaleRange = valueScaleRange.map(v => v - dVal);
		}
	}
	const drag = d3.drag().on('drag', onDrag)
	$: d3.select(raincloud).call(drag)

	onMount(() => {
		if (orientation !== "up") {
			d3.select("linearGradient")
				.attr("gradientTransform", "rotate(-90 1 0)");
		}
	});

	let circleRadius = 5;

	let tooltip;
	let raindrop;
	let raindropCX;
	let raindropCY;
	let tooltipContent = "";
	function showTooltip(event, i) {
        tooltipContent = filteredDataset[i]["Title"];

		let px = valuesScale(values[i]);
		let py = elevationScale(rainElevations[i]);

		raindropCX = px;
		raindropCY = py;
		tooltip.style.visibility = "visible";
		tooltip.style.left = `${px + 10}px`;
		tooltip.style.top = `${py + 10}px`;
		d3.select(event.target)
			.attr("r", 2 * circleRadius);
		raindrop.style.visibility = "visible";
    }
    function hideTooltip(event, i) {
		tooltip.style.visibility = "hidden";
		raindrop.style.visibility = "hidden";
		d3.select(event.target)
			.attr("r", circleRadius);
    }

	let rainElevations = values.map(d => Math.random());

	// $: rainColorScale = d3.scaleSequential()
	// 	.domain([lowerBound, upperBound])
	// 	.interpolator(d3.interpolateRgbBasis(rainColorList));
</script>

<div class="raincloud" bind:this={raincloud} bind:borderBoxSize>
	<h2 class="title"> {axisLabel} </h2>
	<div bind:this={tooltip} class="tooltip" style="visibility:hidden">{tooltipContent}</div>
	<svg {height} {width}>
		<!-- {#key valueScaleRange} -->
			<g class="cloud">
				<!-- TODO : use luminance to encode distance from mean (kind of like choropleth map)
				remember to check charts and channels! -->
				<!-- <defs>
					<linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="0%">
						<stop offset="0%" stop-color={cloudColorList[0]}/>
						{#each cloudColorList as color, i}
							{#if i > 0 && i < cloudColorList.length - 1}
								<stop offset={`${100 / cloudColorList.length * i}%`} stop-color={color}/>
							{/if}
						{/each}
						<stop offset="100%" stop-color={cloudColorList[cloudColorList.length - 1]}/>
					</linearGradient>
				</defs> -->
				<!-- Skew the below filter checks by some small fraction of the bucket size to account for numerical imprecision -->
				<path d={area(hist)} fill={color} stroke-width=0.5px stroke=black/>
			</g>

			<g class="rain">
				{#if orientation === "up"}
					{#each values as value, i}
						<circle r={circleRadius} cx={valuesScale(value)} cy={elevationScale(rainElevations[i])} fill={color} fill-opacity={0.5} stroke=gray stroke-width=0.5px
								on:mouseenter={(event) => showTooltip(event, i)}
								on:mouseleave={(event) => hideTooltip(event, i)}/>
					{/each}
					<circle bind:this={raindrop} r={2 * circleRadius} cx={raindropCX} cy={raindropCY} fill=none stroke=black stroke-width=3px style="visibility:hidden"/>
				{:else}
					{#each values as value}
						<circle r=2 cx={elevationScale(Math.random())} cy={valuesScale(value)} fill={"steelblue"}/>
					{/each}
				{/if}
			</g>

			<g class="boxplot">
				{#if orientation === "up"}
					<rect x={valuesScale(Q25)} y={boxScale(0.8)} height={Math.abs(boxScale(0.8) - boxScale(0.2))} width={Math.abs(valuesScale(Q75) - valuesScale(Q25))} stroke-width=0.1em stroke={color} stroke-opacity={0.75} fill=transparent/>
					<line x1={valuesScale(Q50)} x2={valuesScale(Q50)} y1={boxScale(0.2)} y2={boxScale(0.8)} stroke-width=0.1em stroke={color} stroke-opacity={0.75} />
					<line x1={valuesScale(Q10)} x2={valuesScale(Q25)} y1={boxScale(0.5)} y2={boxScale(0.5)} stroke-width=0.11em stroke={color} stroke-opacity={0.75} />
					<line x1={valuesScale(Q75)} x2={valuesScale(Q90)} y1={boxScale(0.5)} y2={boxScale(0.5)} stroke-width=0.1em stroke={color} stroke-opacity={0.75} />
				{:else}
					<rect y={valuesScale(Q75)} x={boxScale(0.2)} width={Math.abs(boxScale(0.8) - boxScale(0.2))} height={Math.abs(valuesScale(Q75) - valuesScale(Q25))} stroke={color} stroke-opacity={0.75} fill=transparent/>
					<line y1={valuesScale(Q50)} y2={valuesScale(Q50)} x1={boxScale(0.2)} x2={boxScale(0.8)} stroke={color} stroke-opacity={0.75} />
					<line y1={valuesScale(Q10)} y2={valuesScale(Q25)} x1={boxScale(0.5)} x2={boxScale(0.5)} stroke={color} stroke-opacity={0.75} />
					<line y1={valuesScale(Q75)} y2={valuesScale(Q90)} x1={boxScale(0.5)} x2={boxScale(0.5)} stroke={color} stroke-opacity={0.75} />
				{/if}

				<!-- axes -->

				<Axis orientation={orientation === "up" ? "bottom" : "left"} scale={valuesScale} {width} {height} {margin} color="gray" />
				<!-- <Axis orientation={orientation === "up" ? "left" : "bottom"} scale={valuesScale} {width} {height} {margin} label={axisLabel} /> -->
			</g>
		<!-- {/key} -->
	</svg>
</div>

<style>
    .raincloud {
        /* Set dimensions for each raincloud */
        width: calc(25% - 20px); /* Subtract margin */
        height: 16vw; /* Subtract margin */
		/* height: 200px; */
        margin: 10px; /* Add margin between rainclouds */
        float: left; /* Arrange rainclouds in a grid */
		position: relative;
        cursor: move;
		border: 1px solid rgb(255, 255, 255);
    }

	.title {
		position: absolute;
		top: 10px;
		left: 10px;
		color: rgb(100, 100, 100);
	}

    /* Adjust SVG dimensions */
    svg {
        width: 100%;
        height: 100%;
		z-index: 0;
    }

	.tooltip {
        position: absolute;
		width: 50%;
		left:10%;
		top: 10px;
        background-color: white;
        border: 1px solid black;
        padding: 5px;
        border-radius: 3px;
		z-index:5;
    }
</style>

