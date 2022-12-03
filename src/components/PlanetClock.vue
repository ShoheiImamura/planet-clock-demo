<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import SolarSystemCanvas from "./SolarSystemCanvas.vue";
import EarthRotationCanvas from "./EarthRotationCanvas.vue";
import { getUnixTime, fromUnixTime, getYear, getDate, getSeconds, getMonth, getHours, getMinutes } from "date-fns";
import { format as formatTZ } from 'date-fns-tz';

defineProps({
  msg: String,
  displayMenu: {
    type: Boolean,
    default: false,
  }
});

const baseUnixTime = getUnixTime(new Date(1900, 2, 21, 0, 0, 0)); // 基準日時
const unixTimeCount = ref(0); // 基準日時からの経過UnixTime
const dayCount = () => {
  return unixTimeCount.value / (60 * 60 * 24)
}; // 基準日時からの経過日数

const dayUnixTimeCount = () => {
  return unixTimeCount.value % (60 * 60 * 24)
}; // 一日の中の経過unixtime

const countedDateTime = () => { return fromUnixTime(baseUnixTime + unixTimeCount.value) };

// 早送り
const beatSpeed = ref(360); // 1秒間の count 回数
const timePerCount = ref(1); // 1カウントごとに進めるunixtime
const incrementTime = () => {
  if (mode.value === 'auto_increment') {
    unixTimeCount.value += timePerCount.value;
  }
}

let fastForwardInterval: number; // 早送り
let currentInterval: number; // 現在時刻


// 現在の UNIX time を設定
const setCurrentDateTime = () => {
  const currentUnixTime = getUnixTime(new Date());
  unixTimeCount.value = currentUnixTime - baseUnixTime
}

// 操作用
const manualDateTime = ref({
  year: 1990,
  month: 2,
  day: 21,
  hour: 0,
  minute: 0,
  second: 0,
});
const setManualDateTime = () => {
  const manualUnixTime = getUnixTime(new Date(
    manualDateTime.value.year,
    manualDateTime.value.month,
    manualDateTime.value.day,
    manualDateTime.value.hour,
    manualDateTime.value.minute,
    manualDateTime.value.second));
  unixTimeCount.value = manualUnixTime - baseUnixTime;
}
const setTimeToManualDateTime = () => {
  const dateTime = countedDateTime();
  manualDateTime.value = {
    year: getYear(dateTime),
    month: getMonth(dateTime),
    day: getDate(dateTime),
    hour: getHours(dateTime),
    minute: getMinutes(dateTime),
    second: getSeconds(dateTime),
  }
}

// モード
const mode = ref('current'); // current, auto_increment, manual

const setCurrentMode = () => {
  clearInterval(fastForwardInterval);
  setCurrentDateTime();
  currentInterval = setInterval(setCurrentDateTime, 1000); // 毎秒カウントアップ
}

const setManualMode = () => {
  clearInterval(fastForwardInterval);
  clearInterval(currentInterval);
  setTimeToManualDateTime();
}

const setFastForwardMode = () => {
  clearInterval(currentInterval);
  fastForwardInterval = setInterval(incrementTime, 1000 / beatSpeed.value); // 1時間(1/24日)づつカウント
}
// モード変更検知
watch(mode, () => {
  if (mode.value == 'manual') {
    setManualMode()
  } else if (mode.value == 'current') {
    setCurrentMode();
  } else {
    setFastForwardMode();
  }
})

// 操作時変更検知
watch(manualDateTime, () => {
  setManualDateTime();
}, { deep: true })

// 表示用（年月日）
const displayCountedDate = () => {
  return formatTZ(countedDateTime(), "yyyy/MM/dd", { timeZone: 'Asia/Tokyo' });
}
// 表示用（時分秒）
const displayCountedTime = () => {
  return formatTZ(countedDateTime(), "HH:mm:ss", { timeZone: 'Asia/Tokyo' });
}

// 初期表示の設定
onMounted(() => {
  setCurrentMode();
});

const model = ref(null)
</script>

<template>
  <div class="">
    <v-card>
      <v-row class="mt-2 d-flex justify-center" v-if="displayMenu">
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
      <v-row no-gutters class="d-flex d-sm-none">
        <!-- mobile 表示の場合はカルーセル -->
        <v-carousel v-model="model" class="pa-4" :show-arrows="false" :hide-delimiters="true" :height="450">
          <v-carousel-item>
            <v-card class="ma-4" flat>
              <div class="fill-height align-center justify-center">
                <SolarSystemCanvas :day-count="dayCount()" class="ma-2 pa-2" />
                <h2 class="text-center">{{ displayCountedDate() }}</h2>
                <h3 class="text-center">{{ displayCountedTime() }}</h3>
              </div>
            </v-card>
          </v-carousel-item> <v-carousel-item>
            <v-card class="ma-4" flat>
              <div class="fill-height align-center justify-center">
                <EarthRotationCanvas :day-count="dayCount()" :day-unix-time-count="dayUnixTimeCount()"
                  class="ma-2 pa-2" />
                <h3 class="text-center">{{ displayCountedDate() }}</h3>
                <h2 class="text-center">{{ displayCountedTime() }}</h2>
              </div>
            </v-card>
          </v-carousel-item>
        </v-carousel>
        <!-- tablet 以上の場合は横並び -->
      </v-row>
      <v-row no-gutters class="d-none d-sm-flex">
        <v-col cols="6">
          <v-card class="ma-4" flat>
            <div class="fill-height align-center justify-center">
              <SolarSystemCanvas :day-count="dayCount()" class="ma-2 pa-2" />
              <h2 class="text-center">{{ displayCountedDate() }}</h2>
            </div>
          </v-card>
        </v-col>
        <v-col cols="6">
          <v-card class="ma-4" flat>
            <div class="fill-height align-center justify-center">
              <EarthRotationCanvas :day-count="dayCount()" :day-unix-time-count="dayUnixTimeCount()"
                class="ma-2 pa-2" />
              <h2 class="text-center">{{ displayCountedTime() }}</h2>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
    <!-- 操作時 -->
    <v-card v-show="mode == 'manual'" class="pa-2">
      <v-slider v-model="manualDateTime.year" min="1900" max="2200" density="compact" label=" year"></v-slider>
      <v-slider v-model="manualDateTime.month" min="0" max="11" density="compact" label="month"></v-slider>
      <v-slider v-model="manualDateTime.day" min="1" max="31" density="compact" label="  day"></v-slider>
      <v-slider v-model="manualDateTime.hour" min="0" max="24" density="compact" label="  hour"></v-slider>
      <v-slider v-model="manualDateTime.minute" min="0" max="60" density="compact" label="  minute"></v-slider>
    </v-card>
    <!-- 早送り時 -->
    <v-card v-if="mode == 'auto_increment'" class="d-flex justify-center align-center">
      <v-btn-toggle v-model="timePerCount">
        <v-btn :value="0">
          <v-icon>mdi-stop</v-icon>
        </v-btn>
        <v-btn :value="(1)">
          <v-icon>mdi-play</v-icon>
        </v-btn>
        <v-btn :value="(60)">
          <v-icon>mdi-fast-forward</v-icon>
        </v-btn>
        <v-btn :value="(60 * 60)">
          <v-icon>mdi-skip-forward</v-icon>
        </v-btn>
      </v-btn-toggle>
    </v-card>
  </div>
</template>

<style scoped>

</style>
