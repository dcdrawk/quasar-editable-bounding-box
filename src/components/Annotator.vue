<template>
  <div id="annotator">
    <div id="label-bar">
      <h4>Your boxes {{ pointerX }} {{ activeBox }}</h4>
      <ul>
        <li
          v-for="(box, i) in boxes"
          :key="i"
          :class="{'active': i===activeBoxIndex}"
        >
          <input
            v-model="box.label"
            @click="makeBoxActive(i)"
          >
          <a @click="removeBox(i)">x</a>
        </li>
      </ul>
    </div>
    <div
      id="image-wrapper"
      ref="wrapper"
      :style="{backgroundImage: `url(caterpillar.jpg)`}"
      @mousedown="startDrawingBox"
      @touchstart="startDrawingBox"
      @mousemove="changeBox"
      @touchmove="changeBox"
      @mouseup="stopDrawingBox"
      @touchend="stopDrawingBox"
    >
      <Box
        v-if="drawingBox.active"
        :b-width="drawingBox.width"
        :b-height="drawingBox.height"
        :b-top="drawingBox.top"
        :b-left="drawingBox.left"
      />
      <Box
        v-for="(box, i) in boxes"
        :key="i"
        :b-top="box.top"
        :b-left="box.left"
        :b-label="box.label"
        :b-width="box.width"
        :b-height="box.height"
        :b-active="i===activeBoxIndex"
        @select="makeBoxActive(i)"
        @remove="removeBox(i)"
        @resize="startResizing"
      />
    </div>
  </div>
</template>

<script>
import { computed, ref } from 'vue'
import { useMouse, useWindowScroll } from '@vueuse/core'
import Box from 'src/components/Box.vue'
import { pick } from 'lodash'

/**
 * Get Event Name
 * @desc return the name of the event, which depends on type and touch event
 * @param type 'move' | 'end'
 * @param event MouseEvent | TouchEvent
 */
function getEventName (type, event) {
  return type === 'move'
    ? event.touches ? 'touchmove' : 'mousemove'
    : event.touches ? 'touchend' : 'mouseup'
}

export default {
  name: 'AppAnnotator',

  components: { Box },

  setup () {
    const wrapper = ref(null)
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
      if (!wrapper.value) return { x: 0, y: 0 }

      const box = wrapper.value.getBoundingClientRect()

      return {
        x: pointerX.value - box.left - scrollX.value,
        y: pointerY.value - box.top - scrollY.value
      }
    })

    return {
      pointerX,
      pointerY,
      relativePointerCoordinates,
      wrapper
    }
  },

  data () {
    return {
      drawingBox: {
        active: false,
        top: 0,
        left: 0,
        height: 0,
        width: 0
      },
      wrapperWidth: 0,
      boxes: [],
      activeBoxIndex: null,
      isResizing: false,
      resizePosition: undefined
    }
  },

  computed: {
    activeBox () {
      return this.boxes[this.activeBoxIndex]
    },

    resizeDiffX () {
      return this.activeBox.left - this.relativePointerCoordinates.x
    },

    resizeDiffY () {
      return this.activeBox.top - this.relativePointerCoordinates.y
    }
  },

  mounted () {
    this.wrapperWidth = this.$refs.wrapper.offsetWidth

    window.addEventListener('resize', this.handleWindowResize)
  },

  beforeUnmount () {
    window.removeEventListener('resize', this.handleWindowResize)
  },

  methods: {
    startDrawingBox () {
      requestAnimationFrame(() => {
        this.drawingBox = {
          width: 0,
          height: 0,
          top: this.relativePointerCoordinates.y,
          left: this.relativePointerCoordinates.x,
          active: true
        }
      })
    },

    changeBox () {
      if (this.drawingBox.active) {
        requestAnimationFrame(() => {
          this.drawingBox = {
            ...this.drawingBox,
            width: this.relativePointerCoordinates.x - this.drawingBox.left,
            height: this.relativePointerCoordinates.y - this.drawingBox.top
          }
        })
      }
    },

    stopDrawingBox () {
      if (this.drawingBox.active) {
        if (this.drawingBox.width > 5) {
          this.boxes.push({ ...pick(this.drawingBox, ['width', 'height', 'top', 'left']) })
        }

        this.drawingBox = {
          active: false,
          top: 0,
          left: 0,
          height: 0,
          width: 0
        }
      }
    },

    makeBoxActive (i) {
      this.activeBoxIndex = i
    },

    removeBox (i) {
      this.boxes = this.boxes.filter((elem, index) => {
        return index !== i
      })
      this.activeBoxIndex = null
    },

    initResizeHandler () {
      this.wrapperWidth = this.$refs.wrapper.offsetWidth

      window.addEventListener('resize', this.handleResize)
    },

    handleWindowResize () {
      const wrapperWidth = this.$refs.wrapper.offsetWidth
      const scaleFactor = wrapperWidth / this.wrapperWidth

      if (scaleFactor === 1) return

      this.boxes.forEach((box) => {
        box.top = box.top * scaleFactor
        box.left = box.left * scaleFactor
        box.width = box.width * scaleFactor
        box.height = box.height * scaleFactor
      })

      this.wrapperWidth = wrapperWidth
    },

    /**
     * Resize Left
     * @desc set width and x values depending on pointer position
     */
    resizeLeft () {
      const maxLeft = this.activeBox.width + this.activeBox.left

      this.activeBox.width = Math.max(this.activeBox.width + this.resizeDiffX, 0)
      this.activeBox.left = Math.min(this.relativePointerCoordinates.x, maxLeft)
    },

    /**
     * Resize Top
     * @desc set height and y values depending on pointer position
     */
    resizeTop () {
      const maxTop = this.activeBox.height + this.activeBox.top

      this.activeBox.height = Math.max(this.activeBox.height + this.resizeDiffY, 0)
      this.activeBox.top = Math.min(this.relativePointerCoordinates.y, maxTop)
    },

    /**
     * Resize Right
     * @desc set width value depending on pointer position
     */
    resizeRight () {
      this.activeBox.width = Math.max(this.relativePointerCoordinates.x - this.activeBox.left, 0)
    },

    /**
     * Resize Bottom
     * @desc set width value depending on pointer position
     */
    resizeBottom () {
      this.activeBox.height = Math.max(this.relativePointerCoordinates.y - this.activeBox.top, 0)
    },

    /**
     * Start Resizing
     * @desc handle resizing depending on which resize position is selected
     */
    startResizing ({ event, position }) {
      this.isResizing = true
      this.resizePosition = position
      console.log('start...')

      const moveEvent = getEventName('move', event)
      const endEvent = getEventName('end', event)

      document.addEventListener(moveEvent, this.resizeBoundingBox)
      document.addEventListener(endEvent, this.finishBoundingBox)
    },

    /**
     * Resize Bounding Box
     * @desc handle resizing depending on which resize position is selected
     */
    resizeBoundingBox () {
      requestAnimationFrame(() => {
        switch (this.resizePosition) {
          case ('left'):
            this.resizeLeft()
            break

          case ('top-left'):
            this.resizeLeft()
            this.resizeTop()
            break

          case ('top'):
            this.resizeTop()
            break

          case ('top-right'):
            this.resizeRight()
            this.resizeTop()
            break

          case ('right'):
            this.resizeRight()
            break

          case ('bottom-right'):
            this.resizeRight()
            this.resizeBottom()
            break

          case ('bottom'):
            this.resizeBottom()
            break

          case ('bottom-left'):
            this.resizeLeft()
            this.resizeBottom()
            break
        }
      })
    },

    /**
     * Finish Bounding Box
     */
    finishBoundingBox () {
      // unset resizePosition
      if (this.resizePosition) {
        this.resizePosition = undefined
      }

      // unset isResizing
      if (this.drawingBox.active.value) {
        this.isResizing = false
      }

      // Clean up the event listeners
      document.removeEventListener('mousemove', this.drawBoundingBox)
      document.removeEventListener('mouseup', this.finishBoundingBox)
      document.removeEventListener('touchmove', this.drawBoundingBox)
      document.removeEventListener('touchend', this.finishBoundingBox)
    }
  }
}
</script>

<style lang="scss" scoped>
#annotator {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: relative;
  width: 100%;
  height: calc(100vh - 50px);

  #image-wrapper {
    max-height: 640px;
    max-width: 640px;
    width: 100%;
    height: 100%;
    background-repeat: no-repeat;
    background-size: contain;
    position: relative;
  }

  #label-bar {
    float: right;
    margin-right: 50px;
    width: 220px;

    ul {
      padding: 0;

      li {
        list-style-type: none;
        padding: 8px 16px;

        &.active {
          background-color: lightblue;
        }

        a {
          cursor: pointer;
          display: inline-block;
          margin-left: 4px;
          font-weight: bold;
          color: red;
        }
      }
    }
  }
}
</style>