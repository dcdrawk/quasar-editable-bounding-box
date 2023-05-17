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
      @click="removeMyself"
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
    />
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
    'remove'
  ],

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
</style>
