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
    let left_outer = {x: 0, y: 55.8, width: 34, height: 15.8}
    let right_outer = {x: 65.9, y: 55.8, width: 35, height: 15.8}
    let mid_left = {x: 6, y: 40.2, width: 28, height: 28.5}
    let mid_right = {x: 66, y: 40.2, width: 27.8, height: 28.5}
    let mid_middle = { x: 50, y: 14};
    let playershooting = 'src/routes/data/player_stats.json';
    let playerShootingData;



    // Check if document is defined before running client-side code
    if (typeof document !== 'undefined') {
        onMount(() => {
            const container = document.getElementById('container');
            width = container.offsetWidth;
            height = container.offsetHeight;
            svg = d3.select('#overlay');
            svg.on('mousemove', moveTooltip);

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

    function moveTooltip(event) {
    const tooltip = document.getElementById('tooltip');
    const fill = event.target.getAttribute('fill');

    if (fill === 'purple' || fill === 'yellow') {
        tooltip.style.display = 'block';
        tooltip.style.opacity = 1;
        tooltip.style.left = `${event.pageX + 10}px`;
        tooltip.style.top = `${event.pageY + 10}px`;

        // You may want to modify this to set the tooltip text based on the fill
        if (fill === 'purple') {
            // Set tooltip for purple (Above the Break 3)
            const data = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === "Above the Break 3");
            showTooltip(data);
        } else if (fill === 'yellow') {
            // Set tooltip for yellow (Mid-Range)
            const data = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === "Mid-Range");
            showTooltip(data);
        }
    } else {
        hideTooltip();
    }
}


function showTooltip(data) {
    const tooltip = document.getElementById('tooltip');
    tooltip.innerHTML = `FG Made: ${data.sum}<br>` +`FG Attempted: ${data.count}<br>` +
                        `FG%: ${String(Math.round(data.mean * 100 * 100) / 100)+ '%'}`;
}

    function hideTooltip() {
        const tooltip = document.getElementById('tooltip');
        tooltip.style.opacity = 0;
        setTimeout(() => { tooltip.style.display = 'none'; }, 300);
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
            svg.append('rect')
                .attr('x', xScale(middle_outer.x))
                .attr('y', yScale(middle_outer.y))
                .attr('width', xScale(middle_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(middle_outer.height))
                .attr('fill', 'purple')
                .attr('fill-opacity', '1')
                .on('mouseover', () => showTooltip(event, data))
                .on('mouseout', hideTooltip);
            svg.append('rect')
                .attr('x', xScale(left_outer.x))
                .attr('y', yScale(left_outer.y))
                .attr('width', xScale(left_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(left_outer.height))
                .attr('fill', 'purple')
                .attr('fill-opacity', '1')
                .on('mouseover', () => showTooltip(event, data))
                .on('mouseout', hideTooltip);
                console.log('Appended tooltip for rectangle:', tooltipText); 
            svg.append('rect')
                .attr('x', xScale(right_outer.x))
                .attr('y', yScale(right_outer.y))
                .attr('width', xScale(right_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(right_outer.height))
                .attr('fill', 'purple')
                .attr('fill-opacity', '1')
                .on('mouseover', () => showTooltip(event, data))
                .on('mouseout', hideTooltip);
        });
        let mid_range = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Mid-Range")
        mid_range.forEach(data => {
            svg.append('rect')
            .attr('x', xScale(mid_left.x))
            .attr('y', yScale(mid_left.y))
            .attr('width', xScale(mid_left.width) - xScale(0))
            .attr('height', yScale(0) - yScale(mid_left.height))
            .attr('fill', 'yellow')
            .attr('fill-opacity', '1')
            .on('mouseover', () => showTooltip(event, data))
            .on('mouseout', hideTooltip);
            svg.append('rect')
                .attr('x', xScale(mid_right.x))
                .attr('y', yScale(mid_right.y))
                .attr('width', xScale(mid_right.width) - xScale(0))
                .attr('height', yScale(0) - yScale(mid_right.height))
                .attr('fill', 'yellow')
                .attr('fill-opacity', '1')
                .on('mouseover', () => showTooltip(event, data))
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
                .attr('fill', 'yellow')
                .attr('fill-opacity', '1')
                .on('mouseover', () => showTooltip(event, data))
                .on('mouseout', hideTooltip);
        });

        svg.append('rect')
            .attr('x', xScale(leftcornerregion.x))
            .attr('y', yScale(leftcornerregion.y))
            .attr('width', xScale(leftcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(leftcornerregion.height))
            .attr('fill', 'blue')
            .attr('fill-opacity', '1');

        svg.append('rect')
            .attr('x', xScale(rightcornerregion.x))
            .attr('y', yScale(rightcornerregion.y))
            .attr('width', xScale(rightcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(rightcornerregion.height))
            .attr('fill', 'red')
            .attr('fill-opacity', '1');

        svg.append('rect')
            .attr('x', xScale(paintregion.x))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(paintregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(paintregion.height))
            .attr('fill', 'green')
            .attr('fill-opacity', '1');
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
            .attr('fill-opacity', '1');

        svg.append('rect')
            .attr('x', xScale(restrictedregion_rec.x))
            .attr('y', yScale(restrictedregion_rec.y))
            .attr('width', xScale(restrictedregion_rec.width) - xScale(0))
            .attr('height', yScale(0) - yScale(restrictedregion_rec.height))
            .attr('fill', 'orange')
            .attr('fill-opacity', '1');
        
    
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

<style>
    #container {
    position: relative;
    width: 50%;
    margin-left: 0%;
    margin-bottom: 30px;
}

    img {
        display: block;
        width: 100%;
        height: auto;
        left: 500px;
    }
    #overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    #player-info {
    position: absolute;
    top: 0;
    left: 650px;
    text-align: left;
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
    #tooltip{
        background: white;
        border: 1px solid #ccc;
        padding: 10px;
        border-radius: 5px;
        display: none; /* Hidden by default */
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

