# D3.js Custom Linear Scale Vue Component

[![Watch the demo video](https://img.youtube.com/vi/Lg3v1aCPW1s/0.jpg)](https://www.youtube.com/watch?v=Lg3v1aCPW1s)

[Live Demo](https://static.mah.priv.at/apps/custom-linear-scale/)

This project demonstrates a highly configurable linear scale visualization built with D3.js and integrated as a Vue 3 component within a Vite project. It features a custom non-linear scale distribution, various tick mark types, a dynamic indicator, a confidence range, and a simulation mode.

## Features

see [Gemini-generated README](README_gemini.md) for details
## Usage example
````javascript
<script setup>
import { ref } from 'vue';
import LinearScale from './components/LinearScale.vue';

const myValue = ref(0.5);
const myIndicatorColor = ref('#1e40af'); // A blue color
const myOrientation = ref('vertical');
const myDomainBreakpoints = ref([-10, -5, -1, 0, 1, 5, 10]); // Example breakpoints for the scale's domain
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
      :domainBreakpoints="[-10, -5, -1, 0, 1, 5, 10]"
    />
  </div>
</template>
````

## Appearance

The scale was inspired by this Vertical Speed Indicator:


![alt text](assets/stauscheiben-variometer-5-stvl-mebereich-10-ms_35566_1.jpg?version%3D1752579605008)

# Conversation

this was done with Gemini, the complete conversation is here: https://g.co/gemini/share/579d91d9489e