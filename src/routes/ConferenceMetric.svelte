<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let borderBoxSize;

    let data = [];
    // Responsive size or default to 800x800
    let width = borderBoxSize ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize) - margin.left - margin.right : 800;
    let height = borderBoxSize ? Math.min(borderBoxSize[0].blockSize, borderBoxSize[0].inlineSize) - margin.top - margin.bottom : 800;
    const margin = { top: 50, right: 20, bottom: 50, left: 60 };

    const color = d3.scaleOrdinal()
        .domain(["SOSP", "EuroSys", "MICRO"])
        .range(["#1f77b4", "#ff7f0e", "#2ca02c"]);

    let x0 = d3.scaleBand().paddingInner(0.1);
    let x1 = d3.scaleBand().padding(0.05);
    let y = d3.scaleLinear();

    onMount(async () => {
        data = await d3.csv('conference_metric_avg.csv', d => ({
            conference: d.conference,
            metric: d.metric,
            measure: +d.measure
        }));
        updateScales();
    });

    function updateScales() {
        const metrics = Array.from(new Set(data.map(d => d.metric)));
        const conferences = Array.from(new Set(data.map(d => d.conference)));
        x0.domain(metrics).rangeRound([0, width]);
        x1.domain(conferences).rangeRound([0, x0.bandwidth()]);
        y.domain([0, d3.max(data, d => d.measure)]).nice().rangeRound([height, 0]);
    }
</script>

<div class="conferencemetric" bind:borderBoxSize>
    <svg width={width + margin.left + margin.right} height={height + margin.top + margin.bottom}>
        <g transform={`translate(${margin.left}, ${margin.top})`}>
            {#each data as d (d.metric + d.conference)}
                <rect
                    fill={color(d.conference)}
                    x={x0(d.metric) + x1(d.conference)}
                    y={y(d.measure)}
                    width={x1.bandwidth()}
                    height={height - y(d.measure)}
                />
            {/each}
            <g transform={`translate(0, ${height})`}>
                {#each data as d, index}
                    <g transform={`translate(${x0(d.metric) + x0.bandwidth() / 2}, 0)`}>
                        <text y="10" dy="0.71em" text-anchor="middle">{d.metric}</text>
                    </g>
                {/each}
            </g>
            <g class="y-axis">
                {#each d3.ticks(0, d3.max(data, d => d.measure), 10) as tick}
                    <text x="-20" y={y(tick)} dy="0.35em">{tick}</text>
                {/each}
            </g>
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
    .x-axis text, .y-axis text {
        font-size: 12px;
        fill: #000;
    }
    text {
        font-family: 'Arial', sans-serif;
    }
    .conferencemetric {
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>
