<template>
  <div id="annotator">
    <div id="label-bar">
      <h4>Your boxes {{ pointerX }} {{ relativePointerCoordinates }}</h4>
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
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
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
      />
    </div>
  </div>
</template>

<script>
import { computed, ref } from 'vue'
import { useMouse, useWindowScroll } from '@vueuse/core'
import Box from 'src/components/Box.vue'
import { pick } from 'lodash'

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
      activeBoxIndex: null
    }
  },

  mounted () {
    this.wrapperWidth = this.$refs.wrapper.offsetWidth

    window.addEventListener('resize', this.handleResize)
  },

  beforeUnmount () {
    window.removeEventListener('resize', this.handleResize)
  },

  methods: {
    startDrawingBox (e) {
      this.drawingBox = {
        width: 0,
        height: 0,
        top: this.relativePointerCoordinates.y,
        left: this.relativePointerCoordinates.x,
        active: true
      }
    },

    changeBox (e) {
      if (this.drawingBox.active) {
        this.drawingBox = {
          ...this.drawingBox,
          width: this.relativePointerCoordinates.x - this.drawingBox.left,
          height: this.relativePointerCoordinates.y - this.drawingBox.top
        }
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

    handleResize () {
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
