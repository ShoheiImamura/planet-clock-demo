<script setup lang="ts">
import { onMounted, ref } from "vue";
import MyCanvas from "./MyCanvas.vue";
import { format } from "date-fns";

defineProps({
  msg: String,
});

const radius = ref(50);
const count = ref(0);

const currentDatetime = ref("");

const getCurrentDatetime = () => {
  currentDatetime.value = format(new Date(), "yyyy/MM/dd HH:mm:ss");
};

onMounted(() => {
  getCurrentDatetime();
  setInterval(getCurrentDatetime, 1000);
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
