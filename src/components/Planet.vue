<script lang="ts">
export type Planet = {
  radius: number,
  angularVelocity: number,
}
export const planet = () => {
  // 水星
  const Mercury: Planet = {
    radius: 60,
    angularVelocity: (Math.PI * (6 / 24)),
  };

  // 金星
  const Venus: Planet = {
    radius: 80,
    angularVelocity: (Math.PI * (4 / 24)),
  };

  // 地球
  const Earth: Planet = {
    radius: 120,
    angularVelocity: (Math.PI * (2 / 24)),
  };

  // 木星
  const Jupiter: Planet = {
    radius: 150,
    angularVelocity: (Math.PI * (1 / 24)),
  };

  // 土星
  const Saturn: Planet = {
    radius: 180,
    angularVelocity: (Math.PI * (1 / 24)),
  };
  return {
    Mercury, Venus, Earth, Jupiter, Saturn
  }
}
export default {

}
</script>