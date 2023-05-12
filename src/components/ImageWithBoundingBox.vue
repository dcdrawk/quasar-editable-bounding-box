<template>
  <div class="full-width q-pa-md">
    <h5 class="q-mb-none q-mt-sm">
      Image with Bounding Boxes {{ relativePointerCoordinates }} {{ boundingBoxDimensions }}
    </h5>
    <div
      ref="control"
      class="relative-position"
      @mousedown="handleDragStart"
      @mouseup="handleDragEnd"
    >
      <q-img
        class="bounding-image"
        :draggable="false"
        src="/img/example-james-webb-photo.jpg"
      />
      <BoundingBox
        v-if="boundingBoxVisible"
        :x="boundingBoxCoordinates.x"
        :y="boundingBoxCoordinates.y"
        :width="boundingBoxDimensions.width"
        :height="boundingBoxDimensions.height"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
import BoundingBox from './BoundingBox.vue'
import { useMouse } from '@vueuse/core'

const control = ref()

const boundingBoxVisible = ref(false)

const boundingBoxCoordinates = reactive({
  x: 0,
  y: 0
})

const { x: pointerX, y: pointerY } = useMouse()

const relativePointerCoordinates = computed(() => {
  if (!control.value) return { x: 0, y: 0 }

  const box = control.value.getBoundingClientRect()

  return {
    x: pointerX.value - box.left,
    y: pointerY.value - box.top
  }
})

const boundingBoxDimensions = computed(() => {
  return {
    width: relativePointerCoordinates.value.x - boundingBoxCoordinates.x,
    height: relativePointerCoordinates.value.y - boundingBoxCoordinates.y,
    flipX: relativePointerCoordinates.value.x - boundingBoxCoordinates.x < 0
  }
})

function handleDragStart () {
  boundingBoxCoordinates.x = relativePointerCoordinates.value.x
  boundingBoxCoordinates.y = relativePointerCoordinates.value.y
  boundingBoxVisible.value = true
}

function handleDragEnd () {
  boundingBoxVisible.value = false
}
</script>

<style scoped lang="scss">
.bounding-image {
  user-select: none;
}
</style>
