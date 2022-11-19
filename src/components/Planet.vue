<script lang="ts">
export type Planet = {
  radius: number; // 半径（便宜的に長半径を指定）
  orbitalPeriod: number; // 公転周期
  angularVelocity: number;
};
export type Moon = {
  planet: Planet; // 惑星
  radius: number; // 半径（便宜的に長半径を指定）
  orbitalPeriod: number; // 公転周期
  angularVelocity: number;
};

export const planet = () => {
  // 水星
  const Mercury: Planet = {
    radius: 39,
    orbitalPeriod: 0.24,
    angularVelocity: Math.PI * 2 * (1 / 0.24),
  };

  // 金星
  const Venus: Planet = {
    radius: 72,
    orbitalPeriod: 0.62,
    angularVelocity: Math.PI * 2 * (1 / 0.62),
  };

  // 地球
  const Earth: Planet = {
    radius: 100,
    orbitalPeriod: 1,
    angularVelocity: Math.PI * 2 * (1 / 1),
  };

  // 火星
  const Mars: Planet = {
    radius: 152,
    orbitalPeriod: 1.88,
    angularVelocity: Math.PI * 2 * (1 / 1.88),
  };

  // 木星
  const Jupiter: Planet = {
    radius: 170,
    orbitalPeriod: 11.8,
    angularVelocity: Math.PI * 2 * (1 / 11.8),
  };
  // 土星
  const Saturn: Planet = {
    radius: 180,
    orbitalPeriod: 29.4,
    angularVelocity: Math.PI * 2 * (1 / 29.4),
  };
  // 天王星
  const Uranus: Planet = {
    radius: 190,
    orbitalPeriod: 84.0,
    angularVelocity: Math.PI * 2 * (1 / 84.0),
  };
  // 海王星
  const Neptune: Planet = {
    radius: 200,
    orbitalPeriod: 164,
    angularVelocity: Math.PI * 2 * (1 / 164),
  };

  // 月
  const TheMoon: Moon = {
    planet: Earth,
    radius: 15,
    orbitalPeriod: 0.075,
    angularVelocity: Math.PI * 2 * (1 / 0.075),
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