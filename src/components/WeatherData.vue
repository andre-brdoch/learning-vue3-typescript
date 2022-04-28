<template>
  <div>
    <div>
      <label for="city-select" class="block">Select a city</label>
      <select
        name="city"
        id="city-select"
        v-model="selectedCity"
        class="p-2 border"
      >
        <option value="" disabled selected>Select...</option>
        <option
          v-for="location in locations"
          :key="location.name"
          :value="location.name"
        >
          {{ location.name }}
        </option>
      </select>
    </div>

    <slot v-bind="slotData" />
  </div>
</template>

<script lang="ts" setup>
import axios from 'axios';
import { onMounted, reactive, ref, watch } from 'vue';
import { computed } from '@vue/reactivity';

interface Coordinates {
  lat: number;
  long: number;
}

type City = 'Lund' | 'Stockholm' | 'Ljubljana' | 'Helsingborg' | 'Borlänge';

interface Location extends Coordinates {
  name: City;
}

interface ApiPayload {
  hourly: {
    cloudcover: number[];
    temperature_2m: number[];
    time: string[];
  };
}

interface WeatherByHour {
  temperature: number;
  cloudcover: number;
  date: Date;
}

const locations = ref<Location[]>([
  { lat: 55.70584, long: 13.19321, name: 'Lund' },
  { lat: 59.3328, long: 18.0645, name: 'Stockholm' },
  { lat: 46.0569, long: 14.5058, name: 'Ljubljana' },
  { lat: 56.0465, long: 12.6945, name: 'Helsingborg' },
  { lat: 60.4843, long: 15.434, name: 'Borlänge' },
]);
const selectedCity = ref<City>(locations.value[0].name);
let weatherByHours = ref<WeatherByHour[]>([]);

const coordinates = computed(() => {
  const city = locations.value.find(l => l.name === selectedCity.value);
  if (!city) return {};
  return { lat: city.lat, long: city.long };
});
const weatherEndpoint = computed(() => {
  const { lat, long } = coordinates.value;
  return `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${long}&timezone=Europe%2FBerlin&hourly=temperature_2m,cloudcover`;
});
const slotData = computed(() => {
  const { lat, long } = coordinates.value;
  return {
    lat,
    long,
    weatherByHours: weatherByHours.value,
    city: selectedCity.value,
  };
});

const fetchWeather = async () => {
  const response = await axios(weatherEndpoint.value);
  const data: ApiPayload = response.data;

  weatherByHours.value = Array.from(Array(24).keys()).map(i => {
    const {
      temperature_2m: temperatures,
      cloudcover: cloudcovers,
      time: dates,
    } = data.hourly;
    return {
      temperature: temperatures[i],
      cloudcover: cloudcovers[i],
      date: new Date(dates[i]),
    };
  });
};

onMounted(fetchWeather);
watch(weatherEndpoint, fetchWeather);
</script>
