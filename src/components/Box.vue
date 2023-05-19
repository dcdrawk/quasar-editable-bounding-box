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
      @click="removeBox"
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
      @mousedown="selectBox"
    >
      <div
        v-for="(point, index) in resizePoints"
        :key="index"
        class="resize-handle"
        :class="[`resize-handle--${point} `, resizeClass]"
        @mousedown.stop.prevent="$emit('resize', { event: $event, position: point })"
        @touchstart.stop.prevent="$emit('resize', { event: $event, position: point })"
      />
    </div>
  </div>
</template>

<script>
import { colors } from 'quasar'

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
    selectBox () {
      this.$emit('select')
    },

    removeBox () {
      this.$emit('remove')
    }
  }
}
</script>

<style lang="scss" scoped>
.box {
  position: absolute;
  border: 2px #90ee90 solid;
  &:hover, &.active {
    background-color: rgba(144, 238, 144, .2);
  }
  z-index: 3;
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

  &--visible {
    opacity: 1;
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
