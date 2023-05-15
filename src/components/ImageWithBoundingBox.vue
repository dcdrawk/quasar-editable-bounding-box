<template>
  <section class="q-pa-md">
    <div class="flex row wrap justify-center">
      <!-- Settings -->
      <div class="col col-xs-12">
        <h6 class="q-my-sm">
          Bounding Box Settings
        </h6>
        <div class="image-box__settings flex row q-col-gutter-sm">
          <q-select
            v-model="boxSettingsColor"
            class="q-mb-sm col-6"
            filled
            :options="colorOptions"
            emit-value
            map-options
            label="Color"
          >
            <template #prepend>
              <q-icon
                name="circle"
                :color="boxSettingsColor"
              />
            </template>

            <template #option="scope">
              <q-item v-bind="scope.itemProps">
                <q-item-section avatar>
                  <q-icon
                    name="circle"
                    :color="scope.opt.value"
                  />
                </q-item-section>
                <q-item-section>
                  <q-item-label>{{ scope.opt.label }}</q-item-label>
                </q-item-section>
              </q-item>
            </template>
          </q-select>

          <q-select
            v-model="boxSettingsPointStyle"
            class="q-mb-sm col-6"
            filled
            :options="pointOptions"
            label="Point Style"
          />
        </div>
      </div>

      <!-- Instructions -->
      <ul class="col">
        <li>
          Click and drag to create a bounding box.
        </li>
        <li>
          Drag the control points to resize.
        </li>
        <li>
          Only one box can exist at a time.
        </li>
        <li>
          Remove a box by clicking the 'X'.
        </li>
      </ul>
    </div>

    <!-- Control -->
    <div
      ref="control"
      class="image-box__control relative-position overflow-hidden"
      @mousedown.prevent="createBoundingBox"
      @touchstart.prevent="createBoundingBox"
    >
      <q-img
        class="image-box__image"
        :draggable="false"
        src="/img/example-james-webb-photo.jpg"
      />

      <BoundingBox
        v-if="boundingBoxVisible"
        key="new"
        :x="boundingBoxInfo.x"
        :y="boundingBoxInfo.y"
        :width="boundingBoxInfo.width"
        :height="boundingBoxInfo.height"
        :color="boxSettingsColor"
        :rounded="boxHasRoundedPoints"
        :edit="!!resizePosition"
        @resize-start="startResizing"
        @remove="boundingBoxVisible = false"
      />
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
import BoundingBox from './BoundingBox.vue'
import { useMouse, useWindowScroll } from '@vueuse/core'
import { IResizeEvent } from 'src/env'

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
 * Bounding Box Color settings
 */
const boxSettingsColor = ref('red-6')
const colorOptions = [
  { label: 'Red', value: 'red-6' },
  { label: 'Purple', value: 'purple-4' },
  { label: 'Blue', value: 'blue-6' },
  { label: 'Cyan', value: 'cyan-6' },
  { label: 'Green', value: 'green-6' },
  { label: 'Yellow', value: 'yellow-6' },
  { label: 'Orange', value: 'orange-6' },
  { label: 'Grey', value: 'grey-6' }
]

/**
 * Control point style
 */
const boxSettingsPointStyle = ref('Square')
const pointOptions = ['Square', 'Round']
const boxHasRoundedPoints = computed(() => boxSettingsPointStyle.value === 'Round')

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
const startX = ref(0)
const startY = ref(0)
const endX = ref(0)
const endY = ref(0)

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

/**
 * Pointer coordinates relative to control element
 */
const relativePointerCoordinates = computed(() => {
  if (!control.value) return { x: 0, y: 0 }

  const box = control.value.getBoundingClientRect()

  return {
    x: pointerX.value - box.left - scrollX.value,
    y: pointerY.value - box.top - scrollY.value
  }
})

/**
 * Get Event Name
 * @desc return the name of the event, which depends on type and touch event
 * @param type 'move' | 'end'
 * @param event MouseEvent | TouchEvent
 */
function getEventName (type: 'move' | 'end', event: MouseEvent | TouchEvent) {
  return type === 'move'
    ? (event as TouchEvent).touches ? 'touchmove' : 'mousemove'
    : (event as TouchEvent).touches ? 'touchend' : 'mouseup'
}

/**
 * Create Bounding Box
 * @desc create the bounding box, set the starting values,
 * add event listeners depending on touch / mouse
 * @param e: MouseEvent | TouchEvent
 */
function createBoundingBox (event: MouseEvent | TouchEvent) {
  if (boundingBoxVisible.value) return

  const moveEvent = getEventName('move', event)
  const endEvent = getEventName('end', event)

  // Requesting an animation frame fixes janky initial "jump" on touchscreens
  requestAnimationFrame(() => {
    startX.value = relativePointerCoordinates.value.x
    startY.value = relativePointerCoordinates.value.y

    document.addEventListener(moveEvent, drawBoundingBox)
    document.addEventListener(endEvent, finishBoundingBox)
  })
}

/**
 * Draw Bounding Box
 * @desc make the box visible, set the end values which will continue to update until finished
 */
function drawBoundingBox () {
  if (!boundingBoxVisible.value) {
    boundingBoxVisible.value = true
  }

  endX.value = relativePointerCoordinates.value.x
  endY.value = relativePointerCoordinates.value.y

  updateBoundingBox()
}

/**
 * Update Bounding Box
 * @desc set bounding box values based on start / end values
 */
function updateBoundingBox () {
  boundingBoxInfo.x = Math.min(startX.value, endX.value)
  boundingBoxInfo.y = Math.min(startY.value, endY.value)
  boundingBoxInfo.width = Math.abs(endX.value - startX.value)
  boundingBoxInfo.height = Math.abs(endY.value - startY.value)
}

/**
 * Use to track which resize point is selected
 */
const resizePosition = ref<string | null>(null)

/**
 * Finish Bounding Box
 */
function finishBoundingBox () {
  // unset resizePosition
  if (resizePosition.value) {
    resizePosition.value = null
  }

  // unset isResizing
  if (boundingBoxVisible.value) {
    isResizing.value = false
  }

  // Clean up the event listeners
  document.removeEventListener('mousemove', drawBoundingBox)
  document.removeEventListener('mouseup', finishBoundingBox)
  document.removeEventListener('touchmove', drawBoundingBox)
}

/**
 * Resize refs / computed
 */
const isResizing = ref(false)
const resizeDiffX = computed(() => boundingBoxInfo.x - relativePointerCoordinates.value.x)
const resizeDiffY = computed(() => boundingBoxInfo.y - relativePointerCoordinates.value.y)

/**
 * Resize Left
 * @desc set width and x values depending on pointer position
 */
function resizeLeft () {
  boundingBoxInfo.width = boundingBoxInfo.width + resizeDiffX.value
  boundingBoxInfo.x = relativePointerCoordinates.value.x
}

/**
 * Resize Top
 * @desc set height and y values depending on pointer position
 */
function resizeTop () {
  boundingBoxInfo.height = boundingBoxInfo.height + resizeDiffY.value
  boundingBoxInfo.y = relativePointerCoordinates.value.y
}

/**
 * Resize Right
 * @desc set width value depending on pointer position
 */
function resizeRight () {
  boundingBoxInfo.width = relativePointerCoordinates.value.x - boundingBoxInfo.x
}

/**
 * Resize Bottom
 * @desc set width value depending on pointer position
 */
function resizeBottom () {
  boundingBoxInfo.height = relativePointerCoordinates.value.y - boundingBoxInfo.y
}

/**
 * Resize Bounding Box
 * @desc handle resizing depending on which resize position is selected
 */
async function resizeBoundingBox () {
  requestAnimationFrame(() => {
    switch (resizePosition.value) {
      case ('left'):
        resizeLeft()
        break

      case ('top-left'):
        resizeLeft()
        resizeTop()
        break

      case ('top'):
        resizeTop()
        break

      case ('top-right'):
        resizeRight()
        resizeTop()
        break

      case ('right'):
        resizeRight()
        break

      case ('bottom-right'):
        resizeRight()
        resizeBottom()
        break

      case ('bottom'):
        resizeBottom()
        break

      case ('bottom-left'):
        resizeLeft()
        resizeBottom()
        break
    }
  })
}

/**
 * Start Resizing
 * @desc handle resizing depending on which resize position is selected
 */
function startResizing ({ event, position }: IResizeEvent) {
  isResizing.value = true
  resizePosition.value = position

  const moveEvent = getEventName('move', event)
  const endEvent = getEventName('end', event)

  document.addEventListener(moveEvent, resizeBoundingBox)
  document.addEventListener(endEvent, finishBoundingBox)
}
</script>

<style scoped lang="scss">
.image-box {
  &__settings {
    @media (min-width: $breakpoint-sm-min) {
      max-width: 360px;
    }
  }

  &__control {
    max-width: 1000px;
  }

  &__image {
    user-select: none;
  }
}
</style>
