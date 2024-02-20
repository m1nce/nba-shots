<script>
    import { onMount, onDestroy } from 'svelte';
    import pkg from 'lodash';
    const {debounce} = pkg;
    import * as d3 from 'd3';

    let width = 0;
    let height = 0;
    let leftcornerregion = { x: 0, y: 40.2, width: 6, height: 100 };
    let rightcornerregion = { x: 93.9, y: 40.2, width: 6.5, height: 100 };
    let paintregion = { x: 33.9, y: 56, width: 32, height: 44.2 };
    let restrictedregion = { x: 49.9, y: 14.5 };
    let restrictedregion_rec = { x: 42, y: 14.5, width: 15.65, height: 2.4 };

    let svg;

    // Check if document is defined before running client-side code
    if (typeof document !== 'undefined') {
        onMount(() => {
            const container = document.getElementById('container');
            width = container.offsetWidth;
            height = container.offsetHeight;
            svg = d3.select('#overlay');

            window.addEventListener('resize', debounce(handleResize, 300));
            renderSVG();
        });

        onDestroy(() => {
            window.removeEventListener('resize', handleResize);
        });
    }

    function handleResize() {
        const container = document.getElementById('container');
        width = container.offsetWidth;
        height = container.offsetHeight;
        renderSVG();
    }

    function renderSVG() {
        if (!svg) {
            console.error('SVG container not found');
            return;
        }
        console.log('Rendering SVG');

        svg.selectAll('*').remove();

        const xScale = d3.scaleLinear().domain([0, 100]).range([0, width]);
        const yScale = d3.scaleLinear().domain([0, 100]).range([height, 0]);

        const xAxis = d3.axisBottom(xScale);
        const yAxis = d3.axisLeft(yScale);
        
        svg.attr('width', width).attr('height', height);

        svg.append('g').attr('transform', `translate(0,${height})`).call(xAxis);
        svg.append('g').call(yAxis);

        svg.append('rect')
            .attr('x', xScale(leftcornerregion.x))
            .attr('y', yScale(leftcornerregion.y))
            .attr('width', xScale(leftcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(leftcornerregion.height))
            .attr('fill', 'blue')
            .attr('fill-opacity', '0.5');

        svg.append('rect')
            .attr('x', xScale(rightcornerregion.x))
            .attr('y', yScale(rightcornerregion.y))
            .attr('width', xScale(rightcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(rightcornerregion.height))
            .attr('fill', 'red')
            .attr('fill-opacity', '0.5');

        svg.append('rect')
            .attr('x', xScale(paintregion.x))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(paintregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(paintregion.height))
            .attr('fill', 'green')
            .attr('fill-opacity', '0.5');
        let radius = Math.min(width, height) / 2 * .235;
        const arc = d3.arc()
            .innerRadius(0)
            .outerRadius(radius)
            .startAngle(-Math.PI / 2)
            .endAngle(Math.PI / 2);

        svg.append('path')
            .attr('transform', `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
            .attr('d', arc)
            .attr('fill', 'orange')
            .attr('fill-opacity', '0.5');

        svg.append('rect')
            .attr('x', xScale(restrictedregion_rec.x))
            .attr('y', yScale(restrictedregion_rec.y))
            .attr('width', xScale(restrictedregion_rec.width) - xScale(0))
            .attr('height', yScale(0) - yScale(restrictedregion_rec.height))
            .attr('fill', 'orange')
            .attr('fill-opacity', '0.5');
    }
</script>

<style>
    #container {
        position: relative;
        width: 100%;
    }

    #overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    img {
        display: block;
        width: 100%;
        height: auto;
    }
</style>

<div id="container">
    <img src='src/nbacourt.jpg' alt='Basketball Court'>
    <svg id="overlay"></svg>
</div>
