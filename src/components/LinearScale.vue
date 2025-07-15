<script setup>
import * as d3 from 'd3';
import { ref, onMounted, watch, computed } from 'vue';

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
    }
});

// Ref for the SVG element, allowing D3 to select it
const svgRef = ref(null);

// Define fixed SVG dimensions
const initialSvgWidth = 900;
const initialSvgHeight = 200;
const verticalScaleWidth = 300; // Wider SVG for vertical orientation

let scale; // D3 scale instance

/**
 * Creates a custom piecewise linear scale function based on provided percentages.
 * This scale maps values from -10 to 10 onto a pixel range,
 * with specific segments occupying different proportions of the total width/height.
 * @param {number} totalSvgDimension The total dimension of the SVG (width for horizontal, height for vertical).
 * @param {number} p_center Percentage for the [-1, 1] range (e.g., 0.4 for 40%).
 * @param {number} p_mid Percentage for the [-5,-1] and [1,5] ranges (e.g., 0.4 for 40%).
 * @param {number} p_outer Percentage for the [-10,-5] and [5,10] ranges (e.g., 0.2 for 20%).
 * @param {string} orientation 'horizontal' or 'vertical'.
 * @param {number} padding The padding in pixels at the ends of the scale.
 * @returns {function} A function that maps a domain value to a pixel position.
 */
function createCustomScale(totalSvgDimension, p_center, p_mid, p_outer, orientation, padding) {
    let startPx, endPx, effectiveLength;

    if (orientation === 'horizontal') {
        startPx = padding;
        endPx = totalSvgDimension - padding;
        effectiveLength = endPx - startPx;
    } else { // vertical
        startPx = totalSvgDimension - padding; // Scale starts from bottom (higher pixel value)
        endPx = padding; // Scale ends at top (lower pixel value)
        effectiveLength = startPx - endPx;
    }

    const segment1to1Px = p_center * effectiveLength;
    const segment1to5Px = (p_mid / 2) * effectiveLength;
    const segment5to10Px = (p_outer / 2) * effectiveLength;

    const rangePoints = {};
    if (orientation === 'horizontal') {
        rangePoints[0] = startPx + segment5to10Px + segment1to5Px + (segment1to1Px / 2);
        rangePoints[1] = rangePoints[0] + segment1to1Px / 2;
        rangePoints[-1] = rangePoints[0] - segment1to1Px / 2;
        rangePoints[5] = rangePoints[1] + segment1to5Px;
        rangePoints[-5] = rangePoints[-1] - segment1to5Px;
        rangePoints[10] = rangePoints[5] + segment5to10Px;
        rangePoints[-10] = rangePoints[-5] - segment5to10Px;
    } else { // vertical
        rangePoints[0] = startPx - segment5to10Px - segment1to5Px - (segment1to1Px / 2);
        rangePoints[1] = rangePoints[0] - segment1to1Px / 2;
        rangePoints[-1] = rangePoints[0] + segment1to1Px / 2;
        rangePoints[5] = rangePoints[1] - segment1to5Px;
        rangePoints[-5] = rangePoints[-1] + segment1to5Px;
        rangePoints[10] = rangePoints[5] - segment5to10Px;
        rangePoints[-10] = rangePoints[-5] + segment5to10Px;
    }

    const customScale = (value) => {
        if (value >= 0) {
            if (value <= 1) {
                return d3.scaleLinear().domain([0, 1]).range([rangePoints[0], rangePoints[1]])(value);
            } else if (value <= 5) {
                return d3.scaleLinear().domain([1, 5]).range([rangePoints[1], rangePoints[5]])(value);
            } else if (value <= 10) {
                return d3.scaleLinear().domain([5, 10]).range([rangePoints[5], rangePoints[10]])(value);
            }
        } else { // value < 0
            if (value >= -1) {
                return d3.scaleLinear().domain([-1, 0]).range([rangePoints[-1], rangePoints[0]])(value);
            } else if (value >= -5) {
                return d3.scaleLinear().domain([-5, -1]).range([rangePoints[-5], rangePoints[-1]])(value);
            } else if (value >= -10) {
                return d3.scaleLinear().domain([-10, -5]).range([rangePoints[-10], rangePoints[-5]])(value);
            }
        }
        return rangePoints[0]; // Fallback
    };

    customScale.domain = () => [-10, 10];
    customScale.range = () => (orientation === 'horizontal' ? [rangePoints[-10], rangePoints[10]] : [rangePoints[10], rangePoints[-10]]);
    customScale.effectiveLength = effectiveLength; // Expose effective length for confidence box calculation
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
            .attr("y", initialSvgHeight / 2 - 10)
            .attr("width", confidencePx)
            .attr("height", 20);
    } else { // vertical
        d3svg.select(".confidence-box")
            .attr("x", verticalScaleWidth / 2 - 10)
            .attr("y", pos - confidencePx / 2)
            .attr("width", 20)
            .attr("height", confidencePx);
    }
}

/**
 * Draws or redraws the entire scale based on current props.
 */
function drawScale() {
    if (!svgRef.value) {
        console.error("drawScale: SVG element is null. Cannot draw scale.");
        return; // Ensure SVG element is available
    }

    console.log("Drawing scale. Orientation:", props.orientation);
    console.log("Current Padding:", props.scalePadding);
    console.log("Percentages:", props.percentCenter, props.percentMid, props.percentOuter);

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
    const transitionDuration = props.transitionDuration;

    // Set SVG dimensions based on orientation
    if (props.orientation === 'horizontal') {
        d3svg.attr("width", initialSvgWidth)
            .attr("height", initialSvgHeight)
            .attr("viewBox", `0 0 ${initialSvgWidth} ${initialSvgHeight}`);
        scale = createCustomScale(initialSvgWidth, props.percentCenter / 100, props.percentMid / 100, props.percentOuter / 100, props.orientation, props.scalePadding);
        scaleLinePos = initialSvgHeight / 2; // Center vertically

        // Calculate indicator position based on distance percentage
        const indicatorDistancePx = (indicatorDistancePercent / 100) * initialSvgHeight;
        // Indicator points DOWN towards the scale line (from above)
        // Tip at (0, scaleLinePos - indicatorDistancePx + indicatorSize), Base at (0, scaleLinePos - indicatorDistancePx)
        indicatorPoints = `0,${scaleLinePos - indicatorDistancePx + indicatorSize} -${indicatorSize / 2},${scaleLinePos - indicatorDistancePx} ${indicatorSize / 2},${scaleLinePos - indicatorDistancePx}`;
        tickTextOffset = 15; // Below ticks
    } else { // vertical
        d3svg.attr("width", verticalScaleWidth)
            .attr("height", initialSvgWidth) // Use initialSvgWidth as height for vertical
            .attr("viewBox", `0 0 ${verticalScaleWidth} ${initialSvgWidth}`);
        scale = createCustomScale(initialSvgWidth, props.percentCenter / 100, props.percentMid / 100, props.percentOuter / 100, props.orientation, props.scalePadding); // Scale based on height (initialSvgWidth)
        scaleLinePos = verticalScaleWidth / 2; // Center horizontally within the new verticalSvgWidth

        // Calculate indicator position based on distance percentage
        const indicatorDistancePx = (indicatorDistancePercent / 100) * verticalScaleWidth;
        // Indicator points LEFT towards the scale line (from right)
        // Tip at (scaleLinePos - indicatorDistancePx + indicatorSize, 0), Base at (scaleLinePos - indicatorDistancePx, -indicatorSize/2) (scaleLinePos - indicatorDistancePx, indicatorSize/2)
        indicatorPoints = `${scaleLinePos - indicatorDistancePx + indicatorSize},0 ${scaleLinePos - indicatorDistancePx}, -${indicatorSize / 2} ${scaleLinePos - indicatorDistancePx},${indicatorSize / 2}`;
        tickTextOffset = 20; // To the right of ticks
    }

    console.log("Scale range after creation:", scale.range());

    // Draw the main scale line
    d3svg.append("line")
        .attr(props.orientation === 'horizontal' ? "x1" : "y1", scale.range()[0])
        .attr(props.orientation === 'horizontal' ? "y1" : "x1", scaleLinePos)
        .attr(props.orientation === 'horizontal' ? "x2" : "y2", scale.range()[1])
        .attr(props.orientation === 'horizontal' ? "y2" : "x2", scaleLinePos)
        .attr("stroke", "#333")
        .attr("stroke-width", 2)
        .attr("stroke-linecap", "round");

    // Define tick mark values
    const majorTicks = [-10, -5, -1, 0, 1, 5, 10];
    const minorTicks = [-9, -8, -7, -6, -4, -3, -2, 2, 3, 4, 6, 7, 8, 9];
    const halfTicks = [-0.5, 0.5];
    const smallTicks = d3.range(-1, 1.1, 0.1).filter(d =>
        d !== 0 && d !== 1 && d !== -1 && d !== 0.5 && d !== -0.5 &&
        !majorTicks.includes(d) && !minorTicks.includes(d) && !halfTicks.includes(d)
    ).map(d => parseFloat(d.toFixed(2)));

    const allTickValues = [...majorTicks, ...minorTicks, ...halfTicks, ...smallTicks];
    console.log("All tick values:", allTickValues);

    // Create a group for each tick mark
    d3svg.selectAll(".tick")
        .data(allTickValues)
        .enter()
        .append("g")
        .attr("class", "tick")
        .attr("transform", d => {
            const pos = scale(d);
            console.log(`Tick value: ${d}, Scaled position: ${pos}`); // Log each tick position
            return props.orientation === 'horizontal' ?
                `translate(${pos}, ${scaleLinePos})` :
                `translate(${scaleLinePos}, ${pos})`;
        })
        .each(function (d) {
            const g = d3.select(this);
            let tickLength;
            let strokeColor = "#666"; // Default tick color
            let textColor = "#333";   // Default text color
            let textWeight = "normal"; // Default text weight

            if (majorTicks.includes(d)) {
                tickLength = 20;
                strokeColor = "#333"; // Darker for major
                textWeight = "bold";  // Bold for major
                g.classed("major-tick", true);
            } else if (halfTicks.includes(d)) {
                tickLength = 10;
            } else if (smallTicks.includes(d)) {
                tickLength = 5;
            } else { // Minor ticks
                tickLength = 15;
            }

            // Append the tick line with explicit stroke and stroke-width
            g.append("line")
                .attr(props.orientation === 'horizontal' ? "x1" : "y1", 0)
                .attr(props.orientation === 'horizontal' ? "y1" : "x1", 0)
                .attr(props.orientation === 'horizontal' ? "x2" : "y2", 0)
                .attr(props.orientation === 'horizontal' ? "y2" : "x2", tickLength)
                .attr("stroke", strokeColor) // Explicit stroke color
                .attr("stroke-width", majorTicks.includes(d) ? 2 : 1); // Thicker for major

            // Add text labels only for major ticks
            if (majorTicks.includes(d)) {
                g.append("text")
                    .attr(props.orientation === 'horizontal' ? "y" : "x", tickLength + tickTextOffset)
                    .attr(props.orientation === 'horizontal' ? "x" : "y", 0)
                    .attr("text-anchor", props.orientation === 'horizontal' ? "middle" : "start")
                    .attr("fill", textColor)     // Explicit text color
                    .style("font-weight", textWeight) // Explicit font weight
                    .text(d);
            }
        });

    // Add the confidence range box (drawn first so indicator is on top)
    d3svg.append("rect")
        .attr("class", "confidence-box")
        .attr("rx", 4) // Rounded corners
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
    drawScale();
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
        () => props.indicatorDistancePercent
    ],
    () => {
        drawScale();
    }
);
</script>

<template>
    <svg ref="svgRef" class="bg-white rounded-xl shadow-md mb-8"></svg>
</template>

<style scoped>
/* Scoped styles for the LinearScale component */
svg {
    background-color: #ffffff;
    /* White background for the scale */
    border-radius: 12px;
    /* Rounded corners */
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    /* Subtle shadow */
    margin-bottom: 2rem;
    /* Space below the SVG */
}

.tick line {
    /* These are now mostly overridden by explicit D3 attributes, but kept for fallback/general styling */
    stroke: #666;
    stroke-width: 1px;
}

.tick text {
    /* These are now mostly overridden by explicit D3 attributes, but kept for fallback/general styling */
    font-size: 14px;
    fill: #333;
    text-anchor: middle;
}

.major-tick line {
    /* Kept for potential future use or if D3 attributes are removed */
    stroke: #333;
    stroke-width: 2px;
}

.major-tick text {
    /* Kept for potential future use or if D3 attributes are removed */
    font-weight: 700;
}

.indicator-triangle {
    /* Fill color and transition duration will be set dynamically via JS */
    border-radius: 4px;
}

.confidence-box {
    /* Fill color, opacity, and transition duration will be set dynamically via JS */
    border-radius: 4px;
}
</style>
