<h1>The Hottest NBA Players</h1><head><link href='https://fonts.googleapis.com/css?family=Playfair+Display' rel='stylesheet'></head>
<h4>Which NBA Players are the HOTTEST by Field Goal Percentage (FG%)? Only players that partook in the 2022 NBA All-Star Draft are included. The redder the zone, the hotter and better they are in comparison to the All-Star roster average!</h4>
<script>
    import { base } from '$app/paths';
    import { onMount, onDestroy, afterUpdate } from 'svelte';
    import pkg from 'lodash';
    const { debounce } = pkg;
    import * as d3 from 'd3';
    let svg;
    let showPlayerInfo = true;
    let selectedPlayer = "LeBron James";
    let playerImages = {
        "LeBron James": `${base}/lebronjames23.jpeg`,
        "Kevin Durant": `${base}/kevindurant35.jpeg`,
        "Giannis Antetokounmpo": `${base}/giannis34.jpeg`,
        "Joel Embiid": `${base}/joelembiid21.jpeg`,
        "Stephen Curry": `${base}/stephencurry30.jpeg`,
        "Ja Morant": `${base}/jamorant12.jpeg`,
        "DeMar DeRozan": `${base}/demarderozan11.jpeg`,
        "Jayson Tatum": `${base}/jaysontatum0.jpeg`,
        "Nikola Jokic": `${base}/nikolajokic15.jpeg`,
        "Andrew Wiggins": `${base}/andrewwiggins22.jpeg`,
        "Trae Young": `${base}/traeyoung11.jpeg`,
        "Devin Booker": `${base}/devinbooker1.jpeg`,
        "Luka Doncic": `${base}/lukadoncic77.jpeg`,
        "Karl-Anthony Towns": `${base}/kat32.jpeg`,
        "Darius Garland": `${base}/dariusgarland10.jpeg`,
        "Zach LaVine": `${base}/zachlavine8.jpeg`,
        "Chris Paul": `${base}/chrispaul3.jpeg`,
        "Dejounte Murray": `${base}/dejountemurray5.jpeg`,
        "Jimmy Butler": `${base}/jimmybutler22.jpeg`,
        "Khris Middleton": `${base}/khrismiddleton22.jpeg`,
        "Donovan Mitchell": `${base}/donovanmitchell45.jpeg`,
        "LaMelo Ball": `${base}/lameloball1.jpeg`,
        "Fred VanVleet":`${base}/fredvanvleet5.jpeg`,
        "Rudy Gobert": `${base}/rudygobert27.jpeg`,
        "James Harden": `${base}/jamesharden1.jpeg`
    };
    let playerImageSrc = playerImages[selectedPlayer];
    let width = 0;
    let height = 0;
    let leftcornerregion = { x: 0, y: 40.2, width: 6, height: 28.5};
    let rightcornerregion = { x: 93.9, y: 40.2, width: 6.5, height: 28.5};
    let paintregion = { x: 33.9, y: 56, width: 32, height: 44.2 };
    let restrictedregion = { x: 49.9, y: 14.5 };
    let restrictedregion_rec = { x: 42, y: 14.55, width: 15.9, height: 2.4 };
    let middle_outer = {x: 0, y: 100, width: 100, height: 44.2}
    let left_outer = {x: 0, y: 55.8, width: 35, height: 15.8}
    let right_outer = {x: 65.9, y: 55.8, width: 35, height: 15.8}
    let mid_left = {x: 6, y: 40.2, width: 28, height: 28.5}
    let mid_right = {x: 66, y: 40.2, width: 27.8, height: 28.5}
    let mid_middle = { x: 50, y: 14};
    let playershooting = `${base}/player_stats.json`;
    let playerShootingData;
    let averages = {"Above the Break 3": 0.3641223979,
                    "In The Paint (Non-RA)": 0.4604816551,
                    "Left Corner 3": 0.3847352025,
                    "Mid-Range": 0.4461170031,
                    "Restricted Area": 0.6779065226,
                    "Right Corner 3": 0.3843351548}

    // Check if document is defined before running client-side code
    if (typeof document !== 'undefined') {
        onMount(() => {
            const container = document.getElementById('container');
            width = container.offsetWidth;
            height = container.offsetHeight;
            svg = d3.select('#overlay');

            window.addEventListener('resize', debounce(handleResize, 300));
            fetch(playershooting)
            .then(response => response.json())
            .then(data => {
                playerShootingData = data;
                renderSVG();
            })
            .catch(error => console.error('Error loading player shooting data:', error));
        });

        onDestroy(() => {
            window.removeEventListener('resize', handleResize);
        });
    }

    // Handles resizing of window
    function handleResize() {
        const container = document.getElementById('container');
        width = container.offsetWidth;
        height = container.offsetHeight;
        renderSVG();
    }

    // Displays tooltip on hover
    function showTooltip(data, event) {
        const tooltip = document.getElementById('tooltip');
        tooltip.innerHTML = `<b>${data.BASIC_ZONE}</b>:<br>` + 
                            `FG Made: ${data.sum}<br>` +
                            `FG Attempted: ${data.count}<br>` +
                            `FG%: ${String(Math.round(data.mean * 100 * 100) / 100) + '%'}<br>` +
                            `All-Star Average FG%: ${String(Math.round(averages[data.BASIC_ZONE]*100)) + '%'}`;
        
        // Set tooltip position relative to the mouse cursor
        tooltip.style.left = `${event.pageX + 10}px`; // Adjust offset as needed
        tooltip.style.top = `${event.pageY + 10}px`; // Adjust offset as needed

        tooltip.style.opacity='1';
        tooltip.style.display = 'block';
    }  



    function hideTooltip() {
        const tooltip = document.getElementById('tooltip');
        tooltip.style.display = 'none';
        tooltip.style.opacity = 0; // Reset opacity for the next time it's shown
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



        // GENERAL 3PT AREA SVG
        let aboveTheBreak3Data = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Above the Break 3");
        aboveTheBreak3Data.forEach(data => {
            let tooltipText = `${data.BASIC_ZONE}: ${data.mean}`;
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.6) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.2) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .2) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            svg.append('rect')
                .attr('x', xScale(middle_outer.x))
                .attr('y', yScale(middle_outer.y))
                .attr('width', xScale(middle_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(middle_outer.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(left_outer.x))
                .attr('y', yScale(left_outer.y) - 2)
                .attr('width', xScale(left_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(left_outer.height) + 4)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(right_outer.x))
                .attr('y', yScale(right_outer.y) - 2)
                .attr('width', xScale(right_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(right_outer.height) + 4)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // MID RANGE AREA SVG
        let mid_range = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Mid-Range");
        mid_range.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            svg.append('rect')
                .attr('x', xScale(mid_left.x))
                .attr('y', yScale(mid_left.y))
                .attr('width', xScale(mid_left.width) - xScale(0))
                .attr('height', yScale(0) - yScale(mid_left.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
            let threePt_radius = Math.min(width, height) / 2 * 1.39;
            let threePtStartAngle = Math.PI;
            let threePtEndAngle = 0;

            // Calculate the start and end points of the arc
            let threePtStartX = threePt_radius * Math.cos(threePtStartAngle);
            let threePtStartY = threePt_radius * Math.sin(threePtStartAngle);
            let threePtEndX = threePt_radius * Math.cos(threePtEndAngle);
            let threePtEndY = threePt_radius * Math.sin(threePtEndAngle);

            // Create the arc path manually
            let threePtArcPath = `M ${threePtStartX} ${threePtStartY} A ${threePt_radius} ${threePt_radius} 0 0 1 ${threePtEndX} ${threePtEndY}`;

            svg.append('path')
                .attr('transform', `translate(${xScale(mid_middle.x)}, ${yScale(mid_middle.y)})`)
                .attr('d', threePtArcPath)
                .attr('fill', 'none')
                .attr('stroke', 'black')
                .attr('stroke-width', '10');
    
            svg.append('rect')
                .attr('x', xScale(mid_right.x))
                .attr('y', yScale(mid_right.y))
                .attr('width', xScale(mid_right.width) - xScale(0))
                .attr('height', yScale(0) - yScale(mid_right.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            let second_radius = Math.min(width, height) / 2 * 1.40;
            const darc = d3.arc()
                .innerRadius(0)
                .outerRadius(second_radius)
                .startAngle(-Math.PI / 2)
                .endAngle(Math.PI / 2);

            svg.append('path')
                .attr('transform', `translate(${xScale(mid_middle.x)}, ${yScale(mid_middle.y)})`)
                .attr('d', darc)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // RIGHT CORNER 3PT AREA SVG
        let left_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Left Corner 3");
        left_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            if (data.count == 0) {
                fillcolor = '#a3a2af'
            }
            svg.append('rect')
                .attr('x', xScale(rightcornerregion.x))
                .attr('y', yScale(rightcornerregion.y))
                .attr('width', xScale(rightcornerregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(rightcornerregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // LEFT CORNER 3PT AREA SVG
        let right_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Right Corner 3");
        right_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            if (data.count == 0) {
                fillcolor = '#a3a2af'
            }
            svg.append('rect')
                .attr('x', xScale(leftcornerregion.x))
                .attr('y', yScale(leftcornerregion.y))
                .attr('width', xScale(leftcornerregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(leftcornerregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // IN THE PAINT (NON-RA) AREA SVG
        let dapaint = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "In The Paint (Non-RA)");
        dapaint.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            svg.append('circle')
                .attr('cx', xScale(paintregion.x + 16))
                .attr('cy', yScale(paintregion.y))
                .attr('r', xScale(10.8))
                .attr('stroke', 'black')
                .attr('stroke-width', 2)
                .attr('fill', 'none');
            svg.append('rect')
                .attr('x', xScale(paintregion.x))
                .attr('y', yScale(paintregion.y))
                .attr('width', xScale(paintregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(paintregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // RESTRICTED AREA SVG
        let dara = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Restricted Area");
        dara.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#7f54af';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#9a5694';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#b95873';
            } else {
                fillcolor = '#d85a53';
            }
            let radius = Math.min(width, height) / 2 * .235;
            const arc = d3.arc()
                .innerRadius(0)
                .outerRadius(radius)
                .startAngle(-Math.PI / 2)
                .endAngle(Math.PI / 2);

            svg.append('path')
                .attr('transform', `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
                .attr('d', arc)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(restrictedregion_rec.x))
                .attr('y', yScale(restrictedregion_rec.y))
                .attr('width', xScale(restrictedregion_rec.width) - xScale(0))
                .attr('height', yScale(0) - yScale(restrictedregion_rec.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });


        
        // COVER BOTTOM OF NBA COURT IMAGE
        let bottomcornerregion = { x: 0, y:12, width: 100, height: 20};

        svg.append('rect')
            .attr('x', xScale(bottomcornerregion.x))
            .attr('y', yScale(bottomcornerregion.y))
            .attr('width', xScale(bottomcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(bottomcornerregion.height))
            .attr('fill', 'white')
            .attr('fill-opacity', '1');



        // LINE DRAWINGS ON SVG
        svg.append('rect')
            .attr('x', xScale(paintregion.x))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(paintregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(paintregion.height)-2)
            .attr('fill', 'none')
            .attr('fill-opacity', '1')
            .attr('stroke', 'black')
            .attr('stroke-width', '3');

        let outline_radius = Math.min(width, height) / 2 * .235;
        let startAngle = Math.PI;
        let endAngle = 0;
        let startX = outline_radius * Math.cos(startAngle);
        let startY = outline_radius * Math.sin(startAngle);
        let endX = outline_radius * Math.cos(endAngle);
        let endY = outline_radius * Math.sin(endAngle);
        let arcPath = `M ${startX} ${startY} A ${outline_radius} ${outline_radius} 0 0 1 ${endX} ${endY}`;

        svg.append('path')
            .attr('transform', `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
            .attr('d', arcPath)
            .attr('fill', 'none')
            .attr('stroke', 'black')
            .attr('stroke-width', '3');

        svg.append('rect')
            .attr('x', xScale(leftcornerregion.x + 5.3))
            .attr('y', yScale(leftcornerregion.y))
            .attr('width', xScale(0.5))
            .attr('height', xScale(19))
            .attr('fill', 'black')
            .attr('stroke', 'black');

        svg.append('rect')
            .attr('x', xScale(rightcornerregion.x + 0.17))
            .attr('y', yScale(rightcornerregion.y))
            .attr('width', xScale(0.5))
            .attr('height', xScale(19))
            .attr('fill', 'black')
            .attr('stroke', 'black');
        svg.append('rect')
            .attr('x', xScale(paintregion.x + 5.1))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(0.1))
            .attr('height', xScale(29.6))
            .attr('fill', 'black')
            .attr('stroke', 'black');

        svg.append('rect')
            .attr('x', xScale(paintregion.x + 26.8))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(0.1))
            .attr('height', xScale(29.6))
            .attr('fill', 'black')
            .attr('stroke', 'black');
        svg.append('circle')
            .attr('cx', xScale(paintregion.x + 16))
            .attr('cy', yScale(paintregion.y))
            .attr('r', xScale(10.8))
            .attr('stroke', 'black')
            .attr('stroke-width', 2)
            .attr('fill', 'none')
            .style("stroke-dasharray", ("3, 5"));
        svg.append('circle')
            .attr('cx', xScale(paintregion.x + 16))
            .attr('cy', yScale(paintregion.y - 40.5))
            .attr('r', xScale(1.8))
            .attr('stroke', 'orange')
            .attr('stroke-width', 1)
            .attr('fill', 'none');
    }

    // Handle dropdown change event
    function handleDropdownChange(event) {
        // Get the selected option value
        const selectedOption = event.target.value;
        // Update selected player
        selectedPlayer = selectedOption;
        // Get the corresponding photo filename from the mapping object
        playerImageSrc = playerImages[selectedOption];
        renderSVG();
    }
</script>

<link href='https://fonts.googleapis.com/css?family=Playfair+Display:400,700' rel='stylesheet'>
<link href="https://fonts.googleapis.com/css?family=PT+Sans&display=swap" rel="stylesheet">

<style>
    #container {
        animation: fadeIn 1s ease-out;
        position: relative;
        width: 50%;
        margin-left: 8%;
        margin-bottom: 30px;
    }

    h1 {
        animation: fadeInright 1s ease-out;
        text-align: center;
        font-family: 'Playfair Display'; 
        font-size: 45px;
        margin-top: 10px;
    }

    h2 {
        animation: fadeIn 1s ease-out;
        text-align: center;
        margin-top: 5%;
        margin-bottom: 0px;
        padding-top: 0px;
    }

    h4 {
        animation: fadeInright 1s ease-out;
        text-align: center;
        font-family: 'PT Sans', sans-serif;
        margin-left: 5%;
        margin-right: 5%;
        margin-top: 0%;
        margin-bottom: 2%;
        padding-top: 0px;
    }

    p {
        animation: fadeIn 1.5s ease-out;
        text-align:center;
        padding: 1% 10%;
        margin-top: 0px;
    }

    hr {
        border: none; /* Removes the default border */
        height: 1px; /* Sets the height of the line */
        background-color: grey; /* Sets the color of the line */
        margin-top: 5rem; /* Adds space above the line */
        margin-bottom: 0rem; /* Adds space below the line */
    }

    img {
        display: block;
        width: 100%;
        height: auto;
        left: 500px;
        z-index:4;
    }

    #overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 2;
    }

    #player-info {
        position: absolute;
        top: 0;
        left: 130%;
        text-align: center;
        color: black;
        font-family: Arial, sans-serif;
    }

    #player-image {
        width: 263px;
        height: 192px;
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 20px;
    }

    #tooltip {
        background: white;
        border: 1px solid #ccc;
        padding: 10px;
        border-radius: 5px;
        display: none; /* Hidden by default */
        z-index: 3;
        position:absolute;
        transition: opacity 0.2s;
    }

    #legend {
        animation: fadeIn 1s ease-out;
        font-family: Arial, sans-serif;
        display: flex;
        flex-direction: column;
        align-items: left;
        margin-top: 20px;
        margin-left: 8%
    }

    #gradient-legend {
        position: relative;
        margin-bottom: 10px;
    }

    label {
        font-weight: bold;
    }

    select {
        background-color: rgb(225, 225, 225);
        color: black; /* Setting the text color to white for better contrast */
        border: 1.3px solid #ccc; /* Optional: adds a border */
        padding: 0.4em; /* Optional: adds some padding inside the dropdown */
        border-radius: 8px; /* Optional: rounds the corners of the dropdown */
        margin-top: 2%;
        text-align: center;
    }

    @keyframes fadeIn {
        from {
            opacity: 0;
            transform: translateY(-20px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }}
        @keyframes fadeInright {
        from {
            opacity: 0;
            transform: translateY(10px);
        }
        to {
            opacity: 1;
            transform: translateY(-20);
        }
    }

    .gradient-bar {
        width: 240px; /* Increased width to accommodate NA section */
        height: 20px;
        background: linear-gradient(to right, 
                                    #5b52d5 0%, #5b52d5 16.6%, /* Significantly below average */
                                    #7f54af 16.6%, #7f54af 33.2%, /* Below average */
                                    #9a5694 33.2%, #9a5694 49.8%, /* Slightly above average */
                                    #b95873 49.8%, #b95873 66.4%, /* Above average */
                                    #d85a53 66.4%, #d85a53 83%, /* Significantly above average */
                                    #a3a2af 83%, #a3a2af 100% /* Not applicable */
                                    );
        border: 1px solid #ccc;
    }

    .percentage-scale {
        display: flex;
        justify-content: space-between;
        width: 240px; /* Increased width to match gradient bar */
    }

    .frequency-scale span {
        margin-bottom: 5px;
    }
</style>


<div id="container">
    <img src={`${base}/nbacourt.jpg`} alt="Basketball Court">
    <svg id="overlay"></svg>

    {#if showPlayerInfo}
        <div id="player-info">

            <!-- Player information -->
            <h2>{selectedPlayer}</h2>
            <img src={playerImageSrc} alt={selectedPlayer} id="player-image">

            <!-- Dropdown menu for changing players -->
            <label for="playerDropdown">Change Player:</label>
            <select id="playerDropdown" on:change={handleDropdownChange}>
                {#each Object.keys(playerImages) as player}
                    <option value={player}>{player}</option>
                {/each}
            </select>
        </div>
    {/if}
</div>

<div id="tooltip" style="position: absolute; opacity: 1; pointer-events: none; transition: opacity 0.2s;"></div>

<div id="legend">
    <div id="gradient-legend">
        <div class="gradient-bar"></div>
        <div class="percentage-scale">
            <span>-6%</span>
            <span>-2%</span>
            <span>0%</span>
            <span>+2%</span>
            <span>+6%</span>
            <span>NA</span>
        </div>
    </div>
    <div class="frequency-scale">
        <span>Field Goal % vs. All-Star Roster Average</span>
    </div>
</div>

<hr>
<h2>Write Up:</h2>
<p> 
    For our interactive visualization, we decided to analyze the field goal percentage of 
    NBA All-Stars from 6 different zones on a basketball court. Choosing the zones themselves 
    was fairly simple as the distinction was already given to us in the dataset. We chose our 
    visual encoding to be the color of the zones itself which varied based on that particular 
    player's field goal percentage from that zone in comparison to the All-Star average. 
    We considered a variety of different color gradients/patterns but since each zone had 
    its own average field goal percentage, it made more sense to have 5 distinct colors to 
    allow the differences to be seen more clearly. In terms of the interaction techniques, 
    we wanted the user to be able to look at the stats of different players, which are listed 
    on a dropdown menu to make it easy to switch between players, and also showed a tooltip 
    based on the zone the mouse was hovered over. We considered displaying all the data in 
    text form on the side, but we figured that that was harder to read and that the data would 
    stand out more if we displayed the different colors and just showed the particular data of 
    the zone that was hovered over. <br><br>

	As for our development process, we didn't designate specific roles or responsibilities but we all 
    naturally ended up working on what we needed to during that particular time. Minchan ended up 
    doing most of the work with the data, finding all the necessary information/components for 
    each player, and lead the end design/styling decisions. Jason worked on displaying the player information as well 
    as the player pictures and being able to switch between the players. David did most of the work 
    on setting up the zones, making sure the overlay/scale was working correctly, implementing the tooltip correctly, and 
    making the field color appear correctly. We all ended up having to work together to discuss our design 
    decisions as well as ironing out the countless bugs that appeared during the process. In terms 
    of time spent working in total, we probably spent around 40 hours to end up with our finished 
    visualization. The biggest chunk of time came from having to learn how to create our zones so 
    that it lined up with the NBA court properly so we could even display our data in the first place. 
    But once we figured that out, we also had to spend a good amount of time working on fixing bugs and 
    also trying to figure out how to solve an issue that we had when rescaling the window size of the website. 
</p>