<script lang="ts">
import type { Planet } from "./Planet.vue";

/** 惑星会合 */
export const planetaryConjunction = () => {
  /** 内合 */
  const isInnerConjunction = (planet1: Planet, planet2: Planet, year: number) => {
    const radian1 = planet1.angle(year);
    const radian2 = planet2.angle(year);
    const subRadian = Math.abs(radian1 - radian2) % (2 * Math.PI);
    const subDegree = subRadian * (180 / Math.PI);
    return subDegree < 1;
  }
  /** 外合 */
  const isOuterConjunction = (planet1: Planet, planet2: Planet, year: number) => {
    const radian1 = planet1.angle(year);
    const radian2 = planet2.angle(year);
    const subRadian = Math.abs(radian1 - radian2) % (2 * Math.PI);
    const subDegree = subRadian * (180 / Math.PI);
    return Math.abs(subDegree - 180) < 1;
  }
  return { isInnerConjunction, isOuterConjunction };
}

export type planetaryConjunctionType = {
  planet1: Planet, planet2: Planet, conjunctionType: 'inner' | 'outer'
}

export default {};
</script>