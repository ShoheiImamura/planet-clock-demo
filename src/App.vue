<script setup lang="ts">
import { ref } from 'vue';
import PlanetClock from "./components/PlanetClock.vue";

// const displayMenu = ref(false);
</script>

<template >
  <v-app>
    <v-app-bar title="PLANET CLOCK" class="text-center"></v-app-bar>
    <v-main>
      <PlanetClock />
    </v-main>
  </v-app>
</template>

<style scoped>

</style>
