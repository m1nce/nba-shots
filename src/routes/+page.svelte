<h1>The Hottest NBA Players</h1><head><link href='https://fonts.googleapis.com/css?family=Playfair+Display' rel='stylesheet'></head>
<p>Based on players that were in the 2022 NBA All Star Draft, we see which players shoot better from six different zones. The redder the hotter and better they are in comparison to the roster average!</p>
<script>
    import { onMount, onDestroy, afterUpdate } from 'svelte';
    import pkg from 'lodash';
    const { debounce } = pkg;
    import * as d3 from 'd3';
    let svg;
    let showPlayerInfo = true;
    let selectedPlayer = "LeBron James";
    let playerImages = {
        "LeBron James": 'src/routes/data/photos/lebronjames23.jpeg',
        "Kevin Durant": 'src/routes/data/photos/kevindurant35.jpeg',
        "Giannis Antetokounmpo": 'src/routes/data/photos/giannis34.jpeg',
        "Joel Embiid": 'src/routes/data/photos/joelembiid21.jpeg',
        "Stephen Curry": 'src/routes/data/photos/stephencurry30.jpeg',
        "Ja Morant": 'src/routes/data/photos/jamorant12.jpeg',
        "DeMar DeRozan": 'src/routes/data/photos/demarderozan11.jpeg',
        "Jayson Tatum": 'src/routes/data/photos/jaysontatum0.jpeg',
        "Nikola Jokic": 'src/routes/data/photos/nikolajokic15.jpeg',
        "Andrew Wiggins": 'src/routes/data/photos/andrewwiggins22.jpeg',
        "Trae Young": 'src/routes/data/photos/traeyoung11.jpeg',
        "Devin Booker": 'src/routes/data/photos/devinbooker1.jpeg',
        "Luka Doncic": 'src/routes/data/photos/lukadoncic77.jpeg',
        "Karl-Anthony Towns": 'src/routes/data/photos/kat32.jpeg',
        "Darius Garland": 'src/routes/data/photos/dariusgarland10.jpeg',
        "Zach LaVine": 'src/routes/data/photos/zachlavine8.jpeg',
        "Chris Paul": 'src/routes/data/photos/chrispaul3.jpeg',
        "Dejounte Murray": 'src/routes/data/photos/dejountemurray5.jpeg',
        "Jimmy Butler": 'src/routes/data/photos/jimmybutler22.jpeg',
        "Khris Middleton": 'src/routes/data/photos/khrismiddleton22.jpeg',
        "Donovan Mitchell": 'src/routes/data/photos/donovanmitchell45.jpeg',
        "LaMelo Ball": 'src/routes/data/photos/lameloball1.jpeg',
        "Fred VanVleet": 'src/routes/data/photos/fredvanvleet5.jpeg',
        "Rudy Gobert": 'src/routes/data/photos/rudygobert27.jpeg',
        "James Harden": 'src/routes/data/photos/jamesharden1.jpeg'
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
    let playershooting = 'src/routes/data/player_stats.json';
    let playerShootingData;

    let averages = {"Above the Break 3":0.3641223979,"In The Paint (Non-RA)":0.4604816551,"Left Corner 3":0.3847352025,"Mid-Range":0.4461170031,"Restricted Area":0.6779065226,"Right Corner 3":0.3843351548}
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

    function handleResize() {
        const container = document.getElementById('container');
        width = container.offsetWidth;
        height = container.offsetHeight;
        renderSVG();
    }




    function showTooltip(data, event) {
    const tooltip = document.getElementById('tooltip');
    tooltip.innerHTML = `<b>${data.BASIC_ZONE}</b>:<br>` + 
                        `FG Made: ${data.sum}<br>` +
                        `FG Attempted: ${data.count}<br>` +
                        `FG%: ${String(Math.round(data.mean * 100 * 100) / 100) + '%'}`;
    
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
        let aboveTheBreak3Data = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Above the Break 3")
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
        let mid_range = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Mid-Range")
        mid_range.forEach(data => {
            let fillcolor = 'black'
            console.log(data.mean - averages[data.BASIC_ZONE])
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                console.log('shit')
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
        let left_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Left Corner 3")
        left_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                console.log(data.mean)
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                console.log(data.mean)
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
        let right_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Right Corner 3")
        right_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                console.log(data.mean)
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                console.log(data.mean)
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
        let dapaint = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "In The Paint (Non-RA)")
        dapaint.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                console.log(data.mean)
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                console.log(data.mean)
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
        let dara = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Restricted Area")
        dara.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                console.log(data.mean)
                fillcolor = '#5b52d5';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                console.log(data.mean)
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
        let bottomcornerregion = { x: 0, y:12, width: 100, height: 20};

        svg.append('rect')
            .attr('x', xScale(bottomcornerregion.x))
            .attr('y', yScale(bottomcornerregion.y))
            .attr('width', xScale(bottomcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(bottomcornerregion.height))
            .attr('fill', 'white')
            .attr('fill-opacity', '1');
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

        // Calculate the start and end points of the arc
        let startX = outline_radius * Math.cos(startAngle);
        let startY = outline_radius * Math.sin(startAngle);
        let endX = outline_radius * Math.cos(endAngle);
        let endY = outline_radius * Math.sin(endAngle);

        // Create the arc path manually
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

<style>
    #container {
    position: relative;
    width: 50%;
    margin-left: 0%;
    margin-bottom: 30px;
}
h1 {
    text-align:center;
    font-family: 'Playfair Display';font-size: 22px;
    font-size: 45px;
    margin-top: 10px;
    margin-bottom: 5px;
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
        width: 263px; /* Adjust the size as needed */
        height: 192px; /* Adjust the size as needed */
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
    font-family: Arial, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: left;
    margin-top: 20px;
}

#gradient-legend {
    position: relative;
    margin-bottom: 10px;
}
p {
    text-align:center
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
    <img src='src/nbacourt.jpg' alt='Basketball Court' width="500" height="500">
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

