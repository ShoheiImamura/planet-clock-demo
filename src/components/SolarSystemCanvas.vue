
<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import { planet } from "./Planet.vue";
import { Planet, Moon, } from "./Planet.vue";
import { planetaryConjunction } from "./PlanetaryConjunction.vue";
import type { planetaryConjunctionType } from "./PlanetaryConjunction.vue";

const props = defineProps({
  dayCount: {
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
const { Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, TheMoon } = planet();
// 惑星会合
const { isInnerConjunction, isOuterConjunction } = planetaryConjunction();

onMounted(() => {
  // canvas要素を取得
  const canvas = document.getElementById("solarSystemCanvas") as HTMLCanvasElement;
  ctx.value = canvas.getContext("2d");
  if (ctx.value === null) return;
  drawSolarSystem();
  setInterval(drawSolarSystem, 10);
});

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
const planetList = [Uranus, Saturn, Jupiter, Mars, Earth, Venus, Mercury];

// 惑星会合
const eventPlanetaryConjunction = (year: number) => {
  const conjunctionList: planetaryConjunctionType[] = [];
  planetList.forEach((planet1, index1) => {
    planetList.forEach((planet2, index2) => {
      if (index1 <= index2) {
        return;
      }
      if (isInnerConjunction(planet1, planet2, year)) {
        conjunctionList.push({ planet1: planet1, planet2: planet2, conjunctionType: 'inner' });
      }
      if (isOuterConjunction(planet1, planet2, year)) {
        conjunctionList.push({ planet1: planet1, planet2: planet2, conjunctionType: 'outer' });
      }
    });
  })
  return conjunctionList;
}

const drawSolarSystem = () => {
  if (ctx.value === null) return;

  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);

  // 背景
  drawBackGround(canvasScale.value);

  // 惑星
  const planetaryConjunctionList = eventPlanetaryConjunction(dayToYear(props.dayCount))
  if (planetaryConjunctionList.length !== 0) {
    // 惑星会合ありの場合
    planetList.forEach(planet => {
      // list に planet 自身が含まれるかどうか
      const isConjanction = planetaryConjunctionList.some((planetaryConjunction) => {
        return planet == planetaryConjunction.planet1 || planet == planetaryConjunction.planet2
      });
      if (isConjanction) {
        drawPlanetaryOrbit(planet);
        drawLineStarToPlanet(planet, props.dayCount, 'rgba(255,255,255,1)', 2);
        drawPlanet(planet, props.dayCount, planet.size);
      } else {
        drawPlanetaryOrbit(planet);
        drawLineStarToPlanet(planet, props.dayCount, 'rgba(120,120,120,0.5)',);
        drawPlanet(planet, props.dayCount, planet.size, 'rgba(120,120,120,0.5)');
      }
    });
  } else {
    // 惑星会合なしの場合
    planetList.forEach(planet => {
      drawPlanetaryOrbit(planet);
      drawLineStarToPlanet(planet, props.dayCount);
      drawPlanet(planet, props.dayCount, planet.size);
    });
  }

  // 太陽（恒星）
  drawStar(3);

  // 月（衛星）
  drawMoonOrbit(TheMoon, props.dayCount);
  drawMoon(TheMoon, props.dayCount);
};

// 背景
const drawBackGround = (radius: number) => {
  const forCount = 2
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
    planet.radiusRatio * canvasScale.value / 2
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
const drawLineStarToPlanet = (
  planet: Planet,
  days: number = 0,
  lineColor: string = "rgba(255,255,255, 0.5)", // 扇形の色
  lineWidth: number = 1 // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;

  ctx.value.moveTo(centerCoordinate.value.x, centerCoordinate.value.y);
  ctx.value.arc(
    centerCoordinate.value.x,
    centerCoordinate.value.y,
    planet.radiusRatio * canvasScale.value / 2,
    (-1) * planet.angle(dayToYear(days)),
    (-1) * planet.angle(dayToYear(days)),
  )
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
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
    moon.planet.radiusRatio * canvasScale.value / 2
  );
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(
    centerCoordinate.value.x + x,
    centerCoordinate.value.y + y,
    moon.radiusRatio * canvasScale.value / 2,
    0,
    2 * Math.PI,
    true
  );
  ctx.value.stroke();
  ctx.value.closePath();
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
    moon.planet.radiusRatio * canvasScale.value / 2
  );
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    moon.angle(dayToYear(days)),
    moon.radiusRatio * canvasScale.value / 2
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

</style>

<template>
  <div class="d-flex justify-center">
    <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="solarSystemCanvas">
    </canvas>
  </div>
</template>
