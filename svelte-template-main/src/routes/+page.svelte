
<h1>Welcome to SvelteKit</h1>
<script>
    let width = 400;
    let height =735;
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    let region = {
    x: 0,
    y: 40.2,
    width: 16.05,  // Width of the region
    height: 100  // Height of the region
  };
    onMount(() => {
    const xScale = d3.scaleLinear()
        .domain([0, 100]) // Set the domain for your specific data
        .range([0, width]);
  
    const yScale = d3.scaleLinear()
        .domain([0, 100]) // Set the domain for your specific data
        .range([height, 0]);
  
    const svg = d3.select('#overlay')
        .attr('width', width)
        .attr('height', height);
  
    const xAxis = d3.axisBottom(xScale);
    const yAxis = d3.axisLeft(yScale);
  
    svg.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(xAxis);
  
    svg.append("g")
        .call(yAxis);
    svg.append("rect")
      .attr("x", xScale(region.x))
      .attr("y", yScale(region.y))
      .attr("width", xScale(region.width) - xScale(0))
      .attr("height", yScale(0) - yScale(region.height))
      .attr("fill", "blue")
      .attr("stroke", "blue")
      .attr("stroke-width", 2)
      
    svg.append("rect")
      .attr("x", xScale(region.x))
      .attr("y", yScale(region.y))
      .attr("width", xScale(region.width) - xScale(0))
      .attr("height", yScale(0) - yScale(region.height))
      .attr("fill", "blue")
      .attr("stroke", "blue")
      .attr("stroke-width", 2)
  });
  </script>

<style>
    #container {
      position: relative;
      width: fit-content;
    }
  
    #overlay {
      position: absolute;
      top: 0;
      left: 0;
    }
  
    img {
      display: block;
      width: 100%;
      height: auto;
    }
  </style>
  
  <div id="container">
    <img src='src/nbacourt.jpg' alt='Basketball Court' width="500" height="500">
    <svg id="overlay"></svg>
  </div>
  