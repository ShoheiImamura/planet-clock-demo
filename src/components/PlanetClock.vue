<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import SolarSystemCanvas from "./SolarSystemCanvas.vue";
import MoonSyzygyCanvas from "./MoonSyzygyCanvas.vue";
import { format, addDays, set, differenceInDays, getYear, getMonth, getDate } from "date-fns";

defineProps({
  msg: String,
});

const mode = ref('auto_increment'); // now, auto_increment, manual
const calendarYear = ref(1900);
const calendarMonth = ref(2);
const calendarDay = ref(21);
const displayCalendar = () => {
  return new Date(calendarYear.value, calendarMonth.value, calendarDay.value);
}

const displayCountedDate = () => {
  return format(countedDate(), "yyyy/MM/dd");
}

const baseDate = new Date(1900, 2, 21); // 基準日
const dayCount = ref(0); // 基準日からの経過日数


const countedDate = () => { return addDays(baseDate, dayCount.value) };
const beatSpeed = ref(40); // 1秒間のカウント数の逆数

const incrementHour = () => {
  if (mode.value === 'auto_increment') {
    dayCount.value += 1 / 24;
  }
}
onMounted(() => {
  setInterval(incrementHour, 1000 / beatSpeed.value); // 1時間(1/24日)づつカウント
});

// 基準日と表示カレンダーのunixタイムを比較する
const calendarDayFromBaseDate = () => {
  return differenceInDays(displayCalendar(), baseDate);
}

watch(displayCalendar, () => {
  if (mode.value == 'manual') {
    dayCount.value = calendarDayFromBaseDate();
  }
})

const changeManualMode = () => {
  const countedDateTemp = countedDate();
  calendarYear.value = getYear(countedDateTemp)
  calendarMonth.value = getMonth(countedDateTemp)
  calendarDay.value = getDate(countedDateTemp)
  mode.value = 'manual';
}

</script>

<template>
  <div class="">
    <v-card>
      <v-row class="mt-2 d-flex justify-center">
        <v-btn-toggle v-model="mode" dark text>
          <v-btn value="now">
            現在
          </v-btn>
          <v-btn value="auto_increment">
            早送り
          </v-btn>
          <v-btn value="manual">
            操作
          </v-btn>
        </v-btn-toggle>
      </v-row>
      <v-row no-gutters>
        <v-col cols="12" sm="8">
          <SolarSystemCanvas :day-count="dayCount" class="ma-2 pa-2" />
          <h2 class="text-center">{{ displayCountedDate() }}</h2>
        </v-col>
        <v-col cols="12" sm="4">
          <MoonSyzygyCanvas :day-count="dayCount" class="ma-2 pa-2" />
        </v-col>
      </v-row>
    </v-card>
    <div>
    </div>

    <div>
      <button v-show="mode == 'manual'" @click="mode = 'auto_increment'">自動</button>
      <button v-show="mode == 'auto_increment'" @click="changeManualMode()">手動</button>

      <li v-show="mode == 'manual'">
        <ul>
          <input type="range" name="year" min="1900" max="2100" v-model="calendarYear">
        </ul>
        <ul>
          <input type="range" name="year" min="0" max="11" v-model="calendarMonth">
        </ul>
        <ul>
          <input type="range" name="year" min="1" max="31" v-model="calendarDay">
        </ul>
      </li>
    </div>

  </div>
</template>

<style scoped>

</style>
