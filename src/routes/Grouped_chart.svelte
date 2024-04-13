<script>
    import * as d3 from 'd3';
    import Axis from './Axis.svelte';

    export let dataset;
    export let borderBoxSize;

    // Define a list of metric full forms
    const fullForms = [
        { metric: "TH", fullForm: "Throughput" },
        { metric: "EN", fullForm: "Energy Cost" },
        { metric: "LT", fullForm: "Latency" },
        { metric: "EF", fullForm: "Energy Efficiency" },
        { metric: "BW", fullForm: "Bandwidth" },
        { metric: "BR", fullForm: "Bugs Resolved" },
        { metric: "ET", fullForm: "Execution Time" },
        { metric: "WA", fullForm: "Wasted Actions" },
        { metric: "AC", fullForm: "Accuracy" },
        { metric: "SP", fullForm: "Space" },
        { metric: "C", fullForm: "Cost" },
        { metric: "MT", fullForm: "Memory Traffic" },
        { metric: "ER", fullForm: "Error Rate" },
        { metric: "LOC", fullForm: "Lines of Code" },
        { metric: "PR", fullForm: "Pricing Cost" },
        { metric: "OV", fullForm: "Overhead" },
        { metric: "IN", fullForm: "Interference" },
        { metric: "FL", fullForm: "Failure Rate" },
        { metric: "IPC", fullForm: "Instructions Per Cycle" },
        { metric: "AV", fullForm: "Availability" },
        // Add more metric full forms as needed
    ];

    // Group the data by conference
    const groupedData = d3.group(dataset, d => d.Conference);

    // Sort each group by count of metrics and select the top 5 metrics
    const topMetricsByGroup = new Map();
    groupedData.forEach((group, conference) => {
        const metricsCount = d3.rollup(group, v => v.length, d => d.Metric);
        const topMetrics = Array.from(metricsCount.entries())
            .sort((a, b) => b[1] - a[1]) // Sort by count in descending order
            .slice(0, 5) // Select top 5 metrics
            .map(d => d[0]); // Extract metric names
        topMetricsByGroup.set(conference, topMetrics);
    });

    // Prepare data for grouped bar chart
    const dataForChart = [];
    groupedData.forEach((group, conference) => {
        const metricsInGroup = topMetricsByGroup.get(conference);
        metricsInGroup.forEach(metric => {
            const count = group.filter(d => d.Metric === metric).length;
            dataForChart.push({ conference, metric, count });
        });
    });

    // Set up color scale
    const colorScale = d3.scaleOrdinal()
        .domain(Array.from(topMetricsByGroup.values()).flat()) // Domain set to all metrics across all groups
        .range(d3.schemeCategory10); // Adjust color range as needed

    // Set dimensions
    let width = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;
    let height = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;
    const margin = { top: 20, right: 20, bottom: 50, left: 60 };

    // Set up x-axis scale for groups
    const x0 = d3.scaleBand()
        .domain(Array.from(groupedData.keys()))
        .rangeRound([margin.left, width - margin.right])
        .paddingInner(0.1);

    const y = d3.scaleLinear()
        .domain([0, d3.max(dataForChart, d => d.count)])
        .nice()
        .rangeRound([height - margin.bottom, margin.top]);

    // Set up x1-axis scale for top metrics within each group
    const x1Scales = new Map();
    groupedData.forEach((_, conference) => {
        const topMetrics = topMetricsByGroup.get(conference);
        const x1Scale = d3.scaleBand()
            .domain(topMetrics)
            .rangeRound([0, x0.bandwidth()])
            .padding(0.05);
        x1Scales.set(conference, x1Scale);
    });

    // Function to access x1 scale for a specific conference
    const x1Scale = (conference) => x1Scales.get(conference);

    // Function to show tooltip
    function showTooltip(event, count, metric, conference) {
        const tooltip = document.getElementById('tooltip');
        const fullForm = fullForms.find(d => d.metric === metric)?.fullForm;
        tooltip.innerHTML = `${count}`;
        tooltip.style.display = 'block';

        // Calculate the position relative to the bar
		const barRect = event.currentTarget.getBoundingClientRect();
		const tooltipWidth = tooltip.offsetWidth;
		const tooltipHeight = tooltip.offsetHeight;
		const tooltipX = barRect.left + barRect.width / 2 - tooltipWidth / 2;
		const tooltipY = barRect.top - tooltipHeight - 5; // Adjust as needed for spacing

        tooltip.style.left = tooltipX + 'px';
        tooltip.style.top = tooltipY + 'px';
            // Make the bar pop out
        event.currentTarget.style.transform = 'translateY(-5px)';
    }

    // Function to hide tooltip and reset bar position
    function hideTooltip(event) {
        const tooltip = document.getElementById('tooltip');
        tooltip.style.display = 'none';

        // Reset bar position
        event.currentTarget.style.transform = 'translateY(0)';
    }
</script>

<div class="groupedbarchart" bind:borderBoxSize>
    <svg width={width} height={height}>
        <g transform={`translate(${margin.left},${margin.top})`}>
            {#each groupedData.keys() as conference}
                {#each topMetricsByGroup.get(conference) as metric}
                    <rect
                        x={x0(conference) + x1Scale(conference)(metric)}
                        y={y(dataForChart.find(d => d.conference === conference && d.metric === metric)?.count || 0)}
                        width={x1Scale(conference).bandwidth()}
                        height={height - margin.bottom - y(dataForChart.find(d => d.conference === conference && d.metric === metric)?.count || 0)}
                        fill={colorScale(metric)}
                        on:mouseover={(event) => showTooltip(event, dataForChart.find(d => d.conference === conference && d.metric === metric)?.count || 0, metric, conference)}
                        on:focus={(event) => showTooltip(event, dataForChart.find(d => d.conference === conference && d.metric === metric)?.count || 0, metric, conference)}
                        on:mouseout={hideTooltip}
                        on:blur={hideTooltip}
                        tabindex="0"
                        role="button"
                        aria-label={`${dataForChart.find(d => d.conference === conference && d.metric === metric)?.count}`}
                        style="transition: all 0.3s ease; transform-origin: center; cursor: pointer;"
                        class="bar"
                    />
                {/each}
            {/each}
            <Axis orientation="bottom" margin={margin} width={width} height={height} scale={x0} label="Conference" />
            <Axis orientation="left" margin={margin} width={width} height={height} scale={y} label="Count" />
        </g>
        
    </svg>

   <!-- Legend -->
   <table class="legend">
    <tbody>
        {#each Array.from(colorScale.domain()) as metric}
            <tr>
                <td>
                    <svg width="20" height="20">
                        <rect width="20" height="20" fill={colorScale(metric)} />
                    </svg>
                </td>
                <td>{fullForms.find((d) => d.metric === metric).fullForm}</td>
            </tr>
        {/each}
    </tbody>
</table>
<div id="tooltip" style="position: absolute; display: none; background-color: rgba(255, 255, 255, 0.8); padding: 4px; border: 1px solid black; border-radius: 4px;"></div>

</div>

<style>
    /* Add custom styles here */
    .groupedbarchart {
        /* Center the chart */
        display: flex;
        justify-content: center;
        align-items: center;
        border: 1px solid black;
        
    }
    
    .bar:hover {
        transform: translateY(-5px);
    }


</style>
