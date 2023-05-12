<template>
  <div class="full-width q-pa-md">
    <h5 class="q-mb-none q-mt-sm">
      Image with Bounding Boxes
    </h5>
    <div class="relative-position">
      <q-img
        class="bounding-image"
        :draggable="false"
        src="/img/example-james-webb-photo.jpg"
      />
      <BoundingBox
        :width="200"
        :height="200"
        :x="0"
        :y="0"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import BoundingBox from './BoundingBox.vue'
</script>

<style scoped lang="scss">
.bounding-image {
  user-select: none;
}
</style>
