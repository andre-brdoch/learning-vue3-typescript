<template>
  <h1>Weather forecast</h1>
  <p>Stockholm</p>
  <p>{{ formatDate(new Date(state.date), 'MMMM do, yyyy') }}</p>

  <div>
    <article
      v-for="item in state.hoursData"
      :class="['bg-slate-100', 'mb-2', 'p-2']"
    >
      <p>Time: {{ formatDate(new Date(item.time), 'HH:00') }}</p>
      <p>Temperature: {{ item.temperature }}{{ state.units?.temperature }}</p>
      <p>Clouds: {{ item.cloudcover }}{{ state.units?.cloudcover }}</p>
    </article>
  </div>
</template>

<script lang="ts" setup>
import axios from 'axios';
import { onMounted, reactive } from 'vue';
import formatDate from 'date-fns/format';

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
    'https://api.open-meteo.com/v1/forecast?latitude=59.3328&longitude=18.0645&hourly=temperature_2m,cloudcover,windspeed_10m';

  const response = await axios(url);
  const data: ApiPayload = response.data;
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
