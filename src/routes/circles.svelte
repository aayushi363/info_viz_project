<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let data = [];
    let svgElement; // Reference to the SVG element for D3 manipulation

    let tooltipContent = '';  // Content of the tooltip
    let tooltipVisibility = 'hidden';  // CSS visibility property
    let tooltipX = 0;  // X position of the tooltip
    let tooltipY = 0;  // Y position of the tooltip

    onMount(async () => {
        data = await d3.csv('/research_direction.csv', d => ({
            field: d.field,
            count: +d.count  // Convert count to a number
        }));
        generateCircles();
    });

    function generateCircles() {
        const svg = d3.select(svgElement);
        const width = +svg.attr('width');
        const height = +svg.attr('height');

        const colorScale = d3.scaleOrdinal()
            .domain(data.map(d => d.field))
            .range(d3.schemeCategory10);

        const radiusScale = d3.scaleSqrt()
            .domain([0, d3.max(data, d => d.count)])
            .range([10, 60]);

        const simulation = d3.forceSimulation(data)
            .force("center", d3.forceCenter(width / 2, height / 2))
            .force("collide", d3.forceCollide(d => radiusScale(d.count) + 1))
            .on("tick", ticked);

        function ticked() {
            svg.selectAll("circle")
                .data(data)
                .join("circle")
                .attr("r", d => radiusScale(d.count))
                .attr("cx", d => d.x)
                .attr("cy", d => d.y)
                .attr("fill", d => colorScale(d.field))
                .on("mouseover", (event, d) => {
                    tooltipContent = `${d.field}: ${d.count}`;
                    tooltipX = event.clientX;
                    tooltipY = event.clientY - 28; // Position slightly above the cursor
                    tooltipVisibility = 'visible';
                })
                .on("mouseout", () => {
                    tooltipVisibility = 'hidden';
                });
        }
    }
</script>

<div class="circles">
    <svg bind:this={svgElement} width="800" height="600" style="border: 1px solid black;">
        <!-- Tooltip -->
        <foreignObject x={tooltipX} y={tooltipY} width="100" height="50" style="visibility: {tooltipVisibility}; pointer-events: none;">
            <div xmlns="http://www.w3.org/1999/xhtml" style="background: white; border: 1px solid black; padding: 8px; border-radius: 6px;">
                {tooltipContent}
            </div>
        </foreignObject>
    </svg>
</div>

<style>
    .circles {
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #f9f9f9; /* Light grey background */
        border: 1px solid #ccc;
        padding: 20px;
        margin: 20px;
    }

    svg {
        border: 1px solid black;
    }

    foreignObject div {
        font-family: Arial, sans-serif;
        font-size: 14px;
        color: #333;
    }
</style>