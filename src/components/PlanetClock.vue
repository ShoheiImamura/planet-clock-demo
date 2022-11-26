<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import SolarSystemCanvas from "./SolarSystemCanvas.vue";
import MoonSyzygyCanvas from "./MoonSyzygyCanvas.vue";
import { format, addDays, differenceInDays, getYear, getMonth, getDate } from "date-fns";

defineProps({
  msg: String,
});

const mode = ref('auto_increment'); // now, auto_increment, manual
const calendarYear = ref(1900);
const calendarMonth = ref(2);
const calendarDay = ref(21);
const calendarHour = ref(0);
const calendarMinute = ref(0);
const calendarSecond = ref(0);
const displayCalendar = () => {
  return new Date(calendarYear.value, calendarMonth.value, calendarDay.value,
    calendarHour.value, calendarMinute.value, calendarSecond.value);
}

const setCurrentDateTime = () => {
  const currentDateTime = (new Date());
  calendarYear.value = getYear(currentDateTime);
  calendarMonth.value = getMonth(currentDateTime);
  calendarDay.value = getDate(currentDateTime);
}

const displayCountedDate = () => {
  return format(countedDate(), "yyyy/MM/dd");
}
const displayCountedTime = () => {
  return format(countedDate(), "hh:mm:ss");
}

const baseDate = new Date(1900, 2, 21); // 基準日
const dayCount = ref(0); // 基準日からの経過日数

const countedDate = () => { return addDays(baseDate, dayCount.value) };
const beatSpeed = ref(40); // 1秒間のカウント数の逆数
const speed = ref(1 / 24);

const incrementHour = () => {
  if (mode.value === 'auto_increment') {
    dayCount.value += speed.value;
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
  } else if (mode.value == 'current') {
    dayCount.value = calendarDayFromBaseDate();
  }
})

watch(mode, () => {
  if (mode.value == 'manual') {
  } else if (mode.value == 'current') {
    setCurrentDateTime();
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
const model = ref(null)
</script>

<template>
  <div class="">
    <v-card>
      <v-row class="mt-2 d-flex justify-center">
        <v-btn-toggle v-model="mode" dark text mandatory>
          <v-btn value="current">
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
        <v-carousel v-model="model" class="pa-4" :show-arrows="false" :hide-delimiters="true" :height="420">
          <v-carousel-item>
            <v-card class="ma-4" flat>
              <div class="fill-height align-center justify-center">
                <SolarSystemCanvas :day-count="dayCount" class="ma-2 pa-2" />
                <h2 class="text-center">{{ displayCountedDate() }}</h2>
              </div>
            </v-card>
          </v-carousel-item>
          <v-carousel-item>
            <v-card color="grey-lighten-1" class="ma-4">
              <div class="fill-height align-center justify-center">
                <MoonSyzygyCanvas :day-count="dayCount" class="ma-2 pa-2" />
                <h2 class="text-center">{{ displayCountedTime() }}</h2>
              </div>
            </v-card>
          </v-carousel-item>
        </v-carousel>
      </v-row>
    </v-card>
    <!-- 操作時 -->
    <v-card v-show="mode == 'manual'" class="pa-2">
      <v-slider v-model="calendarYear" min="1900" max="2200" density="compact" label=" year"></v-slider>
      <v-slider v-model="calendarMonth" min="0" max="11" density="compact" label="month"></v-slider>
      <v-slider v-model="calendarDay" min="1" max="31" density="compact" label="  day"></v-slider>
    </v-card>
    <!-- 早送り時 -->
    <v-card v-if="mode == 'auto_increment'" class="d-flex justify-center align-center">
      <v-btn-toggle v-model="speed">
        <v-btn :value="0">
          <v-icon>mdi-stop</v-icon>
        </v-btn>
        <v-btn :value="1 / 24">
          <v-icon>mdi-play</v-icon>
        </v-btn>
        <v-btn :value="6 / 24">
          <v-icon>mdi-fast-forward</v-icon>
        </v-btn>
        <v-btn :value="6">
          <v-icon>mdi-skip-forward</v-icon>
        </v-btn>
      </v-btn-toggle>
    </v-card>
  </div>
</template>

<style scoped>

</style>
