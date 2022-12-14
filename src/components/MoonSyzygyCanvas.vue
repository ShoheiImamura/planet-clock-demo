
<script lang="ts" setup>

import { ref, onMounted } from "vue";
import { planet } from "./Planet.vue";
const { TheMoon } = planet();


const props = defineProps({
  dayCount: { type: Number, default: 0, }
});

type Coordinate = {
  x: number;
  y: number;
};
const ctx = ref<CanvasRenderingContext2D | null>(null);
const canvasScale = ref(30);
const centerCoordinate = ref<Coordinate>({
  x: canvasScale.value,
  y: canvasScale.value,
});

/** 日数から年数への変換 */
const dayToYear = (day: number): number => {
  return day / 365.25;
}

onMounted(() => {
  // canvas要素を取得
  const canvas = document.getElementById("moon-syzygy-canvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");

  if (ctx.value === null) return;

  drawMoon();
  setInterval(drawMoon, 10);
});

// 月の描画
const drawMoon = () => {
  const moon = TheMoon;
  const year = dayToYear(props.dayCount);
  const radius = canvasScale.value * 2 / 3;
  const angle = moon.angle(year) - moon.planet.angle(year)

  drawCircle(radius);
  if (Math.sin(angle) >= 0 && Math.cos(angle) >= 0) {
    // 満月 -> 下弦
    drawSemicirlce(radius, Math.PI * 1 / 2);
    drawEllipse(radius, radius * Math.cos(angle));
  } else if (Math.sin(angle) >= 0 && Math.cos(angle) < 0) {
    // 下弦 -> 新月
    drawSemicirlce(radius, Math.PI * 1 / 2);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), 'DimGray');
  } else if (Math.sin(angle) < 0 && Math.cos(angle) < 0) {
    // 新月 -> 上弦
    drawSemicirlce(radius, Math.PI * 3 / 2);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), 'DimGray');
  } else {
    // 上弦 -> 満月
    drawSemicirlce(radius, Math.PI * 3 / 2);
    drawEllipse(radius, radius * Math.cos(angle),);
  }
}
// そのままの月
const drawCircle = (
  radius: number,
  fillColor: string = "DimGray",
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
const drawSemicirlce = (
  radius: number,
  rotation: number,
  fillColor: string = "beige",
  startAngle: number = 0,
  endAngle: number = Math.PI * 1
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    radius,
    rotation + startAngle,
    rotation + endAngle
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};
const drawEllipse = (
  radiusX: number, radiusY: number,
  fillColor: string = "beige",
  startAngle: number = 0,
  endAngle: number = Math.PI * 2
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.ellipse(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    radiusX,
    radiusY,
    Math.PI / 2,
    // 0,
    startAngle,
    endAngle
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
}

</script>

<style scoped>
.canvas {
  border: 1px solid #333;
}
</style>

<template>
  <div>
    <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="moon-syzygy-canvas"> </canvas>
  </div>
</template>
