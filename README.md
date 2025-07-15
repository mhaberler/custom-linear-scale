# D3.js Custom Linear Scale Vue Component

[![Watch the demo video](https://img.youtube.com/vi/Lg3v1aCPW1s/0.jpg)](https://www.youtube.com/watch?v=Lg3v1aCPW1s)

[Live Demo](https://static.mah.priv.at/apps/custom-linear-scale/)

This project demonstrates a highly configurable linear scale visualization built with D3.js and integrated as a Vue 3 component within a Vite project. It features a custom non-linear scale distribution, various tick mark types, a dynamic indicator, a confidence range, and a simulation mode.

## Features

* **Custom Non-Linear Scale:** The scale's range from -10 to 10 is distributed across three segments with configurable percentages:
    * `[-1, 1]`
    * `[1, 5]` and `[-5, -1]`
    * `[5, 10]` and `[-10, -5]`
* **Detailed Tick Marks:** Includes major, minor, half, and even smaller 0.1 interval tick marks.
* **Dynamic Indicator:** A red triangle indicator points towards the scale, with configurable size, color, opacity, and distance from the scale.
* **Confidence Range:** An opaque box centered on the indicator, representing a confidence range. Its width (along the scale), perpendicular dimension, opacity, and color are configurable.
* **Orientation Toggle:** Switch between horizontal and vertical scale orientations.
* **Configurable Padding:** Adjust the space at the ends of the scale.
* **Smooth Transitions:** CSS transitions with configurable duration provide a dampened, visually appealing movement for dynamic elements.
* **Simulation Mode:** A built-in simulation updates the indicator value and confidence range randomly at a configurable frequency (1-10 Hz).
* **Responsive Design:** The scale component adapts to the size of its container, ensuring responsiveness.
* **Vue 3 Component:** Encapsulated as a reusable Vue component.
* **Vite & Tailwind CSS v4:** Modern development setup for fast development and sleek styling.

## Usage example
````javascript
<script setup>
import { ref } from 'vue';
import LinearScale from './components/LinearScale.vue';

const myValue = ref(0.5);
const myIndicatorColor = ref('#1e40af'); // A blue color
const myOrientation = ref('vertical');
</script>

<template>
  <div style="width: 400px; height: 600px; border: 1px solid #ccc;">
    <LinearScale
      :value="myValue"
      :indicatorColor="myIndicatorColor"
      :orientation="myOrientation"
      :scalePadding="60"
      :percentCenter="50"
      :percentMid="30"
      :percentOuter="20"
      :indicatorSize="25"
      :confidenceRangePercent="10"
      :confidenceBoxCrossDimension="30"
      :transitionDuration="0.5"
    />
  </div>
</template>
````

## Appearance

The scale was inspired by this Vertical Speed Indicator:


![alt text](assets/stauscheiben-variometer-5-stvl-mebereich-10-ms_35566_1.jpg?version%3D1752579605008)

# Conversation

this was done with Gemini, the complete conversation is here: https://gemini.google.com/share/75d7bfdb242a