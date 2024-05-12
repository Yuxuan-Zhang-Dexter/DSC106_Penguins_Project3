<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from 'topojson-client';

    export let us;
    export let data; // Ensure data is ready for future use

    let svgElement;
    let tooltipElement; // Change this to match Svelte binding

    function createChart() {
        if (!us || !svgElement || !tooltipElement) return; // Ensure everything is available

        // Example color scale, adjust according to your data properties
        // const color = d3.scaleQuantize([1, 10], d3.schemeBlues[9]);

        const svg = d3.select(svgElement);
        const tooltip = d3.select(tooltipElement); // Select the tooltip correctly with D3

        svg.selectAll("*").remove(); // Clear previous SVG contents

        const color = d3.scaleQuantize([1, 10], d3.schemeBlues[9]);
        const path = d3.geoPath();
        const valuemap = new Map(data.map(d => [d.id, d['2020']]));

        const counties = topojson.feature(us, us.objects.counties);
        const states = topojson.feature(us, us.objects.states);
        const statemap = new Map(states.features.map(d => [d.id, d]));
        const statemesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);

        const zoom = d3.zoom()
            .scaleExtent([1, 8])
            .on('zoom', (event) => svg.selectAll('path').attr('transform', event.transform));
        svg.call(zoom);

        // Draw counties
        svg.append("g")
            .selectAll("path")
            .data(counties.features)
            .join("path")
                .attr("fill", d => color(valuemap.get(d.id)))
                .attr("d", path)
                .attr("fill", "#ccc") // Replace this with your color scale logic
                .on('mouseover', (event, d) => {
                    tooltip
                        .style("visibility", "visible")
                        .text(`County: ${d.properties.name}`);
                })
                .on('mousemove', (event) => {
                    tooltip
                        .style("top", `${event.pageY - 10}px`)
                        .style("left", `${event.pageX + 10}px`);
                })
                .on('mouseout', () => {
                    tooltip.style("visibility", "hidden");
                })
            .append("title")
                .text(d => `${d.properties.name}, ${statemap.get(d.id.slice(0, 2)).properties.name}\n${valuemap.get(d.id)}%`);

        // Draw state borders
        svg.append("path")
            .datum(topojson.mesh(us, us.objects.states, (a, b) => a !== b))
            .attr("fill", "none")
            .attr("stroke", "white")
            .attr("stroke-linejoin", "round")
            .attr("d", path);
    }

    onMount(createChart);
    $: us, data, createChart(); // Reactive update if 'us' or 'data' changes
</script>

<svg bind:this={svgElement}></svg>
<div bind:this={tooltipElement} class="tooltip" style="position: absolute; visibility: hidden; pointer-events: none; background-color: white; padding: 5px; border: 1px solid black;"></div>



<div>Hello from Graph.svelte!</div>

<style>
    /* Styling for any additional div elements inside the component */
    div:not(.tooltip) {
        color: blue;
        font-size: 20px;
        padding: 10px;
        margin: 20px;
        border: 1px solid blue;
        border-radius: 5px;
    }

    /* Ensure the SVG element fills its container and is responsive */
    svg {
        width: 100%;
        height: 100%;  /* Set height to 100% to fill the container */
        display: block;  /* Remove any extra space below the SVG */
    }

    /* Styling for the tooltip */
    .tooltip {
        position: absolute;  /* Change to 'absolute' to better follow mouse movement within the SVG */
        visibility: hidden;  /* Start with the tooltip hidden */
        text-align: center;
        width: auto;
        height: auto;
        background-color: white;  /* Ensure background is opaque for readability */
        border: 1px solid #d4d4d4;
        border-radius: 3px;
        pointer-events: none;  /* Ensure tooltip does not interfere with mouse events */
        padding: 5px;
        font-size: 14px;
        color: black;
        box-shadow: 0 2px 6px rgba(0,0,0,0.1);  /* Optional: Add shadow for better visibility */
        z-index: 100;  /* Ensure tooltip is above all other content */
        transition: opacity 0.3s;  /* Smooth transition for tooltip visibility */
    }
</style>
