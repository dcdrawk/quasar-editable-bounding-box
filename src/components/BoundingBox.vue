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

const boundingBoxStyle = computed(() => ({
  width: `${props.width}px`,
  height: `${props.height}px`,
  transform: `translate(${props.x}px, ${props.y}px)`
}))
</script>

<style scoped>
.bounding-box {
  border: 2px solid red;
  pointer-events: none;
}
</style>
