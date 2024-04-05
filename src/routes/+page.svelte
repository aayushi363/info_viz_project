<script>
	import './style.css';
	import * as d3 from 'd3';
	import Scatterplot from './Scatterplot.svelte';
	import FeatureControls from './FeatureControls.svelte';
	import BarChart from './BarChart.svelte';
	import PlayerList from './PlayerList.svelte';
	import ColorLegend from './ColorLegend.svelte';
	import { onMount } from 'svelte';

	// data comes from the load function in +page.js
	export let data;

	// default features to visualize
	let Groupby = 'None';

	// indices of the brushed data points in the scatter plot
	let selectedIndices = [];

	// callback function to update selectedIndices when the scatterplot is brushed
	function onbrush(indices) {
		selectedIndices = indices;
	}

	// data point that is highlighted in the list
	let highlightedPlayer = null;

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

<h1 class="title">What Performance is Reported by Published System Papers</h1>

<div class="container">
	<div class="header">
		<FeatureControls dataset={data.dataset} bind:Groupby />
	</div>
	<div class="main">
		<!-- <PlayerList dataset={data.dataset} {selectedIndices} {onhover} />
		<Scatterplot
			dataset={data.dataset}
			{xFeature}
			{yFeature}
			{colorFeature}
			{color}
			{highlightedPlayer}
			{onbrush}
		/> -->
		<BarChart dataset={data.dataset} bind:Groupby  />
	</div>
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
		flex-direction: column;
		gap: 2em;
	}

	/* place the feature controls and color legend next to each other */
	/* .header {
		display: flex;
		gap: 2em;
		align-items: center;
	}

	.main {
		/* make this div take up the rest of the container */
		/* flex: 1; */
		/* allow the div to shrink */
		/* min-height: 0; */
		/* place the children next to each other */
		/* display: flex; */
		/* add space between them */
		/* gap: 2em; */
	/* }  */

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
