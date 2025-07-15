<script setup>
import * as d3 from 'd3';
import { ref, onMounted, watch, onUnmounted } from 'vue';

// Define props for all configurable aspects of the scale
const props = defineProps({
    value: {
        type: Number,
        default: 0
    },
    indicatorSize: {
        type: Number,
        default: 20
    },
    indicatorColor: {
        type: String,
        default: '#ef4444'
    },
    indicatorOpacity: {
        type: Number,
        default: 1.0
    },
    indicatorDistancePercent: {
        type: Number,
        default: 5
    },
    confidenceRangePercent: {
        type: Number,
        default: 5
    },
    confidenceOpacity: {
        type: Number,
        default: 0.6
    },
    confidenceColor: {
        type: String,
        default: '#a7f3d0'
    },
    confidenceBoxCrossDimension: {
        type: Number,
        default: 20
    },
    transitionDuration: {
        type: Number,
        default: 0.3
    },
    orientation: {
        type: String,
        default: 'horizontal' // 'horizontal' or 'vertical'
    },
    scalePadding: {
        type: Number,
        default: 50
    },
    percentCenter: {
        type: Number,
        default: 40
    },
    percentMid: {
        type: Number,
        default: 40
    },
    percentOuter: {
        type: Number,
        default: 20
    },
    domainBreakpoints: { // New prop for domain breakpoints
        type: Array,
        default: () => [-10, -5, -1, 0, 1, 5, 10]
    }
});

// Ref for the SVG element, allowing D3 to select it
const svgRef = ref(null);

// Reactive dimensions for the SVG, will be updated by ResizeObserver
const svgWidth = ref(900);
const svgHeight = ref(200);

let scale; // D3 scale instance
let resizeObserver; // To observe container resizing

/**
 * Creates a custom piecewise linear scale function based on provided percentages.
 * This scale maps values from -10 to 10 onto a pixel range,
 * with specific segments occupying different proportions of the total width/height.
 * @param {number} totalDimension The total dimension of the SVG (width for horizontal, height for vertical).
 * @param {number[]} domainBreakpoints The array of domain values that define the segments (e.g., [-10, -5, -1, 0, 1, 5, 10]).
 * @param {number} p_center Percentage for the [-1, 1] range (e.g., 0.4 for 40%).
 * @param {number} p_mid Percentage for the [-5,-1] and [1,5] ranges (e.g., 0.4 for 40%).
 * @param {number} p_outer Percentage for the [-10,-5] and [5,10] ranges (e.g., 0.2 for 20%).
 * @param {string} orientation 'horizontal' or 'vertical'.
 * @param {number} padding The padding in pixels at the ends of the scale.
 * @returns {function} A function that maps a domain value to a pixel position.
 */
function createCustomScale(totalDimension, domainBreakpoints, p_center, p_mid, p_outer, orientation, padding) {
    if (domainBreakpoints.length !== 7 || !domainBreakpoints.includes(0)) {
        console.error("Invalid domainBreakpoints array. Expected [-10, -5, -1, 0, 1, 5, 10] or similar 7-point array including 0.");
        // Fallback to a default linear scale or handle error gracefully
        return d3.scaleLinear().domain([-10, 10]).range(orientation === 'horizontal' ? [padding, totalDimension - padding] : [totalDimension - padding, padding]);
    }

    // Ensure breakpoints are sorted for correct segment calculation
    const sortedBreakpoints = [...domainBreakpoints].sort((a, b) => a - b);

    let startPx, endPx, effectiveLength;

    if (orientation === 'horizontal') {
        startPx = padding;
        endPx = totalDimension - padding;
        effectiveLength = endPx - startPx;
    } else { // vertical
        startPx = totalDimension - padding; // Scale starts from bottom (higher pixel value)
        endPx = padding; // Scale ends at top (lower pixel value)
        effectiveLength = startPx - endPx;
    }

    // Calculate pixel lengths for each segment based on percentages
    // These percentages are applied to the defined segments based on the sorted breakpoints
    const segmentPixelLengths = {
        'center': p_center * effectiveLength, // For [-1, 1]
        'mid': (p_mid / 2) * effectiveLength, // For [1,5] and [-5,-1]
        'outer': (p_outer / 2) * effectiveLength // For [5,10] and [-10,-5]
    };

    const rangePoints = {};
    // Map domain breakpoints to pixel positions
    // Assuming domainBreakpoints are ordered: [-10, -5, -1, 0, 1, 5, 10]
    // Index mapping: 0: -10, 1: -5, 2: -1, 3: 0, 4: 1, 5: 5, 6: 10

    // Start point of the scale (corresponding to the smallest domain breakpoint)
    rangePoints[sortedBreakpoints[0]] = orientation === 'horizontal' ? startPx : startPx;

    // Calculate pixel positions sequentially
    rangePoints[sortedBreakpoints[1]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[0]] + segmentPixelLengths.outer :
        rangePoints[sortedBreakpoints[0]] - segmentPixelLengths.outer;

    rangePoints[sortedBreakpoints[2]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[1]] + segmentPixelLengths.mid :
        rangePoints[sortedBreakpoints[1]] - segmentPixelLengths.mid;

    rangePoints[sortedBreakpoints[3]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[2]] + (segmentPixelLengths.center / 2) :
        rangePoints[sortedBreakpoints[2]] - (segmentPixelLengths.center / 2);

    rangePoints[sortedBreakpoints[4]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[3]] + (segmentPixelLengths.center / 2) :
        rangePoints[sortedBreakpoints[3]] - (segmentPixelLengths.center / 2);

    rangePoints[sortedBreakpoints[5]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[4]] + segmentPixelLengths.mid :
        rangePoints[sortedBreakpoints[4]] - segmentPixelLengths.mid;

    rangePoints[sortedBreakpoints[6]] = orientation === 'horizontal' ?
        rangePoints[sortedBreakpoints[5]] + segmentPixelLengths.outer :
        rangePoints[sortedBreakpoints[5]] - segmentPixelLengths.outer;


    const customScale = (value) => {
        if (value >= 0) {
            if (value <= sortedBreakpoints[4]) { // value <= 1
                return d3.scaleLinear().domain([sortedBreakpoints[3], sortedBreakpoints[4]]).range([rangePoints[sortedBreakpoints[3]], rangePoints[sortedBreakpoints[4]]])(value);
            } else if (value <= sortedBreakpoints[5]) { // value <= 5
                return d3.scaleLinear().domain([sortedBreakpoints[4], sortedBreakpoints[5]]).range([rangePoints[sortedBreakpoints[4]], rangePoints[sortedBreakpoints[5]]])(value);
            } else if (value <= sortedBreakpoints[6]) { // value <= 10
                return d3.scaleLinear().domain([sortedBreakpoints[5], sortedBreakpoints[6]]).range([rangePoints[sortedBreakpoints[5]], rangePoints[sortedBreakpoints[6]]])(value);
            }
        } else { // value < 0
            if (value >= sortedBreakpoints[2]) { // value >= -1
                return d3.scaleLinear().domain([sortedBreakpoints[2], sortedBreakpoints[3]]).range([rangePoints[sortedBreakpoints[2]], rangePoints[sortedBreakpoints[3]]])(value);
            } else if (value >= sortedBreakpoints[1]) { // value >= -5
                return d3.scaleLinear().domain([sortedBreakpoints[1], sortedBreakpoints[2]]).range([rangePoints[sortedBreakpoints[1]], rangePoints[sortedBreakpoints[2]]])(value);
            } else if (value >= sortedBreakpoints[0]) { // value >= -10
                return d3.scaleLinear().domain([sortedBreakpoints[0], sortedBreakpoints[1]]).range([rangePoints[sortedBreakpoints[0]], rangePoints[sortedBreakpoints[1]]])(value);
            }
        }
        return rangePoints[sortedBreakpoints[3]]; // Fallback for 0
    };

    customScale.domain = () => [sortedBreakpoints[0], sortedBreakpoints[6]]; // Overall domain
    customScale.range = () => (orientation === 'horizontal' ? [rangePoints[sortedBreakpoints[0]], rangePoints[sortedBreakpoints[6]]] : [rangePoints[sortedBreakpoints[6]], rangePoints[sortedBreakpoints[0]]]);
    customScale.effectiveLength = effectiveLength;
    return customScale;
}

/**
 * Updates the position of the indicator triangle and the confidence range box.
 */
function updateIndicatorAndConfidence() {
    if (!scale || !svgRef.value) {
        console.warn("updateIndicatorAndConfidence: Scale or SVG not initialized.");
        return; // Ensure scale and SVG are initialized
    }

    const pos = scale(props.value);
    const effectiveLength = scale.effectiveLength;
    const confidencePx = (props.confidenceRangePercent / 100) * effectiveLength;

    const d3svg = d3.select(svgRef.value);

    // Apply transition duration
    d3svg.selectAll(".indicator-triangle, .confidence-box")
        .style("transition", `all ${props.transitionDuration}s ease-out`);

    // Update indicator position
    d3svg.select(".indicator-triangle")
        .attr("transform", props.orientation === 'horizontal' ?
            `translate(${pos}, 0)` :
            `translate(0, ${pos})`)
        .style("fill", props.indicatorColor)
        .style("fill-opacity", props.indicatorOpacity);

    // Update confidence box
    d3svg.select(".confidence-box")
        .style("fill", props.confidenceColor)
        .style("fill-opacity", props.confidenceOpacity);

    if (props.orientation === 'horizontal') {
        d3svg.select(".confidence-box")
            .attr("x", pos - confidencePx / 2)
            .attr("y", svgHeight.value / 2 - (props.confidenceBoxCrossDimension / 2))
            .attr("width", confidencePx)
            .attr("height", props.confidenceBoxCrossDimension);
    } else { // vertical
        d3svg.select(".confidence-box")
            .attr("x", svgWidth.value / 2 - (props.confidenceBoxCrossDimension / 2))
            .attr("y", pos - confidencePx / 2)
            .attr("width", props.confidenceBoxCrossDimension)
            .attr("height", confidencePx);
    }
}

/**
 * Draws or redraws the entire scale based on current props and dynamic dimensions.
 */
function drawScale() {
    if (!svgRef.value) {
        console.error("drawScale: SVG element is null. Cannot draw scale.");
        return; // Ensure SVG element is available
    }

    console.log("Drawing scale. Orientation:", props.orientation);
    console.log("Current Padding:", props.scalePadding);
    console.log("Percentages:", props.percentCenter, props.percentMid, props.percentOuter);
    console.log("SVG Dimensions:", svgWidth.value, svgHeight.value);
    console.log("Domain Breakpoints:", props.domainBreakpoints);


    const d3svg = d3.select(svgRef.value);
    d3svg.selectAll("*").remove(); // Clear previous SVG content

    let scaleLinePos;
    let tickTextOffset;
    let indicatorPoints;

    const indicatorSize = props.indicatorSize;
    const indicatorColor = props.indicatorColor;
    const indicatorOpacity = props.indicatorOpacity;
    const indicatorDistancePercent = props.indicatorDistancePercent;
    const confidenceOpacity = props.confidenceOpacity;
    const confidenceColor = props.confidenceColor;
    const confidenceBoxCrossDimension = props.confidenceBoxCrossDimension;
    const transitionDuration = props.transitionDuration;

    // Set SVG dimensions and viewBox based on orientation and current size
    if (props.orientation === 'horizontal') {
        d3svg.attr("width", svgWidth.value)
            .attr("height", svgHeight.value)
            .attr("viewBox", `0 0 ${svgWidth.value} ${svgHeight.value}`);
        scale = createCustomScale(svgWidth.value, props.domainBreakpoints, props.percentCenter / 100, props.percentMid / 100, props.percentOuter / 100, props.orientation, props.scalePadding);
        scaleLinePos = svgHeight.value / 2; // Center vertically using dynamic height

        // Calculate indicator position based on distance percentage of current SVG height
        const indicatorDistancePx = (indicatorDistancePercent / 100) * svgHeight.value;
        // Indicator points DOWN towards the scale line (from above)
        indicatorPoints = `0,${scaleLinePos - indicatorDistancePx + indicatorSize} -${indicatorSize / 2},${scaleLinePos - indicatorDistancePx} ${indicatorSize / 2},${scaleLinePos - indicatorDistancePx}`;
        tickTextOffset = 15; // Below ticks
    } else { // vertical
        d3svg.attr("width", svgWidth.value)
            .attr("height", svgHeight.value)
            .attr("viewBox", `0 0 ${svgWidth.value} ${svgHeight.value}`);
        scale = createCustomScale(svgHeight.value, props.domainBreakpoints, props.percentCenter / 100, props.percentMid / 100, props.percentOuter / 100, props.orientation, props.scalePadding);
        scaleLinePos = svgWidth.value / 2; // Center horizontally using dynamic width

        // Calculate indicator position based on distance percentage of current SVG width
        const indicatorDistancePx = (indicatorDistancePercent / 100) * svgWidth.value;
        // Indicator points LEFT towards the scale line (from right)
        indicatorPoints = `${scaleLinePos - indicatorDistancePx + indicatorSize},0 ${scaleLinePos - indicatorDistancePx}, -${indicatorSize / 2} ${scaleLinePos - indicatorDistancePx},${indicatorSize / 2}`;
        tickTextOffset = 20; // To the right of ticks
    }

    console.log("Scale range after creation:", scale.range());

    // Define tick mark values
    const majorTicks = props.domainBreakpoints; // Major ticks are the provided breakpoints
    const minorTicks = []; // For integer ticks outside [-1,1]
    const halfTicks = []; // For 0.5 and -0.5
    const smallTicks = []; // For 0.1 increments within [-1,1]

    // Identify the [-1, 1] range from sorted breakpoints
    const zeroIndex = majorTicks.indexOf(0);
    const negOne = majorTicks[zeroIndex - 1]; // Should be -1
    const posOne = majorTicks[zeroIndex + 1]; // Should be 1

    // Generate minor, half, and small ticks ONLY within the [-1, 1] range
    if (negOne !== undefined && posOne !== undefined) {
        d3.range(negOne, posOne + 0.1, 0.1).forEach(d => {
            const val = parseFloat(d.toFixed(2));
            // Only add if it's not a major tick (which are already handled)
            if (!majorTicks.includes(val)) {
                if (val % 0.5 === 0 && val !== 0) { // Half ticks (e.g., -0.5, 0.5)
                    halfTicks.push(val);
                } else if (val % 1 === 0) { // Integer values within [-1,1] that are not major ticks (e.g., none if -1 and 1 are major)
                    // This condition might not add anything if -1 and 1 are always major ticks
                    // but it's kept for robustness if domainBreakpoints change
                    minorTicks.push(val); // Could be used for integer ticks within this range if needed
                } else { // Small ticks (0.1 increments, excluding half/major)
                    smallTicks.push(val);
                }
            }
        });
    }

    // Generate integer ticks for other ranges (outside [-1, 1])
    const minDomain = majorTicks[0];
    const maxDomain = majorTicks[majorTicks.length - 1];

    d3.range(minDomain, maxDomain + 1, 1).forEach(d => { // Iterate through all integers in the full domain
        if (!majorTicks.includes(d) && (d < negOne || d > posOne)) { // If it's an integer, not a major tick, AND outside [-1,1]
            minorTicks.push(d);
        }
    });


    const allTickValues = [...majorTicks, ...minorTicks, ...halfTicks, ...smallTicks].sort((a, b) => a - b);
    console.log("All tick values:", allTickValues);

    // Create a group for each tick mark
    d3svg.selectAll(".tick")
        .data(allTickValues)
        .enter()
        .append("g")
        .attr("class", "tick")
        .attr("transform", d => {
            const pos = scale(d);
            console.log(`Tick value: ${d}, Scaled position: ${pos}`);
            return props.orientation === 'horizontal' ?
                `translate(${pos}, ${scaleLinePos})` :
                `translate(${scaleLinePos}, ${pos})`;
        })
        .each(function (d) {
            const g = d3.select(this);
            let tickLength;
            let strokeColor = "#666";
            let textColor = "#333";
            let textWeight = "normal";

            if (majorTicks.includes(d)) {
                tickLength = 20;
                strokeColor = "#333";
                textWeight = "bold";
                g.classed("major-tick", true);
            } else if (halfTicks.includes(d)) { // -0.5, 0.5
                tickLength = 10;
            } else if (smallTicks.includes(d)) { // 0.1 increments in [-1,1]
                tickLength = 5;
            } else if (minorTicks.includes(d)) { // Integer ticks outside [-1,1]
                tickLength = 15;
            }

            // Append the tick line with explicit stroke and stroke-width
            g.append("line")
                .attr(props.orientation === 'horizontal' ? "x1" : "y1", 0)
                .attr(props.orientation === 'horizontal' ? "y1" : "x1", 0)
                .attr(props.orientation === 'horizontal' ? "x2" : "y2", 0)
                .attr(props.orientation === 'horizontal' ? "y2" : "x2", tickLength)
                .attr("stroke", strokeColor)
                .attr("stroke-width", majorTicks.includes(d) ? 2 : 1);

            // Add text labels only for major ticks
            if (majorTicks.includes(d)) {
                g.append("text")
                    .attr(props.orientation === 'horizontal' ? "y" : "x", tickLength + tickTextOffset)
                    .attr("text-anchor", props.orientation === 'horizontal' ? "middle" : "start")
                    .attr("fill", textColor)
                    .style("font-weight", textWeight)
                    .text(d);
            }
        });

    // Add the confidence range box (drawn first so indicator is on top)
    d3svg.append("rect")
        .attr("class", "confidence-box")
        .attr("rx", 4)
        .attr("ry", 4);

    // Add the indicator triangle
    d3svg.append("polygon")
        .attr("class", "indicator-triangle")
        .attr("points", indicatorPoints);

    // Initial update of indicator and confidence box
    updateIndicatorAndConfidence();
}

// On component mount, draw the initial scale
onMounted(() => {
    // Initialize ResizeObserver
    resizeObserver = new ResizeObserver(entries => {
        for (let entry of entries) {
            if (entry.contentBoxSize) {
                const newWidth = entry.contentBoxSize[0].inlineSize;
                const newHeight = entry.contentBoxSize[0].blockSize;
                if (svgWidth.value !== newWidth || svgHeight.value !== newHeight) {
                    svgWidth.value = newWidth;
                    svgHeight.value = newHeight;
                    drawScale();
                }
            }
        }
    });

    if (svgRef.value && svgRef.value.parentElement) {
        resizeObserver.observe(svgRef.value.parentElement);
    }

    if (svgRef.value && svgRef.value.parentElement) {
        svgWidth.value = svgRef.value.parentElement.clientWidth;
        svgHeight.value = svgRef.value.parentElement.clientHeight;
        drawScale();
    }
});

// Clean up ResizeObserver on unmount
onUnmounted(() => {
    if (resizeObserver && svgRef.value && svgRef.value.parentElement) {
        resizeObserver.unobserve(svgRef.value.parentElement);
    }
});

// Watch for changes in relevant props to redraw or update the scale elements
watch(
    [
        () => props.value,
        () => props.confidenceRangePercent,
        () => props.indicatorColor,
        () => props.indicatorOpacity,
        () => props.confidenceColor,
        () => props.confidenceOpacity,
        () => props.confidenceBoxCrossDimension,
        () => props.transitionDuration
    ],
    () => {
        updateIndicatorAndConfidence();
    }
);

// Watch for changes in props that require a full redraw of the scale
watch(
    [
        () => props.orientation,
        () => props.scalePadding,
        () => props.percentCenter,
        () => props.percentMid,
        () => props.percentOuter,
        () => props.indicatorSize,
        () => props.indicatorDistancePercent,
        () => props.domainBreakpoints // Watch the new prop
    ],
    () => {
        drawScale();
    },
    { deep: true } // Use deep watch for array prop
);
</script>

<template>
    <svg ref="svgRef" class="bg-white rounded-xl shadow-md mb-8"></svg>
</template>

<style scoped>
/* Scoped styles for the LinearScale component */
svg {
    display: block;
    width: 100%;
    height: 100%;
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.tick line {
    stroke: #666;
    stroke-width: 1px;
}

.tick text {
    font-size: 14px;
    fill: #333;
    text-anchor: middle;
}

.major-tick line {
    stroke: #333;
    stroke-width: 2px;
}

.major-tick text {
    font-weight: 700;
}

.indicator-triangle {
    border-radius: 4px;
}

.confidence-box {
    border-radius: 4px;
}
</style>