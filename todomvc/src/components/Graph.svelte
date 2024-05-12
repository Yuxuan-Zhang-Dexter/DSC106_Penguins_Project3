<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from 'topojson-client';
	// import {Legend} from "@d3/color-legend";

    export let us;

    let svgElement;

    function createChart() {
        if (!us || !svgElement) return; // Ensure us data and svg element are available

        const svg = d3.select(svgElement);
        svg.selectAll("*").remove(); // Clear previous SVG contents

        const path = d3.geoPath();

        const counties = topojson.feature(us, us.objects.counties);
        const states = topojson.feature(us, us.objects.states);

        // Draw counties
        svg.append("g")
            .selectAll("path")
            .data(counties.features)
            .join("path")
                .attr("fill", "#ccc")
                .attr("d", path);

        // Draw state borders
        svg.append("path")
            .datum(topojson.mesh(us, us.objects.states, (a, b) => a !== b))
            .attr("fill", "none")
            .attr("stroke", "white")
            .attr("stroke-linejoin", "round")
            .attr("d", path);
    }

    onMount(() => {
        createChart();
    });

    $: us, createChart(); // Reactive update if us changes
</script>

<svg bind:this={svgElement}></svg>

<div>Hello from Graph.svelte!</div>

<style>
    div {
        color: blue;
        font-size: 20px;
        padding: 10px;
        margin: 20px;
        border: 1px solid blue;
        border-radius: 5px;
    }
    svg {
        width: 100%;  
		/* // Ensure SVG fills the container */
        height: 100%;
    }
</style>
