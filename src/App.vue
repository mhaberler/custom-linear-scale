<script setup>
import { ref, watch, onMounted, onUnmounted, computed } from 'vue';
import LinearScale from './components/LinearScale.vue';

// Reactive state for all configurable options
const currentValue = ref(0);
const indicatorSize = ref(20);
const indicatorColor = ref('#ef4444');
const indicatorOpacity = ref(1.0);
const indicatorDistancePercent = ref(5);
const confidenceRangePercent = ref(5);
const confidenceOpacity = ref(0.6);
const confidenceColor = ref('#a7f3d0');
const confidenceBoxCrossDimension = ref(20);
const transitionDuration = ref(0.3);
const orientation = ref('horizontal');
const scalePadding = ref(50);
const simulationFrequency = ref(1);
const isSimulationRunning = ref(false);

// Reactive states for container dimensions
const scaleContainerWidth = ref(900);
const scaleContainerHeight = ref(200);

// New reactive states for scale configuration arrays
const majorTicksInput = ref('-10,-5,-1,0,1,5,10');
const minorTicksInput = ref('-0.9,-0.8,-0.7,-0.6,-0.4,-0.3,-0.2,-0.1,0.1,0.2,0.3,0.4,0.6,0.7,0.8,0.9');
const intermediateTicksInput = ref('-0.5,0.5');
const weightsInput = ref('0.1,0.1,0.3,0.3,0.1,0.1');

// Error flags for input parsing
const majorTicksError = ref(false);
const minorTicksError = ref(false);
const intermediateTicksError = ref(false);
const weightsError = ref(false);
const weightsSumError = ref(false);

let simulationInterval = null;

// Helper function to parse comma-separated numbers
const parseNumberArray = (input, errorRef) => {
  try {
    const parsed = input.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n));
    errorRef.value = false;
    return parsed;
  } catch (e) {
    errorRef.value = true;
    return [];
  }
};

// Computed properties for parsed and validated arrays
const parsedMajorTicks = computed(() => {
  const parsed = parseNumberArray(majorTicksInput.value, majorTicksError);
  // Basic validation: Must have at least 2 points and be sorted
  if (parsed.length < 2 || !parsed.every((val, i, arr) => i === 0 || val >= arr[i - 1])) {
    majorTicksError.value = true;
    return [];
  }
  return parsed;
});

const parsedMinorTicks = computed(() => parseNumberArray(minorTicksInput.value, minorTicksError));
const parsedIntermediateTicks = computed(() => parseNumberArray(intermediateTicksInput.value, intermediateTicksError));

const parsedWeights = computed(() => {
  const parsed = parseNumberArray(weightsInput.value, weightsError);
  // Validate weights length and sum
  if (parsedMajorTicks.value.length > 0 && parsed.length !== (parsedMajorTicks.value.length - 1)) {
    weightsError.value = true;
    return [];
  }
  const sum = parsed.reduce((acc, val) => acc + val, 0);
  // Check if sum is close to 1 (allowing for floating point inaccuracies)
  if (Math.abs(sum - 1.0) > 0.001) {
    weightsSumError.value = true;
  } else {
    weightsSumError.value = false;
  }
  return parsed;
});

// Watch for changes in input fields to trigger re-parsing and re-render
watch([majorTicksInput, minorTicksInput, intermediateTicksInput, weightsInput], () => {
  // Computed properties will automatically re-evaluate, triggering LinearScale re-render
});

// Update simulation range based on major ticks
watch(parsedMajorTicks, (newVal) => {
  if (isSimulationRunning.value) {
    stopSimulation();
    startSimulation(); // Restart simulation with new range
  }
});


const toggleSimulation = () => {
  isSimulationRunning.value = !isSimulationRunning.value;

  if (isSimulationRunning.value) {
    startSimulation();
  } else {
    stopSimulation();
  }
};

const startSimulation = () => {
  stopSimulation(); // Clear any existing interval first
  const frequency = simulationFrequency.value;
  const intervalMs = 1000 / frequency;

  simulationInterval = setInterval(() => {
    const domain = parsedMajorTicks.value;
    if (domain.length > 1) {
      const minVal = domain[0];
      const maxVal = domain[domain.length - 1];
      currentValue.value = Math.random() * (maxVal - minVal) + minVal;
    } else {
      currentValue.value = 0; // Fallback if domain is invalid
    }

    confidenceRangePercent.value = Math.random() * 15 + 1; // Random confidence between 1% and 16%
  }, intervalMs);
};

const stopSimulation = () => {
  if (simulationInterval) {
    clearInterval(simulationInterval);
    simulationInterval = null;
  }
};

// Ensure simulation stops when component is unmounted
onUnmounted(() => {
  stopSimulation();
});
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-f0f4f8 text-333 p-4">
    <h1 class="text-3xl font-bold mb-6 text-gray-800">Custom Linear Scale</h1>

    <!-- Scale container with dynamic width and height -->
    <div id="scale-container"
      class="flex justify-center items-center overflow-hidden bg-white rounded-xl shadow-md mb-8"
      :style="{ width: scaleContainerWidth + 'px', height: scaleContainerHeight + 'px' }">
      <LinearScale :value="currentValue" :indicatorSize="indicatorSize" :indicatorColor="indicatorColor"
        :indicatorOpacity="indicatorOpacity" :indicatorDistancePercent="indicatorDistancePercent"
        :confidenceRangePercent="confidenceRangePercent" :confidenceOpacity="confidenceOpacity"
        :confidenceColor="confidenceColor" :confidenceBoxCrossDimension="confidenceBoxCrossDimension"
        :transitionDuration="transitionDuration" :orientation="orientation" :scalePadding="scalePadding"
        :majorTicks="parsedMajorTicks" :minorTicks="parsedMinorTicks" :intermediateTicks="parsedIntermediateTicks"
        :weights="parsedWeights" />
    </div>

    <!-- Main controls container with responsive grid -->
    <div class="controls-container mt-8 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 w-full max-w-4xl">
      <!-- Value Slider - Full width -->
      <div class="control-section col-span-full">
        <label for="value-slider" class="text-lg font-medium text-gray-700">Adjust Indicator Value:</label>
        <input type="range" id="value-slider" :min="parsedMajorTicks[0] || -10"
          :max="parsedMajorTicks[parsedMajorTicks.length - 1] || 10" step="0.01" v-model.number="currentValue"
          :disabled="isSimulationRunning">
        <div class="text-gray-600">Current Value: <span id="current-value">{{ currentValue.toFixed(2) }}</span></div>
      </div>

      <!-- Scale Container Dimensions - Single column -->
      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Scale Container Dimensions:</label>
        <div class="flex flex-wrap justify-center gap-4">
          <div class="flex flex-col items-center">
            <label for="container-width" class="text-sm text-gray-600">Width (px):</label>
            <input type="number" id="container-width" v-model.number="scaleContainerWidth" min="100" step="10"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="container-height" class="text-sm text-gray-600">Height (px):</label>
            <input type="number" id="container-height" v-model.number="scaleContainerHeight" min="100" step="10"
              :disabled="isSimulationRunning">
          </div>
        </div>
      </div>

      <!-- Tick and Weight Inputs - Each in its own column -->
      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Major Ticks (comma-separated, sorted):</label>
        <input type="text" id="major-ticks" v-model="majorTicksInput" class="w-full p-2 border rounded-md"
          :class="{ 'border-red-500': majorTicksError }" :disabled="isSimulationRunning">
        <p v-if="majorTicksError" class="text-red-600 text-sm mt-1">Invalid format. Must be comma-separated numbers,
          sorted, and have at least 2 points.</p>
      </div>

      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Minor Ticks (comma-separated):</label>
        <input type="text" id="minor-ticks" v-model="minorTicksInput" class="w-full p-2 border rounded-md"
          :class="{ 'border-red-500': minorTicksError }" :disabled="isSimulationRunning">
        <p v-if="minorTicksError" class="text-red-600 text-sm mt-1">Invalid format. Must be comma-separated numbers.</p>
      </div>

      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Intermediate Ticks (comma-separated):</label>
        <input type="text" id="intermediate-ticks" v-model="intermediateTicksInput" class="w-full p-2 border rounded-md"
          :class="{ 'border-red-500': intermediateTicksError }" :disabled="isSimulationRunning">
        <p v-if="intermediateTicksError" class="text-red-600 text-sm mt-1">Invalid format. Must be comma-separated
          numbers.</p>
      </div>

      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Segment Weights (comma-separated, sum to 1):</label>
        <input type="text" id="weights" v-model="weightsInput" class="w-full p-2 border rounded-md"
          :class="{ 'border-red-500': weightsError || weightsSumError }" :disabled="isSimulationRunning">
        <p v-if="weightsError" class="text-red-600 text-sm mt-1">Invalid format or incorrect number of weights. Must
          match (major ticks count - 1).</p>
        <p v-if="weightsSumError" class="text-red-600 text-sm mt-1">Weights must sum to approximately 1.0.</p>
      </div>

      <!-- Indicator Settings - Full width on small, half on medium, one-third on large -->
      <div class="control-section col-span-full md:col-span-2 lg:col-span-1">
        <label class="text-lg font-medium text-gray-700">Indicator Settings:</label>
        <div class="flex flex-wrap justify-center gap-4">
          <div class="flex flex-col items-center">
            <label for="indicator-size" class="text-sm text-gray-600">Size (px):</label>
            <input type="number" id="indicator-size" v-model.number="indicatorSize" min="5" max="50" step="1"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="indicator-color" class="text-sm text-gray-600">Color:</label>
            <input type="color" id="indicator-color" v-model="indicatorColor" :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="indicator-opacity" class="text-sm text-gray-600">Opacity (0-1):</label>
            <input type="range" id="indicator-opacity" v-model.number="indicatorOpacity" min="0" max="1" step="0.05"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="indicator-distance-percent" class="text-sm text-gray-600">Distance (% of SVG):</label>
            <input type="number" id="indicator-distance-percent" v-model.number="indicatorDistancePercent" min="0"
              max="20" step="0.5" :disabled="isSimulationRunning">
          </div>
        </div>
      </div>

      <!-- Confidence Box Settings - Full width on small, half on medium, one-third on large -->
      <div class="control-section col-span-full md:col-span-2 lg:col-span-1">
        <label class="text-lg font-medium text-gray-700">Confidence Box Settings:</label>
        <div class="flex flex-wrap justify-center gap-4">
          <div class="flex flex-col items-center">
            <label for="confidence-range-percent" class="text-sm text-gray-600">Width (% of scale):</label>
            <input type="number" id="confidence-range-percent" v-model.number="confidenceRangePercent" min="0" max="100"
              step="0.1" :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="confidence-box-cross-dimension" class="text-sm text-gray-600">Cross Dimension (px):</label>
            <input type="number" id="confidence-box-cross-dimension" v-model.number="confidenceBoxCrossDimension"
              min="5" max="50" step="1" :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="confidence-opacity" class="text-sm text-gray-600">Opacity (0-1):</label>
            <input type="range" id="confidence-opacity" v-model.number="confidenceOpacity" min="0" max="1" step="0.05"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="confidence-color" class="text-sm text-gray-600">Color:</label>
            <input type="color" id="confidence-color" v-model="confidenceColor" :disabled="isSimulationRunning">
          </div>
        </div>
      </div>

      <!-- Other Controls - Each in its own column -->
      <div class="control-section col-span-1">
        <label for="transition-duration" class="text-lg font-medium text-gray-700">Transition Duration
          (seconds):</label>
        <input type="range" id="transition-duration" v-model.number="transitionDuration" min="0" max="2" step="0.05"
          :disabled="isSimulationRunning">
        <div class="text-gray-600">Current Duration: <span id="current-duration">{{ transitionDuration.toFixed(2)
            }}</span> s</div>
      </div>

      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Orientation:</label>
        <div class="flex gap-4">
          <label class="flex items-center">
            <input type="radio" name="orientation" value="horizontal" v-model="orientation"
              :disabled="isSimulationRunning"> Horizontal
          </label>
          <label class="flex items-center">
            <input type="radio" name="orientation" value="vertical" v-model="orientation"
              :disabled="isSimulationRunning"> Vertical
          </label>
        </div>
      </div>

      <div class="control-section col-span-1">
        <label class="text-lg font-medium text-gray-700">Scale Padding (pixels):</label>
        <input type="number" id="scale-padding" v-model.number="scalePadding" min="0" step="10"
          :disabled="isSimulationRunning">
      </div>

      <!-- Action Buttons - Full width -->
      <div class="control-section col-span-full flex flex-col items-center">
        <button @click="stopSimulation();" class="action-button update-button" :disabled="isSimulationRunning">Update
          Scale</button>
      </div>

      <div class="control-section col-span-full flex flex-col items-center">
        <label for="simulation-frequency" class="text-lg font-medium text-gray-700">Simulation Frequency (Hz):</label>
        <input type="range" id="simulation-frequency" v-model.number="simulationFrequency" min="1" max="10" step="1">
        <div class="text-gray-600">Current Frequency: <span id="current-frequency">{{ simulationFrequency }}</span> Hz
        </div>
        <button @click="toggleSimulation"
          :class="['action-button', 'simulation-button', { 'active': isSimulationRunning }]">
          {{ isSimulationRunning ? 'Stop Simulation' : 'Start Simulation' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Scoped styles for App.vue, mostly for the controls container */
.controls-container {
  /* Removed flex-direction: column and gap from here */
  padding: 1.5rem;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  /* Tailwind grid classes now define the primary layout */
}

.control-section {
  width: 100%;
  /* Ensure sections take full width of their grid column */
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem;
  /* Add some padding to each section for visual separation */
  border: 1px solid #e5e7eb;
  /* Light border for visual grouping */
  border-radius: 8px;
  background-color: #f9fafb;
  /* Slightly different background for sections */
}

input[type="range"] {
  width: 100%;
  -webkit-appearance: none;
  appearance: none;
  height: 8px;
  background: #d1d5db;
  border-radius: 5px;
  outline: none;
  opacity: 0.7;
  transition: opacity .2s;
}

input[type="range"]:hover {
  opacity: 1;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background: #2563eb;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

input[type="range"]::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: #2563eb;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

#current-value {
  font-size: 1.25rem;
  font-weight: bold;
  color: #2563eb;
}

input[type="number"] {
  width: 80px;
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  text-align: center;
  font-size: 1rem;
}

input[type="color"] {
  width: 60px;
  height: 30px;
  border: none;
  padding: 0;
  background: none;
  cursor: pointer;
}

input[type="radio"] {
  margin-right: 0.5rem;
}

.action-button {
  padding: 0.75rem 1.5rem;
  color: white;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.update-button {
  background-color: #10b981;
}

.update-button:hover {
  background-color: #059669;
}

.simulation-button {
  background-color: #f97316;
}

.simulation-button:hover {
  background-color: #ea580c;
}

.simulation-button.active {
  background-color: #dc2626;
}

.simulation-button.active:hover {
  background-color: #b91c1c;
}

@media (max-width: 768px) {
  .controls-container {
    padding: 1rem;
    gap: 1rem;
  }

  .control-section {
    gap: 0.5rem;
  }

  input[type="range"] {
    height: 6px;
  }

  input[type="range"]::-webkit-slider-thumb,
  input[type="range"]::-moz-range-thumb {
    width: 16px;
    height: 16px;
  }

  #current-value {
    font-size: 1rem;
  }
}
</style>