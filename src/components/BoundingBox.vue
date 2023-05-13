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
      >
        <q-icon :name="fasTimes" />
      </div>

      <div
        v-for="(point, index) in resizePoints"
        :key="index"
        draggable="true"
        class="bounding-box__resize"
        :class="point.class"
        :style="point.style"
        @mousedown.prevent.stop="$emit('resize-start', point.position)"
        @mouseup.prevent.stop="$emit('resize-end')"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { fasTimes } from '@quasar/extras/fontawesome-v5'
import { computed } from 'vue'
import { colors } from 'quasar'

defineEmits<{
  (e: 'remove'): void
  (e: 'resize-start'): string
  (e: 'resize-end'): void
}>()

interface Props {
  width: number
  height: number
  x: number
  y: number
}

const props = withDefaults(defineProps<Props>(), {})

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
  transform: `translate(${props.x}px, ${props.y}px) scaleX(${dimensions.value.scaleX}) scaleY(${dimensions.value.scaleY})`
}))

/**
 * Remove button style
 */
const removeStyle = computed(() => ({
  backgroundColor: colors.changeAlpha(colors.getPaletteColor('red-6'), 0.2),
  color: colors.getPaletteColor('red-6'),
  border: `2px solid ${colors.getPaletteColor('red-6')}`,
  left: flipX.value ? '-2px' : 'auto',
  right: flipX.value ? 'auto' : '-2px',
  top: flipY.value ? 'auto' : '-40px',
  bottom: flipY.value ? '-40px' : 'auto',
  borderBottomRightRadius: flipY.value ? '4px' : '0',
  borderBottomLeftRadius: flipY.value ? '4px' : '0',
  borderTopRightRadius: flipY.value ? '0' : '4px',
  borderTopLeftRadius: flipY.value ? '0' : '4px'
}))

/**
 * Resize points
 */
const resizePoints = [{
  position: 'top-left',
  style: { top: '-10px', left: '-10px' }
}, {
  position: 'top',
  style: { top: '0px' },
  class: 'absolute-center'
}, {
  position: 'top-right',
  style: { top: '-10px', right: '-10px' }
}, {
  position: 'right',
  style: { right: '-20px', left: 'auto' },
  class: 'absolute-center'
}, {
  position: 'bottom-right',
  style: { right: '-10px', bottom: '-10px' }
}, {
  position: 'bottom',
  style: { bottom: '-20px', top: 'auto' },
  class: 'absolute-center'
}, {
  position: 'bottom-left',
  style: { left: '-10px', bottom: '-10px' }
}, {
  position: 'left',
  style: { left: '0', right: 'auto' },
  class: 'absolute-center'
}]
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
    opacity: 0;
    transition: opacity 150ms ease;
  }

  &__resize {
    position: absolute;
    cursor: grab;
    width: 20px;
    height: 20px;
    border-radius: 100%;
    background-color: $red-6;
    opacity: 0;
    transition: opacity 150ms ease;
  }

  &:hover {
    .bounding-box__remove,
    .bounding-box__resize {
      opacity: 1;
    }
  }
}
</style>
