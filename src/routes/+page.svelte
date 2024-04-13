<script>
	import './style.css';
	import * as d3 from 'd3';
	import Scatterplot from './Scatterplot.svelte';
	import BarChart from './BarChart.svelte';
	import RaincloudPlot from './RaincloudPlot.svelte';
	import ScattercloudPlot from './ScattercloudPlot.svelte';
	import Grouped_chart from './Grouped_chart.svelte';
	import PlayerList from './PlayerList.svelte';
	import ColorLegend from './ColorLegend.svelte';
	import Circles from './circles.svelte';
	import { onMount } from 'svelte';

	const METRIC_STR = "Metric";

	// data comes from the load function in +page.js
	export let data;

	// indices of the brushed data points in the scatter plot
	let selectedIndices = [];

	// callback function to update selectedIndices when the scatterplot is brushed
	function onbrush(indices) {
		selectedIndices = indices;
	}

	let metrics = Array.from(new Set(data.dataset.map(d => d[METRIC_STR])));
	let metric2str = {
		AC: "accuracy",
		BR: "bugs resolved",
		BW: "bandwidth",
		C: "execution cost",
		EF: "energy efficiency",
		ER: "error rate",
		ET: "execution time",
		FL: "failure/error",
		IN: "interference",
		LT: "latency",
		MT: "memory traffic",
		OV: "overhead",
		PR: "price",
		SP: "space",
		TH: "throughput",
		WA: "waste",
		/* TODO : found these in the dataset, not sure of meaning */
		EN: "energy cost",
		LOC: "lines of code",
		IPC: "instructions per cycle",
		AV: "availability",
	};

	let metric2indices = d3
		.rollup(
			data.dataset.map((d, i) => [d[METRIC_STR], i]),
			g => g.map(([metric, i]) => i),
			([metric, i]) => metric);

	// // data point that is highlighted in the list
	// let highlightedPlayer = null;

	// callback function to update highlightedPlayer when the list is hovered over
	function onhover(player) {
		highlightedPlayer = player;
	}

	// get the unique categories in the dataset sorted by count
	// $: categories = d3
	// 	.groupSort(
	// 		data.dataset,
	// 		(g) => g.length,
	// 		(d) => d[colorFeature]
	// 	)
	// 	.reverse();

	// $: color = d3.scaleOrdinal().domain(categories).range(d3.schemeTableau10);
</script>

<h1 class="title">Exploring Trends and Patterns in Systems Research</h1>

<div class="container">
	<div class="header1">
		<h2>What metric does the author use more frequently in Sytems Research to get Published?</h2>
		<BarChart dataset={data.dataset}/>
	</div>
</div>

<div class="container">
	<div class="header1">
		<h2>Does the choice of metric depends on which Conference we are targetting?</h2>
		<Grouped_chart dataset={data.dataset}/>
	</div>
</div>

<div class=raincloud-container>
	<!-- TODO: Make these form a grid -->
	<h2> Measure distribution for each metric</h2>
	<div class=rain-clouds>
		{#each metrics as metric}
			{#if metric2indices.get(metric).length > 10}
				<div class=rain>
					<RaincloudPlot dataset={data.dataset} feature="Measure" filteredIndices={metric2indices.get(metric)} color="steelblue" buckets=15 axisLabel={metric2str[metric]} />
				</div>
			{/if}
		{/each}
	</div>
</div>

<div class="container">
	<div class="header1">
		<h2>How do category compare to each other?</h2>
		<Circles/>
	</div>
</div>

<div class=scattercloud-container>
	<ScattercloudPlot dataset={data.dataset} xFeature="Measure" yFeature="Citation" color=steelblue buckets=15 />
</div>

<style>
	.container {
		/* set the font */
		font-family: system-ui, sans-serif;
		font-size: 16px;
		/* make the div take up the entire screen */
		height: 100vh;
		width: 100vw;
		/* add 32px of padding around the div */
		padding: 2em;
		/* put the controls on top of the plots with 32px of space in between */
		display: flex;
		
		gap: 2em;
	}
	/* Define margin-top for raincontainer */
	.raincloud-container {
		margin-top: 2em; /* Adjust as needed */
	}
	.rain-clouds {
		display: flex;
  		flex-wrap: wrap;
  		justify-content: space-between;
  		width: 100%;
	}

	.rain {
  		width: 100%;
  		height: 400px;
  		/* background-color: blue; */
  		margin-bottom: 15px;
	}

	/* place the feature controls and color legend next to each other */
	/* .header {
		display: flex;
		gap: 2em;
		align-items: center;
	}*/

	.header1 {
		/* center the barchart */
		/* center the barchart */
		justify-content: center;
		align-items: center;
		/* display: flex; */
		height: 100%;
		flex: 1;
	} 

	/* styles.css */
	.title {
		font-family: 'Arial', sans-serif; /* Use a suitable font-family */
		font-size: 36px;
		color: #3b1195; /* Change to your desired color */
		text-align: center;
		margin-bottom: 30px;
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2); /* Add a subtle text shadow */
	}

	.title::before {
		content: '';
		display: block;
		width: 100%;
		height: 2px; /* Thickness of the underline */
		background-color: #FFD166; /* Change to your desired underline color */
		margin-bottom: 10px; /* Adjust as needed */
	}

	.title::after {
		content: '';
		display: block;
		width: 75%; /* Width of the decorative line */
		height: 2px; /* Thickness of the decorative line */
		background-color: #EF476F; /* Change to your desired color */
		margin: 10px auto 0; /* Adjust margin as needed */
	}

	
</style>
