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
    
</script>

<main>
    {#if error}
        <p class="error">Error loading data. Please try again later.</p>  
        <!-- // Display an error message if there's an error. -->
    {:else if us && data}
        <!-- Slider for selecting the year -->
        <input type="range" min="2020" max="2023" bind:value={selectedYear} />
        <span>{selectedYear}</span>
        <div class="search-bar">
            <input type="text" bind:value={searchInput} placeholder="Search for a county, state" />
            <button on:click={handleSearch}>Search</button>
            <button on:click={clearSearch}>Clear</button>
        </div>
        <!-- Pass selectedYear to the Graph component -->
        <Graph {us} {data} {selectedYear} {selectedCountyId}/>
        <!-- // Render the Graph component if data is loaded successfully. -->
    {:else}
        <p class="loading">Loading...</p>  
        <!-- // Show a loading message while data is being fetched. -->
    {/if}
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        height: 100vh;  /* // Make the main element cover the entire viewport height. */
        background-color: #f0f0f0;  /* // Set a light grey background color. */
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
        margin-top: 10px;
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
</style>
