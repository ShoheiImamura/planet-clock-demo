<script setup lang="ts">
import { ref } from 'vue';
import PlanetClock from "./components/PlanetClock.vue";

const displayMenu = ref(false);
</script>

<template>
  <v-app>
    <v-app-bar title="PLANET CLOCK" class="text-center">
      <v-app-bar-nav-icon @click="(displayMenu = !displayMenu)"><v-icon
          btn>mdi-dots-vertical</v-icon></v-app-bar-nav-icon>
    </v-app-bar>
    <v-main>
      <PlanetClock :display-menu="displayMenu" />
    </v-main>
  </v-app>
</template>

<style scoped>

</style>
