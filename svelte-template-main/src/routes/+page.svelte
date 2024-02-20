<script>
    import { onMount, onDestroy, afterUpdate } from 'svelte';
    import pkg from 'lodash';
    const { debounce } = pkg;
    import * as d3 from 'd3';

    let width = 0;
    let height = 0;
    let leftcornerregion = { x: 0, y: 40.2, width: 6, height: 100 };
    let rightcornerregion = { x: 93.9, y: 40.2, width: 6.5, height: 100 };
    let paintregion = { x: 33.9, y: 56, width: 32, height: 44.2 };
    let restrictedregion = { x: 49.9, y: 14.5 };
    let restrictedregion_rec = { x: 42, y: 14.5, width: 15.65, height: 2.4 };

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

        afterUpdate(() => {
            playerImageSrc = playerImages[selectedPlayer];
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

    // Handle dropdown change event
    function handleDropdownChange(event) {
        // Get the selected option value
        const selectedOption = event.target.value;
        // Get the corresponding photo filename from the mapping object
        selectedPhoto = optionPhotoMap[selectedOption];
    }
</script>

<style>
    #container {
        position: relative;
        width: 100%;
        margin-bottom: 350px; /* Adjust the padding as needed */
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

    #player-info {
        position: absolute;
        bottom: -300px;
        left: 50%;
        transform: translateX(-50%);
        text-align: center;
        color: white;
        font-family: Arial, sans-serif;
    }

    #player-image {
        width: 263px; /* Adjust the size as needed */
        height: 192px; /* Adjust the size as needed */
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 20px;
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
        <select id="playerDropdown" bind:value={selectedPlayer}>
            <option value="LeBron James">LeBron James</option>
            <option value="Kevin Durant">Kevin Durant</option>
            <option value="Giannis Antetokounmpo">Giannis Antetokounmpo</option>
            <option value="Joel Embiid">Joel Embiid</option>
            <option value="Stephen Curry">Stephen Curry</option>
            <option value="Ja Morant">Ja Morant</option>
            <option value="DeMar DeRozan">DeMar DeRozan</option>
            <option value="Jayson Tatum">Jayson Tatum</option>
            <option value="Nikola Jokic">Nikola Jokic</option>
            <option value="Andrew Wiggins">Andrew Wiggins</option>
            <option value="Trae Young">Trae Young</option>
            <option value="Devin Booker">Devin Booker</option>
            <option value="Luka Doncic">Luka Doncic</option>
            <option value="Karl-Anthony Towns">Karl-Anthony Towns</option>
            <option value="Darius Garland">Darius Garland</option>
            <option value="Zach LaVine">Zach LaVine</option>
            <option value="Chris Paul">Chris Paul</option>
            <option value="Dejounte Murray">Dejounte Murray</option>
            <option value="Jimmy Butler">Jimmy Butler</option>
            <option value="Khris Middleton">Khris Middleton</option>
            <option value="Donovan Mitchell">Donovan Mitchell</option>
            <option value="LaMelo Ball">LaMelo Ball</option>
            <option value="Fred VanVleet">Fred VanVleet</option>
            <option value="Rudy Gobert">Rudy Gobert</option>
            <option value="James Harden">James Harden</option>
        </select>
    </div>
    {/if}
</div>
