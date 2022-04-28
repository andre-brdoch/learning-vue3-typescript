<template>
  <main class="py-6 px-2 mx-auto max-w-lg">
    <header class="text-center mb-8">
      <h1 class="text-3xl font-bold mb-4">Hourly Weather Forecast</h1>
      <p class="text-xl">
        Stockholm, {{ formatDate(new Date(state.date), 'MMMM do, yyyy') }}
      </p>
    </header>

    <div class="grid grid-cols-1 gap-2">
      <WeatherCard
        v-for="item in state.hoursData"
        :time="formatDate(new Date(item.time), 'h a')"
        :temperature="item.temperature"
        :cloudcover="item.cloudcover"
      />
    </div>
  </main>
</template>

<script lang="ts" setup>
import axios from 'axios';
import { onMounted, reactive } from 'vue';
import formatDate from 'date-fns/format';
import WeatherCard from './components/WeatherCard.vue';

interface ApiPayload {
  hourly: {
    cloudcover: number[];
    temperature_2m: number[];
    time: string[];
  };
  hourly_units: {
    cloudcover: string;
    temperature_2m: string;
    time: string;
  };
}

interface HourData {
  temperature: number;
  cloudcover: number;
  time: string;
}

type Units = {
  [U in keyof HourData]: string;
};

interface State {
  hoursData: HourData[];
  units?: Units;
  date: string;
}

const state = reactive<State>({
  hoursData: [],
  units: undefined,
  date: new Date().toString(),
});

onMounted(async () => {
  const url =
    'https://api.open-meteo.com/v1/forecast?latitude=59.3328&longitude=18.0645&hourly=temperature_2m,cloudcover';

  const response = await axios(url);
  const data: ApiPayload = response.data;

  console.log(data);

  state.hoursData = Array.from(Array(24).keys()).map(i => {
    const {
      temperature_2m: temperatures,
      cloudcover: cloudcovers,
      time: times,
    } = data.hourly;
    return {
      temperature: temperatures[i],
      cloudcover: cloudcovers[i],
      time: times[i],
    };
  });
  const {
    temperature_2m: temperature,
    cloudcover: cloudcover,
    time: time,
  } = data.hourly_units;
  state.units = { temperature, cloudcover, time };
});
</script>
