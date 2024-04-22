<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let data = [];
    let filteredData = [];
    let svgElement; // Reference to the SVG element for D3 manipulation
    let conferences = [];
    let selectedConference = ''; // Bound to the dropdown for selecting conference

    let tooltipVisibility = 'hidden';  // CSS visibility property
    let tooltipContent = '';  // Content of the tooltip
    let tooltipX = 0;  // X position of the tooltip relative to the circle
    let tooltipY = 0;  // Y position of the tooltip relative to the circle

    const customColorScheme = [
        "#EA9180",
        "#E99196",
        "#DE95AB",
        "#CB9DBD",
        "#B2A6C7",
        "#96AFC9",
        "#7DB6C2",
        "#6FBBB4",
        '#71BDA0',
        "#80BD8A",
        "#96BB76",
        "#AFB767",
        "#C8B061",
        "#DEA964",
    ]

    onMount(async () => {
        data = await d3.csv('/research_direction.csv', d => ({
            field: d.field,
            count: +d.count,
            conference: d.conference
        }));
        conferences = Array.from(new Set(data.map(d => d.conference))); // Get unique conferences
        selectedConference = 'all'; // Default to all conferences
        updateVisualization();
    });

    function updateVisualization() {
        filteredData = data.filter(d => d.conference === selectedConference);
        generateCircles();
    }

    function generateCircles() {
        const svg = d3.select(svgElement);
        svg.selectAll("g").remove(); // Clear existing circles and legend

        const width = +svg.attr('width') - 400;
        const height = +svg.attr('height');

        const colorScale = d3.scaleOrdinal()
            .domain(filteredData.map(d => d.field))
            .range(customColorScheme);

        const radiusScale = d3.scaleSqrt()
            .domain([0, d3.max(filteredData, d => d.count)])
            .range([10, 60]);

        // Calculate grid positions
        const numCols = Math.floor(width / (60 * 2)); // Maximum number of columns
        const numRows = Math.ceil(filteredData.length / numCols); // Calculate needed rows
        const horizontalSpacing = width / numCols;
        const verticalSpacing = height / numRows;

        const circleGroup = svg.append("g"); // Group for circles
        filteredData.forEach((d, i) => {
            d.x = (i % numCols) * horizontalSpacing + horizontalSpacing / 2; // X position
            d.y = Math.floor(i / numCols) * verticalSpacing + verticalSpacing / 2; // Y position
        });

        circleGroup.selectAll("circle")
            .data(filteredData)
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

        
        // Create legend
        const legend = svg.append("g")
            .attr("transform", `translate(${width + 20}, 30)`)
            .attr("class", "legend");

        const legendEntries = legend.selectAll(null)
            .data(colorScale.domain())
            .enter()
            .append("g")
            .attr("transform", (d, i) => `translate(0, ${i * 25})`);

        legendEntries.append("rect")
            .attr("width", 20)
            .attr("height", 20)
            .attr("fill", colorScale);

        legendEntries.append("text")
            .attr("x", 25)
            .attr("y", 15) // Align text with squares
            .text(d => d)
            .style("font-size", "12px")
            .style("font-family", "Arial, sans-serif");
    }

    function handleConferenceChange(event) {
        selectedConference = event.target.value;
        updateVisualization();
    }
</script>

<div class="circles">
    <select bind:value={selectedConference} on:change={handleConferenceChange}>
        {#each conferences as conference}
            <option value={conference}>{conference}</option>
        {/each}
    </select>
    <svg bind:this={svgElement} width="1200" height="600" style="border: 1px solid black;">
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
