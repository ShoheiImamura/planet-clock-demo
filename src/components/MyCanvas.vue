
<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
// defineProps({});

const ctx = ref<CanvasRenderingContext2D | null>(null);

// キャンバス全体の設定

// 水星
const Mercury = ref({
  radius: 60,
  angularVelocity: Math.PI * (6 / 24),
});

// 金星
const Venus = ref({
  radius: 80,
  angularVelocity: Math.PI * (4 / 24),
});

// 地球
const Earth = ref({
  radius: 120,
  angularVelocity: Math.PI * (2 / 24),
});
// 木星
const Jupiter = ref({
  radius: 150,
  angularVelocity: Math.PI * (1 / 24),
});
// 土星
const Saturn = ref({
  radius: 180,
  angularVelocity: Math.PI * (1 / 24),
});

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
  drawCircleLine(Mercury.value.radius);
  drawFilledArc(
    Mercury.value.radius,
    Mercury.value.angularVelocity * 4,
    Mercury.value.angularVelocity
  );
  drawPointCircle(Mercury.value.radius, Mercury.value.angularVelocity * 4);

  // 金星
  drawCircleLine(Venus.value.radius);
  drawFilledArc(
    Venus.value.radius,
    Venus.value.angularVelocity,
    Venus.value.angularVelocity
  );
  drawPointCircle(Venus.value.radius, Venus.value.angularVelocity);

  // 地球
  drawCircleLine(Earth.value.radius);
  drawFilledArc(
    Earth.value.radius,
    Earth.value.angularVelocity * 23,
    Earth.value.angularVelocity
  );
  drawPointCircle(Earth.value.radius, Earth.value.angularVelocity * 23);

  // 月（衛星）
  drawPointCircle(Earth.value.radius, Earth.value.angularVelocity * 23);
  drawSatelliteCircleLine(
    Earth.value.radius,
    Earth.value.angularVelocity * 23,
    20
  );

  // 木星
  drawCircleLine(Jupiter.value.radius);
  drawFilledArc(
    Jupiter.value.radius,
    Jupiter.value.angularVelocity * 15,
    Jupiter.value.angularVelocity
  );
  drawPointCircle(Jupiter.value.radius, Jupiter.value.angularVelocity * 15);

  // 土星
  drawCircleLine(Saturn.value.radius);
  drawFilledArc(
    Saturn.value.radius,
    Saturn.value.angularVelocity * 25,
    Saturn.value.angularVelocity
  );
  drawPointCircle(Saturn.value.radius, Saturn.value.angularVelocity * 25);

  // 太陽
  drawCenterPoint(3);
});

// 背景
const drawBackGround = (radius: number) => {
  drawFilledCircle(radius, "black");
};

// 中心点
const drawCenterPoint = (radius: number) => {
  drawFilledCircle(radius, "rgb(90,20,30)"); // 太陽の色
};

// 惑星
const drawPointCircle = (
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

// 惑星軌道
const drawCircleLine = (
  radius: number,
  lineColor: string = "rgba(99,99,99, 0.4)" // 軌道の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(200, 200, radius, 0, 2 * Math.PI);
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
