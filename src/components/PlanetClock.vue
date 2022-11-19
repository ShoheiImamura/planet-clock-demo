<script setup lang="ts">
import { onMounted, ref } from "vue";
import MyCanvas from "./MyCanvas.vue";
import { format, getTime } from "date-fns";

defineProps({
  msg: String,
});

const currentDatetime = ref('');

const getCurrentDatetime = () => {
  currentDatetime.value = format(new Date(), "yyyy/MM/dd HH:mm:ss");
  // currentDatetime.value = getTime(new Date());
};

onMounted(() => {
  getCurrentDatetime();
  setInterval(getCurrentDatetime, 100);
});
</script>

<template>
  <div class="">
    <div class="">
      <MyCanvas />
      <h1>{{ currentDatetime }}</h1>
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
