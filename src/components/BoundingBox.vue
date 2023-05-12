<template>
  <div
    class="bounding-box absolute-top-left"
    :style="boundingBoxStyle"
  />
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  width: number
  height: number
  x: number
  y: number
}

const props = withDefaults(defineProps<Props>(), {})

const dimensions = computed(() => ({
  width: Math.abs(props.width),
  height: Math.abs(props.height),
  scaleX: props.width > 0 ? 1 : -1,
  scaleY: props.height > 0 ? 1 : -1
}))

const boundingBoxStyle = computed(() => ({
  width: `${dimensions.value.width}px`,
  height: `${dimensions.value.height}px`,
  transform: `translate(${props.x}px, ${props.y}px) scaleX(${dimensions.value.scaleX}) scaleY(${dimensions.value.scaleY})`
}))
</script>

<style scoped>
.bounding-box {
  border: 2px solid red;
  pointer-events: none;
  transform-origin: 0 0;
}
</style>
