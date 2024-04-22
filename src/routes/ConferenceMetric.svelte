<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    export let borderBoxSize;


    let data = [];
    // Set dimensions
    let width = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;
    let height = borderBoxSize
        ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize)
        : 800;
    const margin = { top: 20, right: 20, bottom: 50, left: 60 };

    // Color scale for conferences
    const color = d3.scaleOrdinal()
        .domain(["SOSP", "EuroSys", "MICRO"])
        .range(["#1f77b4", "#ff7f0e", "#2ca02c"]);

    // Scales for the chart
    let x0 = d3.scaleBand().rangeRound([0, width]).paddingInner(0.1);
    let x1 = d3.scaleBand().padding(0.05);
    let y = d3.scaleLinear().rangeRound([height, 0]);

    onMount(async () => {
        data = await d3.csv('conference_metric_avg.csv', d => ({
            conference: d.conference,
            metric: d.metric,
            measure: +d.measure  // convert measure to number
        }));
        updateScales(data);
    });

    function updateScales(data) {
        // Set domains for the scales
        const metrics = Array.from(new Set(data.map(d => d.metric)));
        const conferences = Array.from(new Set(data.map(d => d.conference)));
        
        x0.domain(metrics);
        x1.domain(conferences).rangeRound([0, x0.bandwidth()]);
        y.domain([0, d3.max(data, d => d.measure)]).nice();
    }
</script>

<div class="conferencemetric" bind:borderBoxSize>
    <svg width={width + margin.left + margin.right} height={height + margin.top + margin.bottom}>
        <g transform={`translate(${margin.left}, ${margin.top})`}>
            <!-- Draw bars -->
            {#each data as d (d.metric + d.conference)}
                <rect
                    fill={color(d.conference)}
                    x={x0(d.metric) + x1(d.conference)}
                    y={y(d.measure)}
                    width={x1.bandwidth()}
                    height={height - y(d.measure)}
                />
            {/each}
    
            <!-- x-axis -->
            <g transform={`translate(0,${height})`} class="axis">
                {#each x0.domain() as metric}
                    <g transform={`translate(${x0(metric) + x0.bandwidth() / 2},0)`}>
                        <text y={20} text-anchor="middle">{metric}</text>
                    </g>
                {/each}
            </g>
            <!-- y-axis -->
            <g class="axis">
                {#each d3.ticks(0, d3.max(data, d => d.measure), 10) as tick}
                    <text x="-9" y={y(tick)} dy="0.35em">{tick}</text>
                {/each}
            </g>
            <!-- Legend -->
            <g transform="translate(0, -20)">
                {#each color.domain() as conference, index}
                    <g transform={`translate(${index * 120},0)`}>
                        <rect width="18" height="18" fill={color(conference)}></rect>
                        <text x="24" y="9" dy="0.35em">{conference}</text>
                    </g>
                {/each}
            </g>
        </g>
    </svg>
</div>

<style>
    .axis text {
        font-size: 12px;
        fill: #000;
    }
    text {
        font-family: 'Arial', sans-serif;
    }
    .conferencemetric {
        /* Center the chart */
        display: flex;
        justify-content: center;
        align-items: center;
        /* border: 1px solid black; */
    }
</style>
