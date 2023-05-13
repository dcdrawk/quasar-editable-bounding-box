<template>
  <div class="full-width q-pa-md">
    <div
      ref="control"
      v-touch-pan.prevent.capture.mouse="handlePan"
      class="relative-position overflow-hidden"
      @mousemove="handleMouseMove"
      @mouseup="handleResizeEnd"
    >
      <q-img
        class="bounding-image full-width"
        :draggable="false"
        src="/img/example-james-webb-photo.jpg"
      />

      <!-- New Bounding Box -->
      <BoundingBox
        v-if="boundingBoxVisible"
        :x="boundingBoxInfo.x"
        :y="boundingBoxInfo.y"
        :width="boundingBoxInfo.width"
        :height="boundingBoxInfo.height"
      />

      <!-- List of Bounding boxes-->
      <BoundingBox
        v-for="(box, index) in boundingBoxes"
        :key="`${index}${box.x}{box.y}`"
        :x="box.x"
        :y="box.y"
        :width="box.width"
        :height="box.height"
        :edit="box.isEditing"
        @remove="removeBoundingBox(index)"
        @resize-start="handleResize($event, box)"
        @resize-end="handleResizeEnd"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, toValue } from 'vue'
import BoundingBox from './BoundingBox.vue'
import { useMouse, useWindowScroll } from '@vueuse/core'

interface IBoundingBox {
  x: number
  y: number
  width: number
  height: number
  isEditing: boolean
}
/**
 * Is the bounding box visible?
 */
const boundingBoxVisible = ref(false)

/**
 * Info used to draw new bounding boxes
 */
const boundingBoxInfo = reactive<IBoundingBox>({
  x: 0,
  y: 0,
  width: 0,
  height: 0,
  isEditing: false
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
  if (newInfo.isFirst) {
    setScrollStart(scrollX.value, scrollY.value)

    boundingBoxInfo.x = relativePointerCoordinates.value.x
    boundingBoxInfo.y = relativePointerCoordinates.value.y
    boundingBoxVisible.value = true
  } else if (newInfo.isFinal) {
    pushBoundingBox(boundingBoxInfo)
    boundingBoxVisible.value = false
    boundingBoxInfo.width = 0
    boundingBoxInfo.height = 0
    setScrollStart(0, 0)
  } else {
    boundingBoxInfo.width = newInfo.offset.x + scrollXDiff.value
    boundingBoxInfo.height = newInfo.offset.y + scrollYDiff.value
  }
}

const boundingBoxes = ref<IBoundingBox[]>([])

function pushBoundingBox (box: IBoundingBox) {
  boundingBoxes.value.push({ ...box })
}

function removeBoundingBox (index: number) {
  boundingBoxes.value.splice(index, 1)
}

const isResizing = ref(false)
const resizePosition = ref('')
const resizeBox = ref()
const resizeDiffX = computed(() => resizeBox.value.x - relativePointerCoordinates.value.x)
const resizeDiffY = computed(() => resizeBox.value.y - relativePointerCoordinates.value.y)

function handleResize (position: unknown, box: IBoundingBox) {
  isResizing.value = true
  resizePosition.value = position as string
  resizeBox.value = box
  resizeBox.value.isEditing = true
}

function handleResizeEnd () {
  isResizing.value = false
  resizeBox.value.isEditing = false
}

function resizeLeft () {
  resizeBox.value.width = resizeBox.value.width + resizeDiffX.value
  resizeBox.value.x = relativePointerCoordinates.value.x
}

function resizeTop () {
  resizeBox.value.height = resizeBox.value.height + resizeDiffY.value
  resizeBox.value.y = relativePointerCoordinates.value.y
}

function resizeRight () {
  resizeBox.value.width = relativePointerCoordinates.value.x - resizeBox.value.x
}

function resizeBottom () {
  resizeBox.value.height = relativePointerCoordinates.value.y - resizeBox.value.y
}

function handleMouseMove () {
  if (!isResizing.value) return

  switch (resizePosition.value) {
    case ('left'):
      resizeLeft()
      break

    case ('top-left'):
      resizeTop()
      resizeLeft()
      break

    case ('top'):
      resizeTop()
      break

    case ('top-right'):
      resizeTop()
      resizeRight()
      break

    case ('right'):
      resizeRight()
      break

    case ('bottom-right'):
      resizeBottom()
      resizeRight()
      break

    case ('bottom'):
      resizeBottom()
      break

    case ('bottom-left'):
      resizeBottom()
      resizeLeft()
      break
  }
}
</script>

<style scoped lang="scss">
.bounding-image {
  user-select: none;
  width: 1200px;
}
</style>
