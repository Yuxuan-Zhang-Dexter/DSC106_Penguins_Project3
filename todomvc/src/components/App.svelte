<script>
    import { onMount } from 'svelte';
    import Graph from './Graph.svelte';
    import * as d3 from 'd3';
  
    let us = null;  // This will hold the topoJSON data.
    let error = null;  // To store potential errors during data fetching.
    let selectedYear = 2020;  // Default year or the most recent year with data
    let data = null // This will hold the county population data

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
    function incrementYear() {
        if (selectedYear < 2023) {
            selectedYear += 1;
        }
    }

    function decrementYear() {
        if (selectedYear > 2020) {
        selectedYear -= 1;
        }
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
        <!-- Pass selectedYear to the Graph component -->
        <Graph {us} {data} {selectedYear}/>
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
        color: #333;  /* // Set dark grey text for loading and error messages. */
    }
    .error {
        color: red;  /* // Highlight errors in red. */
    }
    button {
        margin-left: 10px;
        padding: 5px 10px;
        font-size: 14px;
        cursor: pointer;
    }
</style>
