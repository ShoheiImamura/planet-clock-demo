<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import SolarSystemCanvas from "./SolarSystemCanvas.vue";
import EarthRotationCanvas from "./EarthRotationCanvas.vue";
import { getUnixTime, fromUnixTime, getYear, getHours, getMinutes, getDayOfYear } from "date-fns";
import { format as formatTZ } from 'date-fns-tz';
import { theMoonOrbitalPeriodUnixTime } from './Planet.vue';


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

// 月の満ち欠けカウント
const moonUnixTimeBase = ref({
  base: 0, // 基準値
  enough: 0, // あまり
  baseMin: 0, // 表示上の最小値
  baseMax: 0, // 表示上の最小値
});
const getMoonUnixTime = () => {
  moonUnixTimeBase.value.base = Math.floor(unixTimeCount.value / theMoonOrbitalPeriodUnixTime);
  moonUnixTimeBase.value.enough = unixTimeCount.value % theMoonOrbitalPeriodUnixTime;
  moonUnixTimeBase.value.baseMin = moonUnixTimeBase.value.enough % (60 * 60 * 24);
  moonUnixTimeBase.value.baseMax = moonUnixTimeBase.value.baseMin + (60 * 60 * 24) * 29;
}
const setMoonUnixTime = () => {
  const unixTimeCountTotal = moonUnixTimeBase.value.base * theMoonOrbitalPeriodUnixTime + moonUnixTimeBase.value.enough
  unixTimeCount.value = unixTimeCountTotal
}

// 早送り
const beatSpeed = ref(60); // 1秒間の count 回数
const timePerCount = ref(60); // 1カウントごとに進めるunixtime
const incrementTime = () => {
  if (mode.value === 'auto_increment') {
    unixTimeCount.value += Number(timePerCount.value);
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
  day: 80,
  hour: 0,
  minute: 0,
  second: 0,
});
const setManualDateTime = () => {
  const manualUnixTime = getUnixTime(new Date(
    manualDateTime.value.year,
    0,
    manualDateTime.value.day,
    manualDateTime.value.hour,
    manualDateTime.value.minute,
    manualDateTime.value.second
  ));
  unixTimeCount.value = manualUnixTime - baseUnixTime;
}
const setTimeToManualDateTime = () => {
  const dateTime = countedDateTime();

  manualDateTime.value = {
    year: getYear(dateTime),
    day: getDayOfYear(dateTime),
    hour: 0,
    minute: getHours(dateTime) * 60 + getMinutes(dateTime),
    second: 0, // 0 に初期化
  }
  if (manualDateTime.value.day < 80) {
    manualDateTime.value.day += 365
    manualDateTime.value.year -= 1
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
  getMoonUnixTime();
}, { deep: true })
watch(moonUnixTimeBase, () => {
  setMoonUnixTime();
  setTimeToManualDateTime();
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

const solarSystemCanvasDiv = ref();
const earthRotationCanvasDiv = ref();
const ifShowPlanet = ref(false)
</script>

<template>
  <div>
    <v-card class="">
      <v-row class="ma-2 d-flex justify-center " v-if="displayMenu">
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
    </v-card>
    <v-card class="">
      <v-row no-gutters class="clock-area">
        <v-col cols="12">
          <div class="align-center justify-center" ref="earthRotationCanvasDiv">
            <EarthRotationCanvas :day-count="dayCount()" :day-unix-time-count="dayUnixTimeCount()"
              :div-width="earthRotationCanvasDiv?.clientWidth" class="ma-2 pa-2" />
            <v-icon @click="ifShowPlanet = !ifShowPlanet" id="planet-toggle-icon" color="grey">mdi-compass</v-icon>
          </div>
        </v-col>
        <v-col cols="12" v-if="ifShowPlanet">
          <div class="align-center justify-center" ref="solarSystemCanvasDiv">
            <SolarSystemCanvas :day-count="dayCount()" :div-width="solarSystemCanvasDiv?.clientWidth"
              class="ma-2 pa-2" />
          </div>
        </v-col>
        <v-col cols="12">
          <h3 class="text-center">{{ displayCountedDate() }}</h3>
          <h1 class="text-center">{{ displayCountedTime() }}</h1>
        </v-col>
      </v-row>
    </v-card>
    <!-- 操作時 -->
    <v-card v-show="mode == 'manual'" class="ma-2 pr-6">
      <v-slider v-model="manualDateTime.year" min="1900" max="2200" density="compact" label="years"
        hide-details></v-slider>
      <v-slider v-model="manualDateTime.day" min="80" max="445" density="compact" label=" / year"
        hide-details></v-slider>
      <v-slider v-model="moonUnixTimeBase.enough" :min="moonUnixTimeBase.baseMin" :max="moonUnixTimeBase.baseMax"
        density="compact" :step="(60 * 60 * 24)" label=" / month" hide-details></v-slider>
      <v-slider v-model="manualDateTime.minute" min="0" max="1439" density="compact" label=" / day"
        hide-details></v-slider>
    </v-card>
    <!-- 早送り時 -->
    <v-card v-if="mode === 'auto_increment'" class="ma-2 d-flex justify-center align-center">
      <v-btn-toggle v-model="timePerCount" mandatory>
        <v-btn :value="0">
          <v-icon>mdi-stop</v-icon>
        </v-btn>
        <v-btn :value="1 / 60">
          <v-icon>mdi-play</v-icon>
        </v-btn>
        <v-btn :value="60">
          <v-icon>mdi-fast-forward</v-icon>
        </v-btn>
        <v-btn :value="(60 * 60 * 24)">
          <v-icon>mdi-skip-forward</v-icon>
        </v-btn>
      </v-btn-toggle>
    </v-card>
  </div>
</template>

<style scoped>
.clock-area {
  background-color: #111111;
}

#planet-toggle-icon {
  position: relative;
  bottom: 30px;
  left: 30px;
  z-index: 10;
}
</style>
