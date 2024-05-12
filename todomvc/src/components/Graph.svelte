<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from 'topojson-client';

    export let us;
    export let data; // Ensure data is ready for future use

    let svgElement;
    let tooltipElement; // Change this to match Svelte binding

    // add legend function 
    function Legend(color, {
        title,
        tickSize = 6,
        width = 320,
        height = 44 + tickSize,
        marginTop = 18,
        marginRight = 0,
        marginBottom = 16 + tickSize,
        marginLeft = 0,
        ticks = width / 64,
        tickFormat,
        tickValues
    } = {}) {

        function ramp(color, n = 256) {
            const canvas = document.createElement("canvas");
            canvas.width = n;
            canvas.height = 1;
            const context = canvas.getContext("2d");
            for (let i = 0; i < n; ++i) {
                context.fillStyle = color(i / (n - 1));
                context.fillRect(i, 0, 1, 1);
            }
            return canvas;
        }

        const svg = d3.create("svg")
            .attr("width", width)
            .attr("height", height)
            .attr("viewBox", [0, 0, width, height])
            .style("overflow", "visible")
            .style("display", "block");

        let tickAdjust = g => g.selectAll(".tick line").attr("y1", marginTop + marginBottom - height);
        let x;

        if (color.interpolate) {
            const n = Math.min(color.domain().length, color.range().length);
            x = color.copy().rangeRound(d3.quantize(d3.interpolate(marginLeft, width - marginRight), n));
            svg.append("image")
                .attr("x", marginLeft)
                .attr("y", marginTop)
                .attr("width", width - marginLeft - marginRight)
                .attr("height", height - marginTop - marginBottom)
                .attr("preserveAspectRatio", "none")
                .attr("xlink:href", ramp(color.copy().domain(d3.quantize(d3.interpolate(0, 1), n))).toDataURL());
        } else if (color.interpolator) {
            x = Object.assign(color.copy().interpolator(d3.interpolateRound(marginLeft, width - marginRight)), {
                range() {
                    return [marginLeft, width - marginRight];
                }
            });

            svg.append("image")
                .attr("x", marginLeft)
                .attr("y", marginTop)
                .attr("width", width - marginLeft - marginRight)
                .attr("height", height - marginTop - marginBottom)
                .attr("preserveAspectRatio", "none")
                .attr("xlink:href", ramp(color.interpolator()).toDataURL());
        } else if (color.invertExtent) {
            const thresholds = color.thresholds ? color.thresholds() :
                color.quantiles ? color.quantiles() :
                color.domain();

            const thresholdFormat = tickFormat === undefined ? d => d :
                typeof tickFormat === "string" ? d3.format(tickFormat) :
                tickFormat;

            x = d3.scaleLinear()
                .domain([-1, color.range().length - 1])
                .rangeRound([marginLeft, width - marginRight]);

            svg.append("g")
                .selectAll("rect")
                .data(color.range())
                .join("rect")
                .attr("x", (d, i) => x(i - 1))
                .attr("y", marginTop)
                .attr("width", (d, i) => x(i) - x(i - 1))
                .attr("height", height - marginTop - marginBottom)
                .attr("fill", d => d);

            tickValues = d3.range(thresholds.length);
            tickFormat = i => thresholdFormat(thresholds[i], i);
        } else {
            x = d3.scaleBand()
                .domain(color.domain())
                .rangeRound([marginLeft, width - marginRight]);

            svg.append("g")
                .selectAll("rect")
                .data(color.domain())
                .join("rect")
                .attr("x", x)
                .attr("y", marginTop)
                .attr("width", Math.max(0, x.bandwidth() - 1))
                .attr("height", height - marginTop - marginBottom)
                .attr("fill", color);

            tickAdjust = () => {};
        }

        svg.append("g")
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(d3.axisBottom(x)
                .ticks(ticks, typeof tickFormat === "string" ? tickFormat : undefined)
                .tickFormat(typeof tickFormat === "function" ? tickFormat : undefined)
                .tickSize(tickSize)
                .tickValues(tickValues))
            .call(tickAdjust)
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", marginLeft)
                .attr("y", marginTop + marginBottom - height - 6)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .attr("font-weight", "bold")
                .text(title));

        return svg.node();
    }

    // add createChart function

    function createChart() {
        if (!us || !svgElement || !tooltipElement) return; // Ensure everything is available

        const svg = d3.select(svgElement);
        const tooltip = d3.select(tooltipElement);

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

        // Draw legend
        svg.append("g")
            .attr("transform", "translate(610,20)")
            .append(() => Legend(color, {title: "Estimated Population", width: 260}));

        // Draw counties
        svg.append("g")
            .selectAll("path")
            .data(counties.features)
            .join("path")
                .attr("fill", d => color(valuemap.get(d.id)))
                .attr("d", path)
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
                .text(d => `${d.properties.name}, ${d.properties.name}\n${valuemap.get(d.id)}`);

        // Draw state borders
        svg.append("path")
            .datum(statemesh)
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
