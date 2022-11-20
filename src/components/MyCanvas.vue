
<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import { planet } from "./Planet.vue";
import type { Planet, Moon } from "./Planet.vue";

type Coordinate = {
  x: number;
  y: number;
};

const ctx = ref<CanvasRenderingContext2D | null>(null);
const canvasScale = ref(200);
const centerCoordinate = ref<Coordinate>({
  x: canvasScale.value,
  y: canvasScale.value,
});

// 惑星
const { Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, TheMoon } =
  planet();

// 時刻
const monthCount = ref(0);
const incrementMonth = () => {
  if (beatSpeed.value <= 10 && beatSpeed.value >= 0) {
    monthCount.value += Number(beatSpeed.value);
  }
};
const timeScale = ref(1 / 3600);
// スピード
const beatSpeed = ref(5);

onMounted(() => {
  // canvas要素を取得
  const canvas = document.getElementById("canvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");
  if (ctx.value === null) return;
  drawSolarSystem();
  setInterval(incrementMonth, 50); // todo 例示のために表示 後で消す
});

watch(monthCount, () => {
  // 変更が発生するごとに再描画
  drawSolarSystem();
});

const drawSolarSystem = () => {
  if (ctx.value === null) return;

  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);

  // 背景
  drawFilledCircle(canvasScale.value);

  // 天王星
  drawPlanetaryOrbit(Uranus);
  drawPlanetaryArcArea(Uranus, monthCount.value, 1);
  drawPlanet(Uranus, monthCount.value);
  drawFilledCircle(Uranus.radius - 10); // デザイン調整
  // 土星
  drawPlanetaryOrbit(Saturn);
  drawPlanetaryArcArea(Saturn, monthCount.value, 1);
  drawPlanet(Saturn, monthCount.value);
  drawFilledCircle(Saturn.radius - 10); // デザイン調整
  // 木星
  drawPlanetaryOrbit(Jupiter);
  drawPlanetaryArcArea(Jupiter, monthCount.value, 1);
  drawPlanet(Jupiter, monthCount.value);
  drawBackGround(Jupiter.radius - 18); // デザイン調整
  // 火星
  drawPlanetaryOrbit(Mars);
  drawPlanetaryArcArea(Mars, monthCount.value, 1 / 12);
  drawPlanet(Mars, monthCount.value);
  // 地球
  drawPlanetaryOrbit(Earth);
  drawPlanetaryArcArea(Earth, monthCount.value, 1 / 12);
  drawPlanet(Earth, monthCount.value);
  // 金星
  drawPlanetaryOrbit(Venus);
  drawPlanetaryArcArea(Venus, monthCount.value, 1 / 12);
  drawPlanet(Venus, monthCount.value);

  // 水星
  drawPlanetaryOrbit(Mercury);
  drawPlanetaryArcArea(Mercury, monthCount.value, 1 / 12);
  drawPlanet(Mercury, monthCount.value);

  // 太陽（恒星）
  drawStar(3);

  // 月（衛星）
  drawMoonOrbit(TheMoon, monthCount.value);
  drawMoonArcArea(TheMoon, monthCount.value, 1 / 48);
  drawMoon(TheMoon, monthCount.value);
};

// 背景
const drawBackGround = (radius: number) => {
  drawFilledCircle(radius, "black", Math.PI * 0 / 4, Math.PI * 1 / 4);
  drawFilledCircle(radius, "rgb(30,30,30)", Math.PI * 1 / 4, Math.PI * 2 / 4);
  drawFilledCircle(radius, "black", Math.PI * 2 / 4, Math.PI * 3 / 4);
  drawFilledCircle(radius, "rgb(30,30,30)", Math.PI * 3 / 4, Math.PI * 4 / 4);
  drawFilledCircle(radius, "black", Math.PI * 4 / 4, Math.PI * 5 / 4);
  drawFilledCircle(radius, "rgb(30,30,30)", Math.PI * 5 / 4, Math.PI * 6 / 4);
  drawFilledCircle(radius, "black", Math.PI * 6 / 4, Math.PI * 7 / 4);
  drawFilledCircle(radius, "rgb(30,30,30)", Math.PI * 7 / 4, Math.PI * 8 / 4);
};

// 恒星を描画
const drawStar = (radius: number) => {
  drawFilledCircle(radius, "rgb(90,20,30)"); // 太陽の色
};

// 惑星を描画
const drawPlanet = (
  planet: Planet,
  time: number = 0,
  fillColor: string = "rgba(255,255,200,0.9)"
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    planet.angularVelocity * time * timeScale.value,
    planet.radius
  );
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    3,
    0,
    Math.PI * 2
  );

  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};

// 位置を導出
const getXYByRadians = (radians: number, radius: number): Coordinate => {
  const x = Math.cos(radians) * radius;
  const y = -Math.sin(radians) * radius;
  return {
    x: x,
    y: y,
  };
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
    planet.radius,
    0,
    2 * Math.PI
  );
  ctx.value.stroke();
};

// 惑星円弧塗り潰し
const drawPlanetaryArcArea = (
  planet: Planet,
  time: number = 0, // 開始角度
  period: number = 1, // 角度
  fillColor: string = "rgba(255,255,255, 0.5)" // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();

  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    planet.radius,
    planet.angularVelocity * time * -1 * timeScale.value,
    planet.angularVelocity * (time * -1 * timeScale.value + period)
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.closePath();
  ctx.value.fill();
};

// 衛星軌道
const drawMoonOrbit = (
  moon: Moon,
  time: number = 0,
  lineColor: string = "rgba(99,99,99, 0.4)" // 軌道の色
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x, y } = getXYByRadians(
    moon.planet.angularVelocity * time * timeScale.value,
    moon.planet.radius
  );
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    moon.radius,
    0,
    2 * Math.PI,
    true
  );
  ctx.value.stroke();
};

// 衛星を描画
const drawMoon = (
  moon: Moon,
  time: number = 0,
  fillColor: string = "rgba(255,255,200,0.9)"
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x: planetX, y: planetY } = getXYByRadians(
    moon.planet.angularVelocity * time * timeScale.value,
    moon.planet.radius
  );
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    moon.angularVelocity * time * timeScale.value,
    moon.radius
  );
  ctx.value.arc(
    centerCoordinate.value.x + planetX + x,
    centerCoordinate.value.y + planetY + y,
    2,
    0,
    Math.PI * 2
  );

  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};

// 衛星円弧塗り潰し
const drawMoonArcArea = (
  moon: Moon,
  time: number = 0, // 開始角度
  period: number = 1, // 角度
  fillColor: string = "rgba(255,255,255, 0.5)" // 扇形の色
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x: planetX, y: planetY } = getXYByRadians(
    moon.planet.angularVelocity * time * timeScale.value,
    moon.planet.radius
  );
  ctx.value.beginPath();

  // 惑星位置まで移動
  ctx.value.moveTo(
    centerCoordinate.value.x + planetX,
    centerCoordinate.value.y + planetY
  );
  ctx.value.arc(
    centerCoordinate.value.x + planetX,
    centerCoordinate.value.y + planetY,
    moon.radius,
    moon.angularVelocity * time * -1 * timeScale.value,
    moon.angularVelocity * (time * -1 * timeScale.value + period)
  );

  ctx.value.fillStyle = fillColor;
  ctx.value.closePath();
  ctx.value.fill();
};
</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}

.input-range[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  background-color: #c7c7c7;
  background: #c7c7c7;
  border-color: #c7c7c7;
  color: transparent;
  height: 2px;
  width: 100%;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 50%;
  background: #c7c7c7;
  cursor: ew-resize;
  box-shadow: 0 0 2px 0 #555;
  transition: .3s ease-in-out;
}
</style>

<template>
  <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="canvas">
  </canvas>
  <input type="range" v-model="beatSpeed" min="0" max="10" class="input-range">
</template>
