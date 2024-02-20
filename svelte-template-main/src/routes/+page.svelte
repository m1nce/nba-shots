<h1>Welcome to SvelteKit</h1>
<script>
    let width = 1200;
    let height =735;
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    let leftcornerregion = {
    x: 0,
    y: 40.2,
    width: 5.359,
    height: 100
    };
    let rightcornerregion = {
        x: 85,
        y: 40.2,
        width: 5.359,
        height: 100
    };
    let paintregion = {
        x: 30.8,
        y: 56,
        width: 29,
        height: 44.2
    };
    let restrictedregion = {
        x: 45.2,
        y: 14
    }
    let restrictedregion_rec = {
        x: 38.14,
        y: 14.005,
        width: 14.13,
        height: 2.1
    }
    onMount(() => {
    const xScale = d3.scaleLinear()
        .domain([0, 100])
        .range([0, width]);
  
    const yScale = d3.scaleLinear()
        .domain([0, 100])
        .range([height, 0]);
  
    const svg = d3.select('#overlay')
        .attr('width', width)
        .attr('height', height)
        .attr("viewBox", `0 0 ${width} ${height}`);
  
    const xAxis = d3.axisBottom(xScale);
    const yAxis = d3.axisLeft(yScale);
  
    svg.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(xAxis);
  
    svg.append("g")
        .call(yAxis);
    svg.append("rect")
      .attr("x", xScale(leftcornerregion.x))
      .attr("y", yScale(leftcornerregion.y))
      .attr("width", xScale(leftcornerregion.width) - xScale(0))
      .attr("height", yScale(0) - yScale(leftcornerregion.height))
      .attr("fill", "blue")
      .attr("stroke", "blue")
      .attr("fill-opacity", "0.5")
      .attr("stroke", "none")
      
    svg.append("rect")
      .attr("x", xScale(rightcornerregion.x))
      .attr("y", yScale(rightcornerregion.y))
      .attr("width", xScale(rightcornerregion.width) - xScale(0))
      .attr("height", yScale(0) - yScale(rightcornerregion.height))
      .attr("fill", "red")
      .attr("stroke", "red")
      .attr("fill-opacity", "0.5")
      .attr("stroke", "none")
    svg.append("rect")
      .attr("x", xScale(paintregion.x))
      .attr("y", yScale(paintregion.y))
      .attr("width", xScale(paintregion.width) - xScale(0))
      .attr("height", yScale(0) - yScale(paintregion.height))
      .attr("fill", "green")
      .attr("stroke", "green")
      .attr("fill-opacity", "0.5")
      .attr("stroke", "none");
    const arc = d3.arc()
        .innerRadius(0)
        .outerRadius(85)
        .startAngle(-Math.PI / 2)
        .endAngle(Math.PI/2);
    svg.append("path")
    .attr("transform", `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
        .attr("d", arc)
        .attr("fill", "orange")
        .attr("stroke", "orange")
        .attr("stroke", 'none')
        .attr("fill-opacity", "0.5");
    svg.append("rect")
      .attr("x", xScale(restrictedregion_rec.x))
      .attr("y", yScale(restrictedregion_rec.y))
      .attr("width", xScale(restrictedregion_rec.width) - xScale(0))
      .attr("height", yScale(0) - yScale(restrictedregion_rec.height))
      .attr("fill", "orange")
      .attr("stroke", "orange")
      .attr("fill-opacity", "0.5")
      .attr("stroke", "none")
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
<svg preserveAspectRatio="xMidYMid meet"></svg>

  <div id="container">
    <img src='src/nbacourt.jpg' alt='Basketball Court' width="500" height="500">
    <svg id="overlay"></svg>
  </div>