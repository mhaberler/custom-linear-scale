<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';
import LinearScale from './components/LinearScale.vue';

// Reactive state for all configurable options
const currentValue = ref(0);
const indicatorSize = ref(22);
const indicatorColor = ref('#ef4444');
const indicatorOpacity = ref(1.0);
const indicatorDistancePercent = ref(20);
const confidenceRangePercent = ref(5);
const confidenceOpacity = ref(0.6);
const confidenceColor = ref('#a7f3d0');
const confidenceBoxCrossDimension = ref(10); // New reactive state for cross dimension
const transitionDuration = ref(0.9);
const orientation = ref('horizontal');
const scalePadding = ref(50);
const percentCenter = ref(40);
const percentMid = ref(40);
const percentOuter = ref(20);
const simulationFrequency = ref(1);
const isSimulationRunning = ref(false);

// New reactive states for container dimensions
const scaleContainerWidth = ref(450);
const scaleContainerHeight = ref(100);

let simulationInterval = null;

// Function to check percentage sum and update error message
const checkPercentages = () => {
  const total = percentCenter.value + percentMid.value + percentOuter.value;
  const errorElement = document.getElementById('percentage-error');
  if (errorElement) { // Ensure element exists before accessing classList
    if (total !== 100) {
      errorElement.classList.remove('hidden');
    } else {
      errorElement.classList.add('hidden');
    }
  }
};

// Watch for changes in percentages to update the total sum display
watch([percentCenter, percentMid, percentOuter], () => {
  checkPercentages();
});

// Perform initial check after component is mounted
onMounted(() => {
  checkPercentages();
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
    currentValue.value = Math.random() * 20 - 10; // Random value between -10 and 10
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
        :percentCenter="percentCenter" :percentMid="percentMid" :percentOuter="percentOuter" />
    </div>

    <div class="controls-container mt-8">
      <div class="control-section">
        <label for="value-slider" class="text-lg font-medium text-gray-700">Adjust Indicator Value:</label>
        <input type="range" id="value-slider" min="-10" max="10" step="0.01" v-model.number="currentValue"
          :disabled="isSimulationRunning">
        <div class="text-gray-600">Current Value: <span id="current-value">{{ currentValue.toFixed(2) }}</span></div>
      </div>

      <div class="control-section">
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

      <div class="control-section">
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

      <div class="control-section">
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

      <div class="control-section">
        <label for="transition-duration" class="text-lg font-medium text-gray-700">Transition Duration
          (seconds):</label>
        <input type="range" id="transition-duration" v-model.number="transitionDuration" min="0" max="2" step="0.05"
          :disabled="isSimulationRunning">
        <div class="text-gray-600">Current Duration: <span id="current-duration">{{ transitionDuration.toFixed(2)
            }}</span> s</div>
      </div>

      <div class="control-section">
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

      <div class="control-section">
        <label class="text-lg font-medium text-gray-700">Scale Padding (pixels):</label>
        <input type="number" id="scale-padding" v-model.number="scalePadding" min="0" step="10"
          :disabled="isSimulationRunning">
      </div>

      <div class="control-section">
        <label class="text-lg font-medium text-gray-700">Range Distribution Percentages:</label>
        <div class="flex flex-wrap justify-center gap-4">
          <div class="flex flex-col items-center">
            <label for="percent-center" class="text-sm text-gray-600">[-1, 1] %:</label>
            <input type="number" id="percent-center" v-model.number="percentCenter" min="0" max="100"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="percent-mid" class="text-sm text-gray-600">[-5,-1] & [1,5] %:</label>
            <input type="number" id="percent-mid" v-model.number="percentMid" min="0" max="100"
              :disabled="isSimulationRunning">
          </div>
          <div class="flex flex-col items-center">
            <label for="percent-outer" class="text-sm text-gray-600">[-10,-5] & [5,10] %:</label>
            <input type="number" id="percent-outer" v-model.number="percentOuter" min="0" max="100"
              :disabled="isSimulationRunning">
          </div>
        </div>
        <p id="percentage-error" class="text-red-600 text-sm mt-2 hidden">Percentages must sum to 100%!</p>
      </div>

      <button @click="stopSimulation();" class="action-button update-button" :disabled="isSimulationRunning">Update
        Scale</button>

      <div class="control-section">
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
  width: 90%;
  /* Responsive width */
  max-width: 700px;
  /* Maximum width */
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  /* Space between sections */
  padding: 1.5rem;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.control-section {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
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
