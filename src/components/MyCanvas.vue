
<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import { planet } from "./Planet.vue";
import { Planet, Moon, planetaryConjunction } from "./Planet.vue";
import PlanetClock from "./PlanetClock.vue";

const props = defineProps({
  dayCount: Number,
});

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


const { innerConjunction, outerConjunction } = planetaryConjunction();

onMounted(() => {
  // canvas要素を取得
  const canvas = document.getElementById("canvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");
  if (ctx.value === null) return;
  drawSolarSystem();
  setInterval(drawSolarSystem, 10);
});


/** 日数から年数への変換 */
const dayToYear = (day: number): number => {
  return day / 365;
}

const drawSolarSystem = () => {
  if (ctx.value === null) return;

  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);

  // 背景
  drawFilledCircle(canvasScale.value);

  // 天王星
  drawBackGround(Uranus.radius);
  drawPlanetaryOrbit(Uranus);
  drawLineStarToPlanet(Uranus, props.dayCount);
  drawPlanet(Uranus, props.dayCount, 4);
  // 土星
  drawPlanetaryOrbit(Saturn);
  drawLineStarToPlanet(Saturn, props.dayCount);
  drawPlanet(Saturn, props.dayCount, 5);
  // 木星
  drawPlanetaryOrbit(Jupiter);
  drawLineStarToPlanet(Jupiter, props.dayCount);
  drawPlanet(Jupiter, props.dayCount, 6);
  // 火星
  drawPlanetaryOrbit(Mars);
  drawLineStarToPlanet(Mars, props.dayCount);
  drawPlanet(Mars, props.dayCount, 5);
  // 地球
  drawPlanetaryOrbit(Earth);
  drawLineStarToPlanet(Earth, props.dayCount);
  drawPlanet(Earth, props.dayCount, 4);
  // 金星
  drawPlanetaryOrbit(Venus);
  drawLineStarToPlanet(Venus, props.dayCount);
  drawPlanet(Venus, props.dayCount);
  // 水星
  drawPlanetaryOrbit(Mercury);
  drawLineStarToPlanet(Mercury, props.dayCount);
  drawPlanet(Mercury, props.dayCount);

  // 太陽（恒星）
  drawStar(3);

  // 月（衛星）
  drawMoonOrbit(TheMoon, props.dayCount);
  drawMoon(TheMoon, props.dayCount);
};

// 背景
const drawBackGround = (radius: number) => {
  const forCount = 12
  for (let index = 0; index < forCount; index++) {
    drawFilledCircle(radius, "black", Math.PI * (index * 2 + 0) / forCount, Math.PI * (index * 2 + 1) / forCount);
    drawFilledCircle(radius, "rgb(20,20,20)", Math.PI * (index * 2 + 1) / forCount, Math.PI * (index * 2 + 2) / forCount);
  }
};

// 恒星を描画
const drawStar = (radius: number) => {
  drawFilledCircle(radius, "rgba(255,255,200,0.9)"); // 太陽の色
};

// 惑星を描画
const drawPlanet = (
  planet: Planet,
  days: number = 0, // 基準日からの日数
  planetRadius: number = 3,
  fillColor: string = "rgba(255,255,200,0.9)",
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    planet.angle(dayToYear(days)),
    planet.radius
  );
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    planetRadius,
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

// 恒星から惑星へのライン
const drawLineStarToPlanet = (
  planet: Planet,
  days: number = 0,
  lineColor: string = "rgba(255,255,255, 0.5)" // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;

  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    planet.radius,
    (-1) * planet.angle(dayToYear(days)),
    (-1) * planet.angle(dayToYear(days)),
  )
  ctx.value.stroke();
  ctx.value.closePath();

}

// 衛星軌道
const drawMoonOrbit = (
  moon: Moon,
  days: number = 0,
  lineColor: string = "rgba(99,99,99, 0.4)" // 軌道の色
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x, y } = getXYByRadians(
    moon.planet.angle(dayToYear(days)),
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
  days: number = 0,
  fillColor: string = "rgba(255,255,200,0.9)"
) => {
  if (ctx.value === null) return;
  // 惑星の位置を導出
  const { x: planetX, y: planetY } = getXYByRadians(
    moon.planet.angle(dayToYear(days)),
    moon.planet.radius
  );
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    moon.angle(dayToYear(days)),
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

</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}
</style>

<template>
  <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="canvas">
  </canvas>

</template>
