
<script lang="ts" setup>
import { ref, onMounted, watch, defineProps } from "vue";
import { planet } from "./Planet.vue";
import { Planet, Moon, } from "./Planet.vue";

const props = defineProps({
  dayCount: {
    type: Number,
    default: 0,
  },
  dayUnixTimeCount: { // 1日の中での unix time
    type: Number,
    default: 0,
  }
});

type Coordinate = {
  x: number;
  y: number;
};

const ctx = ref<CanvasRenderingContext2D | null>(null);
const canvasScale = ref(150);
const centerCoordinate = ref<Coordinate>({
  x: canvasScale.value,
  y: canvasScale.value,
});

// 惑星
const { Earth, TheMoon } = planet();

onMounted(() => {
  // canvas要素を取得
  const canvas = document.getElementById("earthRotationCanvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");
  if (ctx.value === null) return;
  setInterval(drawEarthRotation, 100);
});

const drawEarthRotation = () => {
  if (ctx.value === null) return;
  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);
  drawBackGround(140);
  drawEarth(Earth, props.dayCount, 75);
  drawSun(Earth, props.dayCount, 3);
  drawLineSunToEarth(Earth, props.dayCount);
  drawPlace(3, Earth, props.dayCount, props.dayUnixTimeCount);
  drawPlaceLine(3, Earth, props.dayCount, props.dayUnixTimeCount);
}

/** 日数から年数への変換 */
const dayToYear = (day: number): number => {
  return day / 365.25;
}

// 位置を導出
const getXYByRadians = (radians: number, radius: number): Coordinate => {
  const x = Math.cos(radians) * radius;
  const y = -Math.sin(radians) * radius;
  return {
    x: x,
    y: y,
  };
};

// 背景
const drawBackGround = (radius: number) => {
  drawFilledCircle(radius, "black");
}

// 地球を描画
const drawEarth = (earth: typeof Earth, days: number, radius: number) => {
  const angle = (-1) * earth.angle(dayToYear(days));
  const halfPI = Math.PI * 0.5;
  drawFilledCircle(radius, "rgb(20,20,20)", (angle - halfPI), (angle + halfPI));
  drawFilledCircle(radius, "rgb(50,50,50)", (angle + halfPI), (angle + halfPI * 3));
};

// 地点を描画
const drawPlace = (radius: number, earth: typeof Earth, days: number, time: number) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24));// 場所の相対角度
  const { x, y } = getXYByRadians(
    placeAngle - earthAngle,
    75
  );
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    radius,
    0,
    Math.PI * 2
  );
  ctx.value.fillStyle = 'white';
  ctx.value.fill();
  ctx.value.closePath();
};


// 地点を描画
const drawPlaceLine = (radius: number, earth: typeof Earth, days: number, time: number) => {
  if (ctx.value === null) return;
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24));// 場所の相対角度
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    75,
    (-1) * (placeAngle - earthAngle),
    (-1) * (placeAngle - earthAngle),
  );
  ctx.value.strokeStyle = 'white';
  ctx.value.stroke();
  ctx.value.closePath();
};


// 恒星を描画
const drawSun = (
  earth: typeof Earth,
  days: number = 0, // 基準日からの日数
  StarRadius: number = 3,
  fillColor: string = "rgba(255,255,200,0.9)",
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    earth.angle(dayToYear(days)) + Math.PI, // 地球の方向と逆になる
    1.9 * canvasScale.value / 2
  );
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    StarRadius,
    0,
    Math.PI * 2
  );

  ctx.value.strokeStyle = 'white';
  ctx.value.stroke();
  ctx.value.closePath();
};


// 丸塗り潰し
const drawFilledCircle = (
  radius: number,
  fillColor: string = "black",
  startAngle: number = 0,
  endAngle: number = Math.PI * 2
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    radius,
    startAngle,
    endAngle
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};

// 惑星軌道を描画
const drawPlanetaryOrbit = (
  planet: Planet,
  lineColor: string = "rgba(130,130,130,0.6)" // 軌道の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    planet.radiusRatio * canvasScale.value / 2,
    0,
    2 * Math.PI
  );
  ctx.value.stroke();
  ctx.value.closePath();
};

// 恒星から惑星へのライン
const drawLineSunToEarth = (
  planet: Planet,
  days: number = 0,
  lineColor: string = "rgba(255,255,255, 0.9)", // 線の色
  lineWidth: number = 2, // 線の幅
  lineLengthRatio: number = 1.9 // 線の長さ比
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;
  const sunAngle = planet.angle(dayToYear(days)) + Math.PI;

  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    lineLengthRatio * canvasScale.value / 2,
    (-1) * sunAngle,
    (-1) * sunAngle,
  )
  ctx.value.stroke();

  ctx.value.strokeStyle = 'rgba(255,255,255, 0.5)';
  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    75,
    (-1) * (sunAngle + Math.PI),
    (-1) * (sunAngle + Math.PI),
  )
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
}


</script>

<style scoped>

</style>

<template>
  <div class="d-flex justify-center">
    <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="earthRotationCanvas">
    </canvas>
  </div>
</template>
