<script>
    import { onMount } from 'svelte';
    import Graph from './Graph.svelte';
  
    let us = null;  // This will hold the topoJSON data.
    let error = null;  // To store potential errors during data fetching.

    // Fetch the topoJSON data when the component mounts.
    onMount(() => {
        fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json')
        .then(response => {
            if (!response.ok) {
                throw new Error('Network response was not ok.'); // Handle non-OK responses by throwing an error.
            }
            return response.json(); // Parse the JSON data from the response.
        })
        .then(data => {
            us = data;  // Assign the fetched data to the 'us' variable.
        })
        .catch(err => {
            console.error('Error loading the topoJSON data:', err);  // Log errors to the console.
            error = err;  // Store the error to display in the UI.
        });
    });
</script>

<main>
    {#if error}
        <p class="error">Error loading data. Please try again later.</p>  
        <!-- // Display an error message if there's an error. -->
    {:else if us}
        <Graph {us}/>  
        <!-- // Render the Graph component if data is loaded successfully. -->
    {:else}
        <p class="loading">Loading...</p>  
        <!-- // Show a loading message while data is being fetched. -->
    {/if}
</main>

<style>
    main {
        display: flex;
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
</style>
