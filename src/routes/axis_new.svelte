<script>
    import { select, scaleBand, scaleLinear, axisBottom, axisLeft } from 'd3';
  
    export let data;
    export let width = 600;
    export let height = 400;
    export let margin = { top: 20, right: 30, bottom: 40, left: 50 };
  
    let xScale, yScale;
  
    function createScales() {
      xScale = scaleBand()
        .domain(data.map(d => d.metric))
        .range([margin.left, width - margin.right])
        .padding(0.1);
  
      yScale = scaleLinear()
        .domain([0, Math.max(...data.map(d => d.count))])
        .nice()
        .range([height - margin.bottom, margin.top]);
    }
  
    function createAxis() {
      const svg = select('.chart');
  
      svg.select('.x-axis')
        .attr('transform', `translate(0, ${height - margin.bottom})`)
        .call(axisBottom(xScale));
  
      svg.select('.y-axis')
        .attr('transform', `translate(${margin.left}, 0)`)
        .call(axisLeft(yScale));
    }
  
    function updateScales() {
      xScale.range([margin.left, width - margin.right]);
      yScale.range([height - margin.bottom, margin.top]);
    }
  
    $: createScales();
    $: createAxis();
  </script>
  
  <svg class="chart" width={width} height={height}>
    <g class="x-axis"></g>
    <g class="y-axis"></g>
  </svg>
  
  <style>
    .axis path,
    .axis line {
      fill: none;
      stroke: black;
      shape-rendering: crispEdges;
    }
  
    .axis text {
      font-family: sans-serif;
      font-size: 10px;
    }
  </style>
  