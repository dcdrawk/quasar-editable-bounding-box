<template>
  <div
    class="bounding-box absolute-top-left"
    :style="boundingBoxStyle"
  >
    <div class="bounding-box__inner full-width full-height relative-position">
      <div
        class="bounding-box__remove q-pa-sm flex justify-center items-center"
        :style="removeStyle"
        @click="$emit('remove')"
        @touchstart="$emit('remove')"
      >
        <q-icon :name="fasTimes" />
      </div>

      <div
        v-for="(point, index) in resizePoints"
        :key="index"
        class="bounding-box__resize"
        :class="[`bounding-box__resize--${point} `, resizeClass]"
        :style="[resizeStyle]"
        @mousedown.stop.prevent="$emit('resize-start', { event: $event, position: point })"
        @touchstart.stop.prevent="$emit('resize-start', { event: $event, position: point })"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { fasTimes } from '@quasar/extras/fontawesome-v5'
import { computed } from 'vue'
import { colors } from 'quasar'
import { IResizeEvent } from 'src/env'

defineEmits<{
  (e: 'remove'): void
  (e: 'resize-start', event: IResizeEvent): void
  (e: 'resize-end'): void
}>()

interface Props {
  width: number
  height: number
  x: number
  y: number
  edit?: boolean
  color?: string
  rounded?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  edit: false,
  color: 'green-6',
  rounded: false
})

const flipX = computed(() => props.width < 0)
const flipY = computed(() => props.height < 0)

/**
 * Bounding box dimensions
 */
const dimensions = computed(() => ({
  width: Math.abs(props.width),
  height: Math.abs(props.height),
  scaleX: flipX.value ? -1 : 1,
  scaleY: flipY.value ? -1 : 1
}))

/**
 * Bounding box style
 */
const boundingBoxStyle = computed(() => ({
  width: `${dimensions.value.width}px`,
  height: `${dimensions.value.height}px`,
  transform: `translate(${props.x}px, ${props.y}px) scaleX(${dimensions.value.scaleX}) scaleY(${dimensions.value.scaleY})`,
  outlineColor: colors.getPaletteColor(props.color)
}))

/**
 * Remove button style
 */
const removeStyle = computed(() => ({
  backgroundColor: colors.changeAlpha(colors.getPaletteColor(props.color), 0.2),
  color: colors.getPaletteColor(props.color),
  border: `2px solid ${colors.getPaletteColor(props.color)}`,
  left: flipX.value ? 'auto' : '-2px',
  right: flipX.value ? '-2px' : 'auto',
  top: flipY.value ? 'auto' : '-40px',
  bottom: flipY.value ? '-40px' : 'auto',
  borderBottomRightRadius: flipY.value ? '4px' : '0',
  borderBottomLeftRadius: flipY.value ? '4px' : '0',
  borderTopRightRadius: flipY.value ? '0' : '4px',
  borderTopLeftRadius: flipY.value ? '0' : '4px'
}))

/**
 * Resize styles
 * Ensures that the resize circles are visible while resizing
 */
const resizeClass = computed(() => ({
  'bounding-box__resize--visible': props.edit
}))

const resizeStyle = computed(() => ({
  backgroundColor: colors.getPaletteColor(props.color),
  borderRadius: props.rounded ? '50%' : '0'
}))

/**
 * Resize points
 */
const resizePoints = [
  'top-left',
  'top',
  'top-right',
  'right',
  'bottom-right',
  'bottom',
  'bottom-left',
  'left'
]
</script>

<style scoped lang="scss">

.bounding-box {
  outline: 2px solid $red-6;
  transform-origin: 0 0;

  &__remove {
    position: absolute;
    pointer-events: auto;
    cursor: pointer;
    width: 40px;
    height: 40px;
    transition: opacity 150ms ease;
  }

  &__resize {
    position: absolute;
    width: 10px;
    height: 10px;
    transition: opacity 150ms ease;
    cursor: pointer;

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

  &:hover {
    .bounding-box__remove,
    .bounding-box__resize {
      opacity: 1;
    }
  }
}
</style>
