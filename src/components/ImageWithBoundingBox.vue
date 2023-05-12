<template>
  <div class="full-width q-pa-md">
    <h5 class="q-mb-none q-mt-sm">
      Image with Bounding Boxes {{ relativePointerCoordinates }}
    </h5>
    <div
      ref="control"
      v-touch-pan.prevent.capture.mouse="handlePan"
      class="relative-position overflow-hidden"
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

const boundingBoxDimensions = reactive({
  width: 0,
  height: 0
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

function handlePan ({ ...newInfo }) {
  if (newInfo.isFirst) {
    boundingBoxCoordinates.x = relativePointerCoordinates.value.x
    boundingBoxCoordinates.y = relativePointerCoordinates.value.y
    boundingBoxVisible.value = true
  } else if (newInfo.isFinal) {
    boundingBoxVisible.value = false
    boundingBoxDimensions.width = 0
    boundingBoxDimensions.height = 0
  } else {
    boundingBoxDimensions.width = newInfo.offset.x
    boundingBoxDimensions.height = newInfo.offset.y
  }
}
</script>

<style scoped lang="scss">
.bounding-image {
  user-select: none;
}
</style>
