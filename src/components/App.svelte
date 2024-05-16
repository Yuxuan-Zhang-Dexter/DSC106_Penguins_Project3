<script>
    import { onMount } from 'svelte';
    import Graph from './Graph.svelte';
    import * as d3 from 'd3';
  
    let us = null;  // This will hold the topoJSON data.
    let error = null;  // To store potential errors during data fetching.
    let selectedYear = 2020;  // Default year or the most recent year with data
    let data = null // This will hold the county population data
    let searchInput = '';  // Search input value
    let selectedCountyId = null;  // The ID of the county to zoom in
    let showOverview = false;  // State to control the visibility of the project overview

    // Fetch the topoJSON data when the component mounts.
    onMount(async () => {  // Make the function async to use await
        try {
            const topoResponse = await fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json');
            if (!topoResponse.ok) {
                throw new Error('Network response was not ok.');
            }
            us = await topoResponse.json();
        } catch (err) {
            console.error('Error loading the topoJSON data:', err);
            error = err;
        }

        try {
            // Correctly use the fetched data
            const populationData = await d3.csv('https://raw.githubusercontent.com/Yuxuan-Zhang-Dexter/DSC106_Penguins_Project3/yuxuan/state_county_population.csv');
            data = populationData
            console.log(data)
        } catch (err) {
            console.error('Error loading the population data:', err);
            error = err;
        }
    });
    function handleSearch() {
        const county = data.find(d => d.county_state.toLowerCase() === searchInput.toLowerCase());
        if (county) {
            selectedCountyId = county.id;
        } else {
            selectedCountyId = null;
        }
    }

    function clearSearch() {
        searchInput = '';
        selectedCountyId = null;
    }

     function toggleOverview() {
        showOverview = !showOverview;
    }
    
</script>


<main>
    <h1>Estimated Population Visualization in the United States from 2020 to 2023</h1>
    {#if error}
        <p class="error">Error loading data. Please try again later.</p>  
    {:else if us && data}
        <div class="controls">
            <!-- Slider for selecting the year -->
            <input type="range" min="2020" max="2023" bind:value={selectedYear} />
            <span>{selectedYear}</span>
        </div>
        <div class="search-bar">
            <input type="text" bind:value={searchInput} placeholder="Search for a county, state" />
            <button on:click={handleSearch}>Search</button>
            <button on:click={clearSearch}>Clear</button>
        </div>
        <!-- Pass selectedYear to the Graph component -->
        <Graph {us} {data} {selectedYear} {selectedCountyId} />
        <!-- Render the Graph component if data is loaded successfully. -->
        <button on:click={toggleOverview}>
            {#if showOverview} Hide Project Overview {/if}
            {#if !showOverview} Show Project Overview {/if}
        </button>
        {#if showOverview}
            <section class="project-overview">
                <h2>Understanding Demographic Changes in the United States</h2>
                <h3>Project Overview</h3>
                <p>We started this project with the goal of understanding demographic changes in the United States. Given the large number of cities, we decided to focus on counties to provide a manageable and insightful level of granularity. Counties strike a balance between the broad overview provided by states and the plethora of details provided by cities, making them ideal for our analysis.</p>
                <h3>Visual Coding</h3>
                <p>For our visual coding, we chose a choropleth map to represent the log population data. This choice allows us to effectively use color intensity to convey the scale population density of different counties. By using the blue scheme, we can clearly distinguish between counties with different scale population density. Dark colors indicate higher scale population density, while light colors indicate lower scale population density. Counties with missing data are shown in black to ensure clarity and accessibility.</p>
                <h3>Data Preprocessing</h3>
                <p>In the data preprocessing stage, the estimated population is calculated by adding the population base with births, subtracting deaths, and adding migration. Since estimated populations are large numbers, we applied a natural logarithm (log base e) to scale the estimated population shown in our visualization, making it easier to see differences among different counties.</p>
                <h3>User Interaction Enhancements</h3>
                <p>To enhance user interaction, we added the following features to the map:</p>
                <ul>
                    <li><strong>Zooming and Tooltips:</strong> Users can zoom in and out of the map, and when the mouse hovers over an area, the log population in the county is displayed, along with the name of the county and the name of the state. This interactive element encourages users to explore the data in greater detail.</li>
                    <li><strong>Search Feature:</strong> Users can search for a specific county by entering the "county, state" order. When a search is performed, the map automatically zooms in and highlights the county in red. Hovering over the highlighted county displays the log population in that county, providing immediate visual feedback. After canceling the search, the map reverts to its original size.</li>
                    <li><strong>Click to Zoom:</strong> Users can click on a county area on the map to zoom into that area. When the mouse hovers over the area, demographic information is displayed. Clicking the same county area again will revert the map to its original display.</li>
                    <li><strong>Time Slider:</strong> We track the change of estimated population from 2020 to 2023. A slider allows users to visualize changes in the population across the whole map over time. By moving the slider, users can see how the scale population density of each county has changed year by year.</li>
                </ul>
                <h3>Considered Alternatives</h3>
                <p>We also considered other visualization methods such as heat maps and bar charts. However, heat maps are less effective at showing discrete regions such as counties and states, and bar charts lack the geographic context we need for our analysis. Ultimately, the interactive terrain map we chose best balances clarity, usability, and detailed geographic representation.</p>
                <h3>Development Process</h3>
                <p>Our development team consisted of three members:</p>
                <ul>
                    <li><strong>Yuxuan Zhang:</strong> Further refined the data cleaning, made it suitable for use on the website, and created the structure of the website. Yuxuan integrated data with visual components, laying the foundation for our project. (16 hours)</li>
                    <li><strong>Tony:</strong> Responsible for data mining and initial data cleaning, ensuring the data was accurate, and displaying our results on GitHub. (14 hours)</li>
                    <li><strong>Yiming Jia:</strong> Involved in the creation of the website and added interactive tools to ensure that the website is visually appealing and user-friendly. (14 hours)</li>
                </ul>
                <p>Throughout the development process, the most time-consuming tasks were data cleaning and ensuring the display of the visualizations.</p>
            </section>
        {/if}
    {:else}
        <p class="loading">Loading...</p>  
        <!-- Show a loading message while data is being fetched. -->
    {/if}
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        height: 100vh;  /* Make the main element cover the entire viewport height. */
        background-color: #f0f0f0;  /* Set a light grey background color. */
    }
    .error, .loading {
        font-size: 16px;
        color: #333;
    }
    .error {
        color: red;
    }
    .controls {
        margin-bottom: 20px;
        text-align: center;
    }
    input[type="range"] {
        width: 300px;
        margin-bottom: 10px;
    }
    span {
        display: block;
        margin-bottom: 10px;
        font-size: 18px;
    }
    .search-bar {
        margin-top: 20px; /* Adjust this value to move the search bar down */
        display: flex;
        gap: 10px;
    }
    input[type="text"] {
        padding: 10px;
        font-size: 16px;
        width: 300px;
    }
    button {
        margin-left: 10px;
        padding: 5px 10px;
        font-size: 14px;
        cursor: pointer;
    }
    .project-overview {
        margin-top: 40px; /* Increase the top margin to create more space above the text */
        padding: 20px;
        background-color: white;
        border-radius: 8px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        max-width: 800px;
        text-align: left;
    }
    .project-overview h2, .project-overview h3 {
        margin-top: 20px;
    }
    .project-overview p, .project-overview ul {
        margin-bottom: 16px;
    }
    .project-overview ul {
        padding-left: 20px;
    }
    .project-overview li {
        margin-bottom: 8px;
    }
</style>