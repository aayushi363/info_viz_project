<script>
	import './style.css';
	import * as d3 from 'd3';
	import Scatterplot from './Scatterplot.svelte';
	import BarChart from './BarChart.svelte';
	import RaincloudPlot from './RaincloudPlot.svelte';
	import ScattercloudPlot from './ScattercloudPlot.svelte';
	import Grouped_chart from './grouped_new.svelte';
	import PlayerList from './PlayerList.svelte';
	import ColorLegend from './ColorLegend.svelte';
	import Circles from './circles.svelte';
	import ConferenceMetric from './ConferenceMetric.svelte';
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

	// let raincloudColors = ["#a6cee3", "#1f78b4", "#b2df8a", "#33a02c", "#fb9a99", "#e31a1c", "#fdbf6f", "#ff7f00", "#cab2d6", "#6a3d9a", "#ffff99"];
	let raincloudColors = ["#8dd3c7", "#ffffb3", "#bebada", "#fb8072", "#80b1d3", "#fdb462", "#b3de69", "#fccde5", "#d9d9d9", "#bc80bd", "#ccebc5"];
	let raincloudMetrics = metrics.filter(m => metric2indices.get(m).length > 10).map((m,i) => [m,raincloudColors[i % raincloudColors.length]]);
</script>

<div class="introduction">
	<h1 class="title">Exploring how metrics and categories vary across conferences to understand the dynamics influencing Systems Research publications</h1>
	<p>
		Step into the fascinating world of Systems Research! Here, we delve into the big ideas that shape the field of Computer Science. Our webpage is your gateway to exploring the latest trends and intriguing patterns. We've meticulously collected data from three major conferences dedicated to Systems Research. Join us on this journey as we uncover the exciting developments and insights in this dynamic field.	</p>
</div>

<div class="container">
	<div class="header1">
		<h2>Let's dive into the realm of research and uncover the hottest themes that have captivated the minds of researchers, boasting the highest number of papers.
		</h2>
		<h2 style="text-align: center; color:darkred;">Domain Question: How do categories compare to each other?</h2>
		<h2 style="text-align: center; color:darkblue;">Data Question: Distribution of papers amongst 12 major categories</h2>

		<Circles/>
	</div>
</div>

<div class="container">
	<div class="header1">
		<h2>Let us understand the relative performance of different metrics presented at three distinct conferences. It's a comparison between conferences regarding the metrics they cover.
		</h2>
		<h2 style="text-align: center; color:darkred;">Domain question: How do metrics compare across three different conferences?</h2>
		<h2 style="text-align: center; color:darkblue;padding: 1.5%">Data Question: How do the values of various metrics published at SOSP, EuroSys and MICRO compare? </h2>

		<ConferenceMetric/>
	</div>
</div>

<div class="container">
	<div class="header1">
		<h2> Describing the predominant metrics used in Systems Research helps identify core evaluation criteria, track prevalent research trends, and evaluate the relevance of emerging methodologies. This visualization shows that Throughput is the most popular metric in the Systems Research community. 
			</h2>
			<p> </p>
		<h2 style="text-align: center; color:darkred;">Domain Question: What metric does the author use more frequently in Sytems Research to get Published?</h2>
		<h2 style="text-align: center; color:darkblue; padding: 1.5%">Data Question: For each metric, what is the number of papers published in Systems Research? </h2>
		<BarChart dataset={data.dataset}/>
	</div>
</div>

<div class="container">
	<div class="header1">
		<h2> Studying metric choices across conferences unveils community preferences, aiding researchers in aligning their work with conference expectations and organizers in gauging research diversity and evolution.
		</h2>
		<h2  style="text-align: center; color:darkred;">Does the choice of metric depends on which Conference we are targetting?</h2>
		<h2 style="text-align: center; color:darkblue; padding: 1.5%">Data Question:How does the order of the top 5 metrics vary across different conferences?</h2>
		<Grouped_chart dataset={data.dataset}/>
	</div>
</div>

<div class=raincloud-container>
	<div class="header1">
	<h2> Let us explore a visualization where users can not only observe the distribution of measures for each metric but also discern the shape of the distribution. For instance, all these distributions exhibit strong clustering. Additionally, users can effortlessly compare distributions of different measures.</h2>
	<h2 style="text-align: center; color:darkred;"> Domain Question: What is the distribution of the measures?	</h2>
	<h2 style="text-align: center; color:darkblue;"> Data Question: How does the shape of measure distribution vary across various metrics</h2>
	</div>
	{#each raincloudMetrics as [metric, color]}
		<div class=rain>
			<RaincloudPlot dataset={data.dataset} feature="Measure" filteredIndices={metric2indices.get(metric)} buckets=25 axisLabel={metric2str[metric]} {color}/>
		</div>
	{/each}
</div>
<!-- <div class=scattercloud-container>
	<ScattercloudPlot dataset={data.dataset} xFeature="Measure" yFeature="Citation" buckets=15 />
</div> -->
<style>
    /* Introduction Section */
    .introduction {
        text-align: center;
        font-weight: bold;
        font-size: 23px;
        margin-bottom: 50px;
        padding: 20px; /* Add padding */
        background-color: #f0f0f0; /* Add background color */
        box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1); /* Add box shadow */
        border-radius: 10px; /* Add border radius */
        max-width: 800px; /* Limit max width */
        margin: 0 auto; /* Center horizontally */
        color: #3b1195; /* Text color */
    }

    /* Container */
    .container {
        font-family: system-ui, sans-serif;
        font-size: 16px;
        height: auto; /* Change height to auto */
        width: 100vw;
        padding: 2em;
        display: flex;
        flex-direction: column; /* Adjust flex direction */
        gap: 2em;
        align-items: center; /* Center content horizontally */
    }

    /* Raincontainer */
    .raincontainer {
        margin-top: 2em;
        text-align: center;
    }

    /* Header */
    .header1 {
        justify-content: center;
        align-items: center;
        flex: 1;
        text-align: center; /* Center text */
    }

    /* Title */
    .title {
        font-family: 'Arial', sans-serif;
        font-size: 36px;
        margin-bottom: 20px;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
    }

    /* Title decoration */
    .title::before {
        content: '';
        display: block;
        width: 100%;
        height: 2px;
        background-color: #FFD166;
        margin-bottom: 10px;
    }

    .title::after {
        content: '';
        display: block;
        width: 75%;
        height: 2px;
        background-color: #EF476F;
        margin: 10px auto 0;
    }
</style>
