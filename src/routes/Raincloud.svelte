<!-- Developed based on https://gist.github.com/vijithassar/c60dafea4431f292660d6f5e0487e470 -->

<script>
	import * as d3 from 'd3';
	import Axis from './Axis.svelte';

	export let dataset;
	export let feature;
	export let filteredIndices;
	export let color;
	export let buckets;
	export let orientation = "up";
	export let axisLabel;

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

	const margin = { top: 25, right: 20, bottom: 50, left: 60 };
	const elevationCloudFrac = 0.7; // percentage of space for cloud (remainder for rain)
	const rainExtraMargin = {up : 20, down: 20};
	const boxplotHeightFrac = 0.05; // height of boxplot relative to container

	$: innerHeight = height - margin.top - margin.bottom;
	$: innerWidth = width - margin.left - margin.right;

	$: cloudMargin = orientation === "up"
		? { top: margin.top, right: margin.right, left: margin.left,
			bottom: margin.bottom + (innerHeight - Math.round(innerHeight * elevationCloudFrac)) }
		: { top: margin.top, right: margin.right, bottom: margin.bottom,
			left: margin.left + (innerWidth - Math.round(innerWidth * elevationCloudFrac)) }
	$: rainMargin = orientation === "up"
		? { right: margin.right, bottom: margin.bottom + rainExtraMargin.down, left: margin.left,
			top: margin.top + Math.round(innerHeight * elevationCloudFrac) + rainExtraMargin.up }
		: { top: margin.top, bottom: margin.bottom, left: margin.left + rainExtraMargin.down,
			right: margin.right + Math.round(innerWidth * elevationCloudFrac) + rainExtraMargin.up };

	// Get the values that will be displayed
	let values = filteredIndices.map((i) => dataset[i]).map(d => d[feature]);

	// Compute histogram of values mapping value in middle of bucket value to count
	// Figure out size of buckets based on non-outliers and number of buckets
	let [visLowerBound, visUpperBound] = d3.extent(filterOutliers(values));
	let [lowerBound, upperBound] = d3.extent(values);
	let bucketSize = (visUpperBound - visLowerBound) / (buckets - 2);
	let hist = d3
		.bin()
		.thresholds(d3.range(lowerBound - bucketSize, upperBound + 1, bucketSize))
		(values)
		.map(bin => [(bin.x0 + bin.x1) / 2.0, bin.length]);
	let countExtent = d3.extent(hist.map(d => d[1]));

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

	// Used for creating clouds by cloud
	$: countScale = d3
		.scaleLinear()
		.domain(countExtent)
		.range(orientation === "up"
				? [height - cloudMargin.bottom, cloudMargin.top]
				: [cloudMargin.left, width - cloudMargin.right]);
				
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
				? [countScale(0) - innerHeight * boxplotHeightFrac * 0.5, countScale(0) + innerHeight * boxplotHeightFrac * 0.5]
				: [countScale(0) - innerWidth * boxplotHeightFrac * 0.5, countScale(0) + innerWidth * boxplotHeightFrac * 0.5]);
	$: Q10 = d3.quantile(values, 0.1);
	$: Q25 = d3.quantile(values, 0.25);
	$: Q50 = d3.quantile(values, 0.5);
	$: Q75 = d3.quantile(values, 0.75);
	$: Q90 = d3.quantile(values, 0.9);

	let raincloud;
	function onDrag(event) {
		// let xp = Math.sign(valuesScaleInv(event.dx) - valuesScaleInv(0)) * Math.sqrt(Math.abs(valuesScaleInv(event.dx) - valuesScaleInv(0))) * 2;
		let xp = valuesScaleInv(event.dx) - valuesScaleInv(0);
		if (valueScaleRange[0] - xp < lowerBound) {
			valueScaleRange = [lowerBound, lowerBound + (visUpperBound - visLowerBound)];
		} else if (valueScaleRange[1] - xp > upperBound) {
			valueScaleRange = [upperBound - (visUpperBound - visLowerBound), upperBound];
		} else {
			valueScaleRange = valueScaleRange.map(v => v - xp);
		}
	}
	const drag = d3.drag().on('drag', onDrag)
	$: d3.select(raincloud).call(drag)
</script>

<div class="raincloud" bind:this={raincloud} bind:borderBoxSize>
	<svg {height} {width}>
		{#key valueScaleRange}
			<g class="cloud">
				<path d={area(hist)} fill={color} stroke-width=0.01em stroke=black/>
			</g>

			<g class="rain">
				{#if orientation === "up"}
					{#each values as value}
						<circle r=2 cx={valuesScale(value)} cy={elevationScale(Math.random())} fill={color}/>
					{/each}
				{:else}
					{#each values as value}
						<circle r=2 cx={elevationScale(Math.random())} cy={valuesScale(value)} fill={color}/>
					{/each}
				{/if}
			</g>

			<g class="boxplot">
				{#if orientation === "up"}
					<rect x={valuesScale(Q25)} y={boxScale(0)} height={Math.abs(boxScale(1) - boxScale(0))} width={Math.abs(valuesScale(Q75) - valuesScale(Q25))} stroke-width=0.1em stroke=black fill=transparent/>
					<line x1={valuesScale(Q50)} x2={valuesScale(Q50)} y1={boxScale(0)} y2={boxScale(1)} stroke-width=0.1em stroke=black />
					<line x1={valuesScale(Q10)} x2={valuesScale(Q25)} y1={boxScale(0.5)} y2={boxScale(0.5)} stroke-width=0.11em stroke=black />
					<line x1={valuesScale(Q75)} x2={valuesScale(Q90)} y1={boxScale(0.5)} y2={boxScale(0.5)} stroke-width=0.1em stroke=black />
				{:else}
					<rect y={valuesScale(Q75)} x={boxScale(0)} width={Math.abs(boxScale(1) - boxScale(0))} height={Math.abs(valuesScale(Q75) - valuesScale(Q25))} stroke=black fill=transparent/>
					<line y1={valuesScale(Q50)} y2={valuesScale(Q50)} x1={boxScale(0)} x2={boxScale(1)} stroke=black />
					<line y1={valuesScale(Q10)} y2={valuesScale(Q25)} x1={boxScale(0.5)} x2={boxScale(0.5)} stroke=black />
					<line y1={valuesScale(Q75)} y2={valuesScale(Q90)} x1={boxScale(0.5)} x2={boxScale(0.5)} stroke=black />
				{/if}

				<!-- axes -->

				<Axis orientation={orientation === "up" ? "bottom" : "left"} scale={valuesScale} {width} {height} {margin} label={axisLabel} />
		
				<!-- <Axis orientation={orientation === "up" ? "left" : "bottom"} scale={valuesScale} {width} {height} {margin} label={axisLabel} /> -->
			</g>
		{/key}
	</svg>
</div>

<style>
    .raincloud {
        /* Set dimensions for each raincloud */
        width: calc(33.33% - 20px); /* Subtract margin */
        height: calc(33.33% - 20px); /* Subtract margin */
        margin: 10px; /* Add margin between rainclouds */
        float: left; /* Arrange rainclouds in a grid */
        cursor: move;
		border: 1px solid black;

    }

    /* Adjust SVG dimensions */
    svg {
        width: 100%;
        height: 100%;
    }
</style>

