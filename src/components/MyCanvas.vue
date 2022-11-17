
<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import { planet } from './Planet.vue'
import type { Planet } from './Planet.vue'

const ctx = ref<CanvasRenderingContext2D | null>(null);

// 惑星
const { Mercury, Venus, Earth, Jupiter, Saturn } = planet();

onMounted(() => {
  console.log("on mounted...");
  // canvas要素を取得
  const canvas = document.getElementById("canvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");
  if (ctx.value === null) return;

  ctx.value.clearRect(0, 0, 400, 400);

  // 背景
  drawBackGround(200);

  // 水星
  drawPlanetaryOrbit(Mercury);
  drawPlanetaryArcArea(Mercury, 4, 1);
  drawPlanet(Mercury, 4);

  // 金星
  drawPlanetaryOrbit(Venus);
  drawPlanetaryArcArea(Venus, 1, 1);
  drawPlanet(Venus, 1);

  // 地球
  drawPlanetaryOrbit(Earth);
  drawPlanetaryArcArea(Earth, 23, 1);
  drawPlanet(Earth, 23);


  // 木星
  drawPlanetaryOrbit(Jupiter);
  drawPlanetaryArcArea(Jupiter, 15, 1);
  drawPlanet(Jupiter, 15);

  // 土星
  drawPlanetaryOrbit(Saturn);
  drawPlanetaryArcArea(Saturn, 25, 1);
  drawPlanet(Saturn, 25);

  // 太陽（恒星）
  drawStar(3);

  // 月（衛星）
  drawSatellite(Earth.radius, Earth.angularVelocity * 23);
  drawSatelliteCircleLine(
    Earth.radius,
    Earth.angularVelocity * 23,
    20
  );

});

// 背景
const drawBackGround = (radius: number) => {
  drawFilledCircle(radius, "black");
};

// 恒星を描画
const drawStar = (radius: number) => {
  drawFilledCircle(radius, "rgb(90,20,30)"); // 太陽の色
};

// 衛星を描画
const drawSatellite = (
  radius: number,
  radians: number = 0,
  fillColor: string = "rgba(255,255,200,0.9)"
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(radians, radius);
  ctx.value.arc(x, y, 3, 0, Math.PI * 2);

  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
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
  const { x, y } = getXYByRadians(planet.angularVelocity * time, planet.radius);
  ctx.value.arc(x, y, 3, 0, Math.PI * 2);

  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};

// 位置を導出
const getXYByRadians = (radians: number, radius: number) => {
  const y = 200 + Math.sin(radians) * radius;
  const x = 200 + Math.cos(radians) * radius;
  return {
    x: x,
    y: y,
  };
};

// 丸塗り潰し
const drawFilledCircle = (radius: number, fillColor: string = "black") => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.arc(200, 200, radius, 0, Math.PI * 2);
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};
// 惑星軌道を描画
const drawPlanetaryOrbit = (
  planet: Planet,
  lineColor: string = "rgba(99,99,99, 0.6)" // 軌道の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(200, 200, planet.radius, 0, 2 * Math.PI);
  ctx.value.stroke();
};

// 惑星円弧塗り潰し
const drawFilledArc = (
  radius: number,
  radiansStart: number = 0, // 開始角度
  radians: number = Math.PI * (1 / 4), // 角度
  fillColor: string = "rgba(99,99,99, 0.5)" // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();

  ctx.value.moveTo(200, 200);
  ctx.value.arc(200, 200, radius, radiansStart, radiansStart + radians);
  ctx.value.fillStyle = fillColor;
  ctx.value.closePath();
  ctx.value.fill();
};

// 惑星円弧塗り潰し
const drawPlanetaryArcArea = (
  planet: Planet,
  time: number = 0, // 開始角度
  period: number = 1, // 角度
  fillColor: string = "rgba(99,99,99, 0.75)" // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();

  ctx.value.moveTo(200, 200);
  ctx.value.arc(200, 200, planet.radius, planet.angularVelocity * time, planet.angularVelocity * (time + period));
  ctx.value.fillStyle = fillColor;
  ctx.value.closePath();
  ctx.value.fill();
};

// 衛星軌道
const drawSatelliteCircleLine = (
  planetRadius: number,
  planetRadians: number,
  satelliteRadius: number,
  lineColor: string = "rgba(99,99,99, 0.4)" // 軌道の色
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x, y } = getXYByRadians(planetRadians, planetRadius);
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(x, y, satelliteRadius, 0, 2 * Math.PI);
  ctx.value.stroke();
};

// 衛星描画
const drawSatellitePointCircle = (
  planetRadius: number,
  planetRadians: number,
  satelliteRadius: number,
  satelliteRadians: number,
  lineColor: string = "rgba(99,99,99, 0.4)" // 軌道の色
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x, y } = getXYByRadians(planetRadians, planetRadius);
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(x, y, satelliteRadius, 0, 2 * Math.PI);
  ctx.value.stroke();
};
</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}
</style>
<template>
  <canvas width="400" height="400" class="canvas" id="canvas"> </canvas>
</template>
