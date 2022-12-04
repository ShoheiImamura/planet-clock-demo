
<script lang="ts" setup>
import { ref, onMounted, watch, toRefs } from "vue";
import { planet } from "./Planet.vue";
import { Planet, Moon, } from "./Planet.vue";
import { planetaryConjunction } from "./PlanetaryConjunction.vue";
import type { planetaryConjunctionType } from "./PlanetaryConjunction.vue";

const props = defineProps({
  dayCount: {
    type: Number,
    default: 0,
  },
  divWidth: {
    type: Number,
    default: 0
  }
});

type Coordinate = {
  x: number;
  y: number;
};

const ctx = ref<CanvasRenderingContext2D | null>(null);
const canvasScale = ref(150);

const centerCoordinate = (): Coordinate => {
  return {
    x: canvasScale.value,
    y: canvasScale.value,
  }
};
const { divWidth } = toRefs(props)
watch(divWidth, () => {
  console.log(divWidth.value)
  if (divWidth.value < 300) {
    canvasScale.value = divWidth.value / 2
  } else {
    canvasScale.value = 150
  }
})

// 惑星
const { Mercury, Venus, Earth, Mars, Jupiter, Saturn, TheMoon } = planet();
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
const planetList = [
  // Uranus,
  // Saturn,
  Jupiter,
  Mars,
  Earth,
  Venus,
  Mercury
];

// 惑星会合
const eventPlanetaryConjunction = (planetList: Planet[], year: number) => {
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

const drawPlanetaryConjunction = (planetList: Planet[], days: number) => {
  const planetaryConjunctionList = eventPlanetaryConjunction(planetList, dayToYear(days))
  if (planetaryConjunctionList.length !== 0) {
    // 惑星会合ありの場合
    planetList.forEach(planet => {
      // list に planet 自身が含まれるかどうか
      const isConjanction = planetaryConjunctionList.some((planetaryConjunction) => {
        return planet == planetaryConjunction.planet1 || planet == planetaryConjunction.planet2
      });
      if (isConjanction) {
        drawLineStarToPlanet(planet, days, 'rgba(100,255,255,0.5)', 2);
        drawPlanet(planet, days, 4, 'rgba(100,255,255,0.5)');
      }
    });
  }
}

const drawSolarSystem = () => {
  if (ctx.value === null) return;

  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);
  // 全体背景
  drawBackGround(canvasScale.value, 'black');

  // 惑星図形
  drawPlanetsView([Mercury, Venus, Earth, Mars, Jupiter, Saturn], props.dayCount, 'rgba(128,128,128,0.5)');
  // 地球の背景
  drawBackGround(Earth.radiusRatio * canvasScale.value / 2);
  // 春分線
  drawVernalEquinox();


  // 惑星
  // 土星
  drawPlanet(Saturn, props.dayCount, Saturn.size);
  // 木星
  drawPlanet(Jupiter, props.dayCount, Jupiter.size);
  // 火星
  drawPlanet(Mars, props.dayCount, Mars.size);
  // 地球
  drawPlanetaryOrbit(Earth, 'rgba(100,100,100,0.5)');
  drawLineStarToPlanet(Earth, props.dayCount, 'rgba(255,255,255,1)', 2);
  drawPlanet(Earth, props.dayCount, Earth.size);
  // 金星
  drawPlanet(Venus, props.dayCount, Venus.size);
  // 水星
  drawPlanet(Mercury, props.dayCount, Mercury.size);

  // 太陽（恒星）

  // 月（衛星）
  drawMoon(TheMoon, props.dayCount);

  // 惑星会合
  drawPlanetaryConjunction([Mercury, Venus, Earth, Mars, Jupiter, Saturn], props.dayCount);

};

// 背景
const drawBackGround = (radius: number, fillColor = 'rgba(0,0,0, 0.5)') => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    radius,
    0,
    Math.PI * 2
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();

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
  const { x, y } = planetRectangularCoordinate(planet, days,);
  ctx.value.arc(x, y, planetRadius, 0, Math.PI * 2);

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
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
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
  lineColor: string = "rgba(100,100,100,0.5)" // 軌道の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    planet.radiusRatio * canvasScale.value / 2,
    0,
    2 * Math.PI
  );
  ctx.value.lineWidth = 3;
  ctx.value.stroke();
  ctx.value.closePath();
};

// 恒星から惑星へのライン
const drawLineStarToPlanet = (
  planet: Planet,
  days: number = 0,
  lineColor: string = "rgba(255,255,255, 0.5)", // 扇形の色
  lineWidth: number = 2 // 扇形の色
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;

  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
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
  const { x, y } = planetRectangularCoordinate(moon.planet, days);
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.arc(x, y, moon.radiusRatio * canvasScale.value / 2, 0, 2 * Math.PI, true);
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
  // 惑星の描画位置を導出
  const { x: planetX, y: planetY } = planetRectangularCoordinate(moon.planet, days);
  // 衛星の相対位置を導出
  const { x, y } = getXYByRadians(
    moon.angle(dayToYear(days)),
    moon.radiusRatio * canvasScale.value / 2
  );
  ctx.value.beginPath();
  ctx.value.arc(planetX + x, planetY + y, 2, 0, Math.PI * 2);
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};

// 惑星の直交座標
const planetRectangularCoordinate = (
  planet: Planet,
  days: number,
  center: Coordinate = { x: canvasScale.value, y: canvasScale.value }
): Coordinate => {
  // 位置を導出
  const { x, y } = getXYByRadians(
    planet.angle(dayToYear(days)),
    planet.radiusRatio * canvasScale.value / 2
  );
  return { x: center.x + x, y: center.y + y }
}

// 惑星の位置同士を結ぶ図形
const drawPlanetsView = (planets: Planet[], days: number, fillColor: string = 'rgba(128,128,128,0.8)') => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  for (let i = 0; i < planets.length; i++) {
    const { x, y } = planetRectangularCoordinate(planets[i], days);
    ctx.value.lineTo(x, y);
  }
  ctx.value.closePath();
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
}

const drawVernalEquinox = () => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = 2;
  ctx.value.strokeStyle = 'rgba(100,100,100,1)';

  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.lineTo(centerCoordinate().x + Earth.radiusRatio * canvasScale.value / 2, centerCoordinate().y)
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
}

</script>

<style scoped>

</style>

<template>
  <div class="d-flex justify-center">
    <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="solarSystemCanvas">
    </canvas>
  </div>
</template>
