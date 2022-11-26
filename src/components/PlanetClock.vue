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
const incrementDay = () => {
  dayCount.value++;
}
const countedDate = () => { return addDays(new Date(1970, 1, 1), dayCount.value) };
// スピード
const beatSpeed = ref(10);
watch(dayCount, () => {

})

onMounted(() => {
  getCurrentDatetime();
  setInterval(getCurrentDatetime, 100);
  setInterval(incrementDay, 1000 / beatSpeed.value);
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
