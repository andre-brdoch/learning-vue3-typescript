<template>
  <article class="bg-slate-200 p-2 rounded-md grid grid-cols-3 items-center">
    <p>{{ time }}</p>
    <Icon />
    <p class="text-3xl text-right">
      {{ temperature.toFixed(0) }}{{ temperatureUnit }}
    </p>
  </article>
</template>

<script lang="ts" setup>
import { computed } from '@vue/reactivity';
import DayIcon from 'vue-material-design-icons/WeatherSunny.vue';
import CloudyDayIcon from 'vue-material-design-icons/WeatherPartlyCloudy.vue';
import NightIcon from 'vue-material-design-icons/WeatherNight.vue';
import CloudyNightIcon from 'vue-material-design-icons/WeatherNightPartlyCloudy.vue';

interface Props {
  time: string;
  temperature: number;
  cloudcover: number;
  temperatureUnit?: string;
  cloudcoverUnit?: string;
}
const props = withDefaults(defineProps<Props>(), {
  temperatureUnit: '°C',
  cloudcoverUnit: '%',
});

const Icon = computed(() => {
  return props.cloudcover < 50 ? CloudyDayIcon : DayIcon;
});
</script>
