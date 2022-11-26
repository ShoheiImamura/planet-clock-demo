<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import MyCanvas from "./MyCanvas.vue";
import { format, addDays } from "date-fns";

defineProps({
  msg: String,
});

const currentDatetime = ref('');

const getCurrentDatetime = () => {
  currentDatetime.value = format(new Date(), "yyyy/MM/dd HH:mm:ss");
};
const displayCountedDate = () => {
  return format(countedDate(), "yyyy/MM/dd");
}
// 時刻
const dayCount = ref(0);
const incrementHour = () => {
  dayCount.value += 1 / 24;
}
const countedDate = () => { return addDays(new Date(1900, 0, 1), dayCount.value) };
// スピード
const beatSpeed = ref(100);
onMounted(() => {
  getCurrentDatetime();
  setInterval(getCurrentDatetime, 100);
  setInterval(incrementHour, 1000 / beatSpeed.value);
});
</script>

<template>
  <div class="">
    <div class="">
      <MyCanvas :day-count="dayCount" />
      <h1>{{ displayCountedDate() }}</h1>
    </div>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}

h1 {
  color: beige;
}
</style>
