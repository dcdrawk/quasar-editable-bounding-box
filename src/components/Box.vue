<template>
  <div class="box-wrapper">
    <div
      v-if="bLabel"
      class="label"
      :style="{
        top: (bTop - 10) + 'px',
        left: bLeft + 'px',
        width: (bWidth + 4) + 'px'}"
    >
      {{ bLabel }}
    </div>
    <a
      v-if="bActive"
      class="box-delete"
      :style="{
        top: (bTop - 18) +'px',
        left: (bLeft + bWidth) + 'px'
      }"
      @click="$emit('remove')"
    >
      x
    </a>
    <div
      class="box"
      :style="{
        top: bTop + 'px',
        left: bLeft + 'px',
        width: bWidth + 'px',
        height: bHeight + 'px'
      }"
      :class="{'active': bActive}"
      @touchstart.stop.prevent="$emit('select')"
    >
      <!-- @mousedown.stop.prevent="$emit('select')" -->
      <div
        v-for="(point, index) in resizePoints"
        :key="index"
        class="resize-handle"
        :class="[`resize-handle--${point} `, resizeClass]"
        @mousedown.stop.prevent="handleResize($event, point)"
        @touchstart.stop.prevent="handleResize($event, point)"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'AppBox',

  props: {
    bTop: {
      type: Number,
      default: 0
    },
    bLeft: {
      type: Number,
      default: 0
    },
    bWidth: {
      type: Number,
      default: 0
    },
    bHeight: {
      type: Number,
      default: 0
    },
    bLabel: {
      type: String,
      default: ''
    },
    bActive: {
      type: Boolean,
      default: false
    }
  },

  emits: [
    'select',
    'remove',
    'resize'
  ],

  data () {
    return {
      /**
       * Resize points
       */
      resizePoints: [
        'top-left',
        'top',
        'top-right',
        'right',
        'bottom-right',
        'bottom',
        'bottom-left',
        'left'
      ]
    }
  },

  computed: {
    /**
     * Resize styles
     * Ensures that the resize circles are visible while resizing
     */
    resizeClass () {
      return {
        'resize-handle--visible': this.bActive
      }
    }
  },

  methods: {
    handleResize ($event, position) {
      if (!this.bActive) return

      this.$emit('resize', { event: $event, position })
    }
  }
}
</script>

<style lang="scss" scoped>
.box {
  position: absolute;
  border: 2px #90ee90 solid;
  cursor: pointer;
  z-index: 3;

  &:hover, &.active {
    background-color: rgba(144, 238, 144, .2);
  }
}

.label {
  position: absolute;
  height: 22px;
  font-size: 16px;
  color: #000;
  font-weight: bold;
  background-color: #90ee90;
  z-index: 4;
}

.box-delete {
  position: absolute;
  z-index: 6;
  font-weight: bold;
  color: red;
  cursor: pointer;
  font-size: 24px;
  font-weight: bold;
}

.resize-handle {
  position: absolute;
  width: 10px;
  height: 10px;
  transition: opacity 150ms ease;
  cursor: pointer;
  background-color: #90ee90;
  opacity: 0;
  pointer-events: none;

  &--visible {
    opacity: 1;
    pointer-events: auto;
  }

  $transformDistance: -5px;

  &--top-left {
    top: $transformDistance;
    left: $transformDistance;
  }

  &--top {
    top: $transformDistance;
    left: 50%;
    transform: translateX(-50%);
  }

  &--top-right {
    top: $transformDistance;
    right: $transformDistance;
  }

  &--right {
    top: 50%;
    right: $transformDistance;
    transform: translateY(-50%);
  }

  &--bottom-right {
    bottom: $transformDistance;
    right: $transformDistance;
  }

  &--bottom {
    bottom: $transformDistance;
    left: 50%;
    transform: translateX(-50%);
  }

  &--bottom-left {
    bottom: $transformDistance;
    left: $transformDistance;
  }

  &--left {
    top: 50%;
    left: $transformDistance;
    transform: translateY(-50%);
  }
}
</style>
