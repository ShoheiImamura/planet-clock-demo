<script lang="ts">

export type Planet = {
  /** 日名 */
  japaneseName: string;
  /** 描画上の半径 */
  radius: number;
  /** 描画上の大きさ（半径） */
  size: number;
  /** 公転周期 */
  orbitalPeriod: number;
  /** 1年間で進む角度比 */
  angularVelocity: number;
  /** year 時間経過後の角度位置(ラジアン) */
  angle: (year: number) => number,
};

export type Moon = {
  /** 惑星 */
  planet: Planet;
  /** 描画上の半径 */
  radius: number;
  /** 公転周期 */
  orbitalPeriod: number;
  /** 1年間で進む角度比 */
  angularVelocity: number;
  /** year 時間経過後の角度位置(ラジアン) */
  angle: (year: number) => number,
};


export const planet = () => {
  /** 水星 */
  const Mercury: Planet = {
    radius: 39,
    japaneseName: "水星",
    size: 3,
    orbitalPeriod: 0.24,
    angularVelocity: Math.PI * 2 * (1 / 0.24),
    angle: (year: number) => { return 0 + Mercury.angularVelocity * year },
  };

  /** 金星 */
  const Venus: Planet = {
    radius: 72,
    japaneseName: "金星",
    size: 3,
    orbitalPeriod: 0.62,
    angularVelocity: Math.PI * 2 * (1 / 0.62),
    angle: (year: number) => { return 0 + Venus.angularVelocity * year },
  };

  /** 地球 */
  const Earth: Planet = {
    radius: 100,
    japaneseName: "地球",
    size: 4,
    orbitalPeriod: 1, // 
    angularVelocity: Math.PI * 2 * (1 / 1),
    angle: (year: number) => { return 0 + Earth.angularVelocity * year },
  };

  /** 火星 */
  const Mars: Planet = {
    radius: 152,
    japaneseName: "火星",
    size: 5,
    orbitalPeriod: 1.88,
    angularVelocity: Math.PI * 2 * (1 / 1.88),
    angle: (year: number) => { return 0 + Mars.angularVelocity * year },
  };

  /** 木星 */
  const Jupiter: Planet = {
    radius: 170,
    japaneseName: "木星",
    size: 6,
    orbitalPeriod: 11.8,
    angularVelocity: Math.PI * 2 * (1 / 11.8),
    angle: (year: number) => { return 0 + Jupiter.angularVelocity * year },
  };
  /** 土星 */
  const Saturn: Planet = {
    radius: 180,
    japaneseName: "土星",
    size: 5,
    orbitalPeriod: 29.4,
    angularVelocity: Math.PI * 2 * (1 / 29.4),
    angle: (year: number) => { return 0 + Saturn.angularVelocity * year },
  };
  /** 天王星 */
  const Uranus: Planet = {
    radius: 190,
    japaneseName: "天王星",
    size: 4,
    orbitalPeriod: 84.0,
    angularVelocity: Math.PI * 2 * (1 / 84.0),
    angle: (year: number) => { return 0 + Uranus.angularVelocity * year },
  };
  /** 海王星 */
  const Neptune: Planet = {
    radius: 200,
    japaneseName: "海王星",
    size: 3,
    orbitalPeriod: 164,
    angularVelocity: Math.PI * 2 * (1 / 164),
    angle: (year: number) => { return 0 + Neptune.angularVelocity * year },
  };

  /** 月 */
  const TheMoon: Moon = {
    planet: Earth,
    radius: 15,
    orbitalPeriod: 0.075,
    angularVelocity: Math.PI * 2 * (1 / 0.075),
    angle: (year: number) => { return 0 + TheMoon.angularVelocity * year },
  };
  return {
    // 惑星（Planet）
    Mercury,
    Venus,
    Earth,
    Mars,
    Jupiter,
    Saturn,
    Uranus,
    Neptune,
    // 衛星(Moon)
    TheMoon,
  };
};

export default {};
</script>