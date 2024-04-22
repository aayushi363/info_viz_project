<script>
    import * as d3 from 'd3';
    import Axis from './Axis.svelte';

    export let dataset;

    //dimensions
    let borderBoxSize;

    $: width = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;

    $: height = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;

    const margin = { top: 20, right: 20, bottom: 50, left: 60 };

    //get count of each group
    $: groupcounts = d3.rollup(
        dataset,
        (g) => g.length,
        (d) => d.Metric
    );

    //color scale
    $: colorScale = d3.scaleOrdinal()
        .domain(Array.from(groupcounts.keys()))
        .range(d3.schemeTableau10);

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

    function showTooltip(event, count, metric) {
        const tooltip = document.getElementById('tooltip');
        const fullForm = fullForms.find((d) => d.metric === metric).fullForm;
        tooltip.innerHTML = `${fullForm}: ${count}`;
        tooltip.style.display = 'block';
        // Calculate the position relative to the bar
        const barRect = event.currentTarget.getBoundingClientRect();
        const tooltipWidth = tooltip.offsetWidth;
        const tooltipHeight = tooltip.offsetHeight;
		const tooltipX = Math.min(d.x - tooltipWidth / 2, width - tooltipWidth);
		const tooltipY = d.y - tooltipHeight - 80; // Place it above the circle
        // const tooltipY = barRect.top + barRect.height / 2 - tooltipHeight / 2;

        tooltip.style.left = tooltipX + 'px';
        tooltip.style.top = tooltipY + 'px';
    }

    function hideTooltip() {
        const tooltip = document.getElementById('tooltip');
        tooltip.style.display = 'none';
    }


    // sort the groups by count in descending order
    $: sortedGroupCounts = Array.from(groupcounts.entries())
        .sort((a, b) => a[1] - b[1]);

    //scales
    $: maxCount = d3.max(sortedGroupCounts.map(d => d[1]));

    $: x = d3
        .scaleLinear()
        .domain([0, maxCount])
        .nice()
        .range([margin.left, width - margin.right]);
    $: y = d3
        .scaleBand()
        .domain(sortedGroupCounts.map(d => d[0]))
        .range([height - margin.bottom, margin.top])
        .padding(0.3);

</script>

<div class="barchart" style="position: relative;" bind:borderBoxSize>
    <svg {height} {width}>
        <!-- bars -->
        <g>
            {#each sortedGroupCounts as [metric, count]}
                <rect
                    x={x(0)}
                    y={y(metric)}
                    width={x(count) - x(0)}
                    height={y.bandwidth()}
                    fill={colorScale(metric)}
                    on:mouseover={(event) => showTooltip(event, count, metric)}
                    on:focus={(event) => showTooltip(event, count, metric)}
                    on:mouseout={hideTooltip}
                    on:blur={hideTooltip}
                    tabindex="0"
                    role="button"
                    aria-label={`${metric}: ${count}`}
                    style="transition: all 0.3s ease; transform-origin: center; cursor: pointer;"
                    class="bar"
                />
            {/each}
        </g>
        <!-- axes -->
        <Axis orientation="left" margin={margin} width={width} height={height} scale={y} label="Metric" />
        <Axis orientation="bottom" margin={margin} width={width} height={height} scale={x} label="Count" />
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
    .barchart {
        /* center the barchart */
        justify-content: center;
        align-items: center;
        display: flex;
        height: 100%;
        flex: 1;
        /* border: 1px solid black; */
    }

    .bar:hover {
        transform: translateY(-5px);
    }
</style>
