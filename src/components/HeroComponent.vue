<template>
  <div style="height: 100vh; width: 100%">
    <l-map ref="map" v-model:zoom="zoom" :center="[41.2995, 69.2401]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>

      <l-circle-marker
        v-if="originCountry"
        :lat-lng="originCountry.coordinates"
        :radius="8"
        color="red"
        fill-color="#f03"
        :fill-opacity="0.5"
      />

      <l-marker v-if="destinationCountry" :lat-lng="destinationCountry.coordinates">
        <l-icon :icon-url="destinationCountry.flag" :icon-size="[32, 32]"></l-icon>
      </l-marker>

      <l-marker v-if="movingFlag.url" :lat-lng="movingFlag.location">
        <l-icon :icon-url="movingFlag.url" :icon-size="[32, 32]"></l-icon>
      </l-marker>

      <l-polyline :lat-lngs="flightPath" color="blue"></l-polyline>
    </l-map>
  </div>
</template>


<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LMarker,
  LIcon,
  LCircleMarker,
  LPolyline,
} from "@vue-leaflet/vue-leaflet";
import { countries as countryList } from "./countries.js";

const zoom = ref(5);
const countries = countryList;

const originCountry = ref(null);
const destinationCountry = ref(null);
const movingFlag = ref({
  url: null,
  location: [0, 0],
});
const flightPath = ref([]);
let interval = null;

function getRandomCountry(excludeCountry = null) {
  let randomCountry;
  do {
    randomCountry = countries[Math.floor(Math.random() * countries.length)];
  } while (excludeCountry && randomCountry.name === excludeCountry.name);
  return randomCountry;
}

function animateFlag() {
  let start = null;
  const startCoordinates = originCountry.value.coordinates;
  const targetCoordinates = destinationCountry.value.coordinates;
  const duration = 5000;

  movingFlag.value.url = destinationCountry.value.flag;

  function step(timestamp) {
    if (!start) start = timestamp;
    const progress = timestamp - start;
    const percentage = Math.min(progress / duration, 1);

    const newLat = startCoordinates[0] + (targetCoordinates[0] - startCoordinates[0]) * percentage;
    const newLng = startCoordinates[1] + (targetCoordinates[1] - startCoordinates[1]) * percentage;

    movingFlag.value.location = [newLat, newLng];

    if (percentage < 1) {
      requestAnimationFrame(step);
    } else {
      originCountry.value = destinationCountry.value;
      destinationCountry.value = null;
      flightPath.value = [];
    }
  }

  requestAnimationFrame(step);
}

function startAnimationCycle() {
  interval = setInterval(() => {
    destinationCountry.value = getRandomCountry(originCountry.value);
    flightPath.value = [
      originCountry.value.coordinates,
      destinationCountry.value.coordinates,
    ];
    animateFlag();
  }, 6000);
}

onMounted(() => {
  originCountry.value = getRandomCountry();
  movingFlag.value.location = originCountry.value.coordinates;
  movingFlag.value.url = originCountry.value.flag;

  startAnimationCycle();
});

onBeforeUnmount(() => {
  clearInterval(interval);
});
</script>

