<template>
  <div class="full-width q-pa-md">
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
        :x="boundingBoxInfo.x"
        :y="boundingBoxInfo.y"
        :width="boundingBoxInfo.width"
        :height="boundingBoxInfo.height"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
import BoundingBox from './BoundingBox.vue'
import { useMouse, useWindowScroll } from '@vueuse/core'

/**
 * Is the bounding box visible?
 */
const boundingBoxVisible = ref(false)

/**
 * Info used to draw new bounding boxes
 */
const boundingBoxInfo = reactive({
  x: 0,
  y: 0,
  width: 0,
  height: 0
})

/**
 * Control container template ref
 */
const control = ref()

/**
 * Pointer and Scroll Coordinates
 * See: https://vueuse.org/core/useMouse/
 */
const { x: pointerX, y: pointerY } = useMouse()
const { x: scrollX, y: scrollY } = useWindowScroll()
const scrollYStart = ref(0)
const scrollXStart = ref(0)
const scrollXDiff = computed(() => scrollX.value - scrollXStart.value)
const scrollYDiff = computed(() => scrollY.value - scrollYStart.value)

const tempBoxInfo = reactive({
  scrollX: 0,
  scrollY: 0,
  width: 0,
  height: 0,
  x: 0,
  y: 0
})

const scrolledLast = ref(false)

/**
 * Pointer coordinates relative to control element
 */
const relativePointerCoordinates = computed(() => {
  if (!control.value) return { x: 0, y: 0 }

  const box = control.value.getBoundingClientRect()

  return {
    x: pointerX.value - box.left - scrollX.value + scrollXDiff.value,
    y: pointerY.value - box.top - scrollY.value + scrollYDiff.value
  }
})

/**
 * Set Scroll Start
 * @desc set the scroll position when starting a bounding box
 * @param x: number
 * @param y: number
 */
function setScrollStart (x: number, y: number) {
  scrollXStart.value = x
  scrollYStart.value = y
}

/**
 * Handle Pan
 * @desc when the pan starts, set the x / y coordinates,
 * if the pan is final, reset the bounding box,
 * otherwise update with width / height of the box based on offset coordinates
 */
function handlePan ({ ...newInfo }) {
  if (scrolledLast.value) {
    scrollXStart.value = tempBoxInfo.scrollX
    scrollYStart.value = tempBoxInfo.scrollY
  }

  if (newInfo.isFirst) {
    setScrollStart(scrollX.value, scrollY.value)

    boundingBoxInfo.x = relativePointerCoordinates.value.x
    boundingBoxInfo.y = relativePointerCoordinates.value.y
    boundingBoxVisible.value = true
  } else if (newInfo.isFinal) {
    boundingBoxVisible.value = false
    boundingBoxInfo.width = 0
    boundingBoxInfo.height = 0
    scrolledLast.value = false
    setScrollStart(0, 0)
  } else {
    boundingBoxInfo.width = newInfo.offset.x + scrollXDiff.value
    boundingBoxInfo.height = newInfo.offset.y + scrollYDiff.value
  }
}
</script>

<style scoped lang="scss">
.bounding-image {
  user-select: none;
  width: 1200px;
}
</style>
