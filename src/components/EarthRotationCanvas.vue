
<script lang="ts" setup>
import { ref, onMounted, watch, toRefs } from "vue";
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
  drawBackGround(canvasScale.value); // 全体背景
  drawEarth(Earth, props.dayCount, canvasScale.value / 2);
  drawSun(Earth, props.dayCount, 2, 'rgba(255,0,0,1)');
  drawLineSunToEarth(Earth, props.dayCount, 'rgba(255,0,0,0.8)', 2);
  drawPlanetSurface(canvasScale.value / 2, 'rgba(255,255,255,0.8)');
  drawPlaceLine(3, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(255,255,255,0.8)');
  drawPlace(6, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(255,255,255,0.8)');
  drawPlace(3, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(0,0,0,1)');
  drawMoon();
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
  drawFilledCircle(radius, "rgb(0,0,0)");
}

// 地球を描画
const drawEarth = (earth: typeof Earth, days: number, radius: number) => {
  const angle = (-1) * earth.angle(dayToYear(days));
  const halfPI = Math.PI * 0.5;
  drawFilledCircle(radius, "rgb(255,255,255,0)", (angle - halfPI), (angle + halfPI));
  drawFilledCircle(radius, "rgba(255,255,255,0.375)", (angle + halfPI), (angle + halfPI * 3));
};

// 地点を描画
const drawPlace = (radius: number, earth: typeof Earth, days: number, time: number, fillColor: string = 'rgba(255,0,0,1)') => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24));// 場所の相対角度
  const { x, y } = getXYByRadians(
    placeAngle - earthAngle,
    canvasScale.value / 2
  );
  ctx.value.arc(
    centerCoordinate().x + x,
    centerCoordinate().y + y,
    radius,
    0,
    Math.PI * 2
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
  ctx.value.closePath();
};


// 地点までの線
const drawPlaceLine = (radius: number, earth: typeof Earth, days: number, time: number, lineColor: string = 'rgba(255,0,0,1)', lineWidth: number = 3) => {
  if (ctx.value === null) return;
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24));// 場所の相対角度
  const { x, y } = getXYByRadians(
    placeAngle - earthAngle,
    canvasScale.value / 2
  );
  ctx.value.strokeStyle = lineColor;
  ctx.value.lineWidth = lineWidth;
  ctx.value.beginPath();
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.lineTo(centerCoordinate().x + x, centerCoordinate().y + y);
  ctx.value.stroke();
  ctx.value.closePath();
};


// 恒星を描画
const drawSun = (
  earth: typeof Earth,
  days: number = 0, // 基準日からの日数
  StarRadius: number = 3,
  strokeColor: string = "rgba(255,255,255,0.9)",
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    earth.angle(dayToYear(days)) + Math.PI, // 地球の方向と逆になる
    1.9 * canvasScale.value / 2
  );
  ctx.value.arc(
    centerCoordinate().x + x,
    centerCoordinate().y + y,
    StarRadius,
    0,
    Math.PI * 2
  );

  ctx.value.strokeStyle = strokeColor;
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

// 地球の枠
const drawPlanetSurface = (
  radius: number,
  lineColor: string = "rgba(255,255,255,0.6)", // 軌道の色
  lineWidth: number = 3
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.strokeStyle = lineColor;
  ctx.value.lineWidth = lineWidth;
  ctx.value.arc(centerCoordinate().x, centerCoordinate().y, radius, 0, 2 * Math.PI);
  ctx.value.stroke();
  ctx.value.closePath();
};

// 0 時ライン
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
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    canvasScale.value / 2,
    (-1) * (sunAngle + Math.PI),
    (-1) * (sunAngle + Math.PI),
  )
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
}


// 月の位置
const moonCoordinate = ref({
  x: 50,
  y: 50,
})

const setMoonCoordinate = (
  moon: Moon,
  days: number = 0,
  radius: number = canvasScale.value * 4 / 5
) => {

  // 位置を導出
  const { x, y } = getXYByRadians(
    moon.angle(dayToYear(days)),
    radius
  );
  moonCoordinate.value = {
    x: centerCoordinate().x + x, y: centerCoordinate().y + y
  }
}
// 月の描画
const drawMoon = (distance: number = canvasScale.value * 4 / 5) => {
  const moon = TheMoon;
  const year = dayToYear(props.dayCount);
  const radius = canvasScale.value * 2 / 30;
  const angle = moon.angle(year) - moon.planet.angle(year)

  setMoonCoordinate(moon, props.dayCount, distance)

  drawCircle(radius);
  if (Math.sin(angle) >= 0 && Math.cos(angle) >= 0) {
    // 満月 -> 下弦
    drawSemicirlce(radius, Math.PI * 1 / 2);
    drawEllipse(radius, radius * Math.cos(angle));
  } else if (Math.sin(angle) >= 0 && Math.cos(angle) < 0) {
    // 下弦 -> 新月
    drawSemicirlce(radius, Math.PI * 1 / 2);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), '#212121');
  } else if (Math.sin(angle) < 0 && Math.cos(angle) < 0) {
    // 新月 -> 上弦
    drawSemicirlce(radius, Math.PI * 3 / 2);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), '#212121');
  } else {
    // 上弦 -> 満月
    drawSemicirlce(radius, Math.PI * 3 / 2);
    drawEllipse(radius, radius * Math.cos(angle),);
  }
}
// そのままの月
const drawCircle = (
  radius: number,
  fillColor: string = "#212121",
  startAngle: number = 0,
  endAngle: number = Math.PI * 2
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(moonCoordinate.value.x, moonCoordinate.value.y);
  ctx.value.arc(
    moonCoordinate.value.x,
    moonCoordinate.value.y,
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
  fillColor: string = "rgba(255,255,255,1)",
  startAngle: number = 0,
  endAngle: number = Math.PI * 1
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(moonCoordinate.value.x, moonCoordinate.value.y);
  ctx.value.arc(
    moonCoordinate.value.x,
    moonCoordinate.value.y,
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
  ctx.value.moveTo(moonCoordinate.value.x, moonCoordinate.value.y);
  ctx.value.ellipse(
    moonCoordinate.value.x,
    moonCoordinate.value.y,
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

</style>

<template>
  <div class="d-flex justify-center">
    <canvas :width="canvasScale * 2" :height="canvasScale * 2" class="canvas" id="earthRotationCanvas">
    </canvas>
  </div>
</template>
