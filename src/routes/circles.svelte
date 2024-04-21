<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let data = [];
    let svgElement; // Reference to the SVG element for D3 manipulation

    let tooltipVisibility = 'hidden';  // CSS visibility property
    let tooltipContent = '';  // Content of the tooltip
    let tooltipX = 0;  // X position of the tooltip relative to the circle
    let tooltipY = 0;  // Y position of the tooltip relative to the circle

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

        // Calculate grid positions
        const numCols = Math.floor(width / (60 * 2)); // Maximum number of columns
        const numRows = Math.ceil(data.length / numCols); // Calculate needed rows
        const horizontalSpacing = width / numCols;
        const verticalSpacing = height / numRows;

        data.forEach((d, i) => {
            d.x = (i % numCols) * horizontalSpacing + horizontalSpacing / 2; // X position
            d.y = Math.floor(i / numCols) * verticalSpacing + verticalSpacing / 2; // Y position
        });

        svg.selectAll("circle")
            .data(data)
            .join("circle")
            .attr("r", d => radiusScale(d.count))
            .attr("cx", d => d.x)
            .attr("cy", d => d.y)
            .attr("fill", d => colorScale(d.field))
            .on("mouseover", (event, d) => {
                tooltipContent = `${d.field}: ${d.count}`;
                let tooltipWidth = 150;  // Define the tooltip width
                let tooltipHeight = 100; // Define the tooltip height
                // Adjust tooltip position to avoid overflow
                tooltipX = Math.min(d.x - tooltipWidth / 2, width - tooltipWidth);
                tooltipY = d.y - tooltipHeight - 40; // Place it above the circle
                if (tooltipY < 0) {
                    tooltipY = d.y + 60; // If out of upper boundary, place it below the circle
                }
                tooltipVisibility = 'visible';
            })
            .on("mouseout", () => {
                tooltipVisibility = 'hidden';
            });
    }
</script>

<div class="circles">
    <svg bind:this={svgElement} width="800" height="600" style="border: 1px solid black;">
        <foreignObject x={tooltipX} y={tooltipY} width="150" height="100" style:visibility={tooltipVisibility} class="tooltip">
            <div xmlns="http://www.w3.org/1999/xhtml" style="background: white; border: 1px solid black; padding: 10px; border-radius: 5px;">
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
        padding: 20px;
        margin: 20px;
    }

    svg {
        border: 1px solid black;
    }

    .tooltip {
        position: absolute; /* Adjust position relative to the SVG element */
        z-index: 1000; /* Ensure tooltip is rendered above all circles */
    }

    foreignObject div {
        font-family: Arial, sans-serif;
        font-size: 14px;
        color: #333;
    }
</style>
