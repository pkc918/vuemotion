<script setup lang="ts">
import { ref, watch } from 'vue'
import type { FigureOptions, Growable } from '@vue-motion/lib'
import { Path, figure } from '@vue-motion/lib'
import { defineWidget } from '@vue-motion/core'
// import type { MathFunction } from './math-function'

export interface MathFunctionOptions extends FigureOptions, Growable {
  fn: (x: number) => number
  ranges: [number, number]
  domain: [number, number]
  divisionX?: number
  divisionY?: number
  color?: string
  width?: number
}

const props = defineProps<MathFunctionOptions>()
const options = defineWidget(props)

function generateSvgPath(mathFunc: (x: number) => number, ranges: {
  x: [number, number]
  y: [number, number]
}, scaleX: number, scaleY: number) {
  const { x: [xMin, xMax], y: [yMax] } = ranges
  const path = []
  const step = (xMax - xMin) / 100 // 分为 100 份

  for (let x = xMin; x <= xMax; x += step) {
    const y = mathFunc(x)
    // 将数学函数的值转换为SVG坐标
    const svgX = (x - xMin) * scaleX
    const svgY = (yMax - y) * scaleY // 反转 y 轴以符合 SVG 坐标系统
    path.push(`${x === xMin ? 'M' : 'L'} ${svgX} ${svgY}`)
  }

  return path.join(' ')
}

const path = ref('')

watch(props, () => {
  path.value = generateSvgPath(options.fn, {
    x: options.ranges,
    y: options.domain ?? [Infinity, Infinity],
  }, options.divisionX ?? 100, options.divisionY ?? 100)
}, {
  immediate: true,
})
</script>

<template>
  <g v-bind="figure(options)">
    <Path :d="path" />
  </g>
</template>
