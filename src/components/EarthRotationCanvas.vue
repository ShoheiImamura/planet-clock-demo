
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

// 全体の角度調整 右側を太陽とする
const canvasRotate = ref(Math.PI * 2 / 2);

const drawEarthRotation = () => {
  if (ctx.value === null) return;
  ctx.value.clearRect(0, 0, canvasScale.value * 2, canvasScale.value * 2);
  drawBackGround(canvasScale.value); // 全体背景
  drawEarth(Earth, props.dayCount, canvasScale.value / 2, 'rgba(255,255,255,0.5)', 'rgba(0,0,0,1)', canvasRotate.value);
  drawPlanetSurface(canvasScale.value / 2, 'rgba(255,255,255,1)');
  // drawLineSunToEarth('rgba(255,50,50,1)', 3, canvasScale.value * 19 / 20);
  drawSun(Earth, props.dayCount, 3, 'rgba(255,50,50,1)', 4, canvasRotate.value);
  drawPlaceLine(3, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(255,255,255,1)', 5, canvasRotate.value);
  drawPlace(6, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(255,255,255,1)', canvasRotate.value);
  drawPlace(3, Earth, props.dayCount, props.dayUnixTimeCount, 'rgba(0,0,0,1)', canvasRotate.value);
  drawMoonLine(canvasScale.value * 3 / 4, 2, 'rgba(255,255,255,1)', canvasRotate.value);
  drawMoon(canvasScale.value * 3 / 4, 10, 'rgba(255,255,255,1)', 'rgba(25,25,25,1)', canvasRotate.value);
  drawSecondHand(props.dayUnixTimeCount);
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
const drawEarth = (earth: typeof Earth, days: number, radius: number,
  lightColor: string = "rgba(175,175,175,1)",
  blackColor: string = "rgb(0,0,0,1)",
  canvasRotate: number = 0,
) => {
  const angle = (-1) * earth.angle(dayToYear(days));
  drawFilledCircle(radius, blackColor, - (Math.PI) / 2 + canvasRotate, (Math.PI) / 2 + canvasRotate);
  drawFilledCircle(radius, lightColor, (Math.PI) / 2 + canvasRotate, - (Math.PI) / 2 + canvasRotate);
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 春分~秋分
  if (Math.sin(angle) < 0) {
    ctx.value.ellipse(
      centerCoordinate().x,
      centerCoordinate().y,
      canvasScale.value / 2,
      (-1) * canvasScale.value / 2 * Math.sin(angle) * 23.4 / 90, // 地軸のズレ分ずらす
      - Math.PI / 2 + canvasRotate,
      // 0,
      0,
      Math.PI
    );
    ctx.value.fillStyle = lightColor;
  } else {
    ctx.value.ellipse(
      centerCoordinate().x,
      centerCoordinate().y,
      canvasScale.value / 2,
      canvasScale.value / 2 * Math.sin(angle) * 23.4 / 90,
      - Math.PI / 2 + canvasRotate,
      // 0,
      Math.PI,
      0
    );
    ctx.value.fillStyle = blackColor;
  }
  ctx.value.fill();
  ctx.value.closePath();
};

// 地点を描画
const drawPlace = (radius: number, earth: typeof Earth, days: number, time: number,
  fillColor: string = 'rgba(255,0,0,1)', canvasRotate: number = 0) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24)) + canvasRotate;// 場所の相対角度
  const { x, y } = getXYByRadians(
    placeAngle, // 右位置に調整
    canvasScale.value / 2 * Math.cos(Math.PI * 35 / 180), // 緯度調整
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
const drawPlaceLine = (radius: number, earth: typeof Earth,
  days: number, time: number, lineColor: string = 'rgba(255,0,0,1)', lineWidth: number = 3,
  canvasRotate: number = 0) => {
  if (ctx.value === null) return;
  // 位置を導出
  const earthAngle = (-1) * earth.angle(dayToYear(days)) // 地球の角度
  const placeAngle = Math.PI * 2 * (time / (60 * 60 * 24)) + canvasRotate;// 場所の相対角度
  const { x, y } = getXYByRadians(
    placeAngle, // 右の位置に調整
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
  strokeColor: string = "rgba(255,255,255,)",
  lineWidth: number = 5,
  canvasRotate: number = 0,
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  // 位置を導出
  const { x, y } = getXYByRadians(
    Math.PI + canvasRotate, // 左に調整
    1.9 * canvasScale.value / 2
  );
  ctx.value.arc(
    centerCoordinate().x + x,
    centerCoordinate().y + y,
    StarRadius,
    0,
    Math.PI * 2
  );

  ctx.value.lineWidth = lineWidth;
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

// 12 時ライン
const drawLineSunToEarth = (

  lineColor: string = "rgba(255,255,255, 0.9)", // 線の色
  lineWidth: number = 2, // 線の幅
  lineLength: number = canvasScale.value * 9 / 10,
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    lineLength,
    Math.PI,
    Math.PI,
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
  radius: number = canvasScale.value * 0.9,
  canvasRotate: number = 0
) => {

  // 位置を導出
  const { x, y } = getXYByRadians(
    moon.relativeAngle(dayToYear(days)) + canvasRotate,
    radius
  );
  moonCoordinate.value = {
    x: centerCoordinate().x + x, y: centerCoordinate().y + y
  }
}
// 中心から月までの線
const drawMoonLine = (distance: number = canvasScale.value * 0.9,
  lineWidth: number = 2, lineColor: string = 'rgba(255,255,255,1)',
  canvasRotate: number = 0) => {
  const moon = TheMoon;
  const year = dayToYear(props.dayCount);
  const radius = canvasScale.value * 2 / 30;
  const angle = moon.relativeAngle(year)

  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    distance,
    - angle + canvasRotate,
    - angle + canvasRotate,
    false,
  )
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
}
// 月の描画
const drawMoon = (
  distance: number = canvasScale.value * 0.9,
  radius: number = canvasScale.value * 2 / 30,
  moonColor: string = 'rgba(255,255,200,1)',
  moonShadowColor: string = 'rgba(40,40,40,1)',
  canvasRotate: number = 0,
) => {
  const moon = TheMoon;
  const year = dayToYear(props.dayCount);
  const angle = moon.relativeAngle(year) + canvasRotate
  setMoonCoordinate(moon, props.dayCount, distance, canvasRotate)
  drawCircle(radius, moonShadowColor);
  if (Math.sin(angle) >= 0 && Math.cos(angle) >= 0) {
    // 新月 -> 上弦
    drawSemicirlce(radius, - angle + canvasRotate, 0, Math.PI, moonColor, canvasRotate);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), moonShadowColor, -angle);
  } else if (Math.sin(angle) >= 0 && Math.cos(angle) < 0) {
    // 上弦 -> 満月
    drawSemicirlce(radius, - angle + canvasRotate, 0, Math.PI, moonColor, canvasRotate);
    drawEllipse(radius, - radius * Math.cos(angle), moonColor, -angle);
  } else if (Math.sin(angle) < 0 && Math.cos(angle) < 0) {
    // 満月 -> 下弦
    drawSemicirlce(radius, - angle + canvasRotate, Math.PI, 0, moonColor, canvasRotate);
    drawEllipse(radius, - radius * Math.cos(angle), moonColor, -angle);
  } else {
    // 下弦 -> 新月
    drawSemicirlce(radius, - angle + canvasRotate, Math.PI, 0, moonColor, canvasRotate);
    drawEllipse(radius, radius * Math.abs(Math.cos(angle)), moonShadowColor, -angle);
  }
}
// そのままの月
const drawCircle = (
  radius: number,
  fillColor: string = 'rgba(75,75,75,1)',
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
  startAngle: number = 0,
  endAngle: number = Math.PI * 1,
  fillColor: string = "rgba(255,255,255,1)",
  canvasRotate: number = 0,
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(moonCoordinate.value.x, moonCoordinate.value.y);
  ctx.value.arc(
    moonCoordinate.value.x,
    moonCoordinate.value.y,
    radius,
    rotation + startAngle + canvasRotate,
    rotation + endAngle + canvasRotate
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
};
const drawEllipse = (
  radiusX: number, radiusY: number,
  fillColor: string = "rgba(255,255,255,1)",
  rotation: number = Math.PI / 2,
  startAngle: number = 0,
  endAngle: number = Math.PI * 2,
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.moveTo(moonCoordinate.value.x, moonCoordinate.value.y);
  ctx.value.ellipse(
    moonCoordinate.value.x,
    moonCoordinate.value.y,
    radiusX,
    radiusY,
    rotation,
    // 0,
    startAngle,
    endAngle
  );
  ctx.value.fillStyle = fillColor;
  ctx.value.fill();
}
// 秒針
const drawSecondHand = (
  second: number,
  lineColor: string = "rgba(255,255,255, 0.9)", // 線の色
  lineWidth: number = 2, // 線の幅
  lineLength: number = canvasScale.value * 9 / 10,
) => {
  if (ctx.value === null) return;
  ctx.value.beginPath();
  ctx.value.lineWidth = lineWidth;
  ctx.value.strokeStyle = lineColor;
  ctx.value.moveTo(centerCoordinate().x, centerCoordinate().y);
  ctx.value.arc(
    centerCoordinate().x,
    centerCoordinate().y,
    lineLength,
    secondToAngle(second),
    secondToAngle(second),
  )
  ctx.value.stroke();
  ctx.value.closePath();
  ctx.value.lineWidth = 1; // reset
}
// 
const secondToAngle = (second: number): number => {
  return - second * Math.PI * 2 / 60;
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
