<script>
	import * as d3 from'd3';
	import Axis from './Axis.svelte';

	export let dataset;
	export let Groupby;

	//dimensions
	let borderBoxSize;

	$: width = borderBoxSize
		? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
		:400;

	$: height = borderBoxSize
		? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
		:400;

	const margin = {top: 20, right: 20, bottom: 50, left: 60};

	//get count of each group
	$: groupcounts = d3.rollup(
		dataset, 
		(g) => g.length, 
		(d) => d[Groupby]
		);
	
	//scales
	$: maxCount = d3.max(Array.from(groupcounts.values()));

	$: x = d3
		   .scaleLinear()
		   .domain([0, maxCount])
		   .nice()
		   .range([margin.left, width - margin.right]);
	$: y = d3
		   .scaleBand()
		   .domain(Array.from(groupcounts.keys()))
		   .range([height - margin.bottom, margin.top])
		   .padding(0.1);

</script>

<div class="barchart" bind:borderBoxSize>
	<svg {height} {width}>
		<!-- bars -->
		<g>
			{#each Array.from(groupcounts.entries()) as [group, count]}
				<rect
					x={x(0)}
					y={y(group)}
					width={x(count) - x(0)}
					height={y.bandwidth()}
					fill="steelblue"
				/>
			{/each}
		</g>
		<!-- axes -->
		<Axis orientation="left" margin={margin} width={width} height={height} scale={y} label={Groupby} />
		<Axis orientation="bottom" margin={margin} width={width} height={height} scale={x} label="Count" />
	</svg>
</div>

<style>
	.barchart {
		/* center the barchart */
		display: flex;
		height: 100%;
		flex: 1;
	}
	
</style>