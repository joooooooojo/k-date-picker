<script>
export default {
  name: "KBottomPopup",
  options: {
    virtualHost: true,
  },
  // #ifndef VUE3
  model: {
    prop: "modelValue",
    event: "update:modelValue",
  },
  // #endif
  props: {
    modelValue: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      show: false,
      stopPropagation: false,
    };
  },
  methods: {
    maskClick() {
      if (this.stopPropagation) {
        this.stopPropagation = false;
        return;
      }
      this.$emit("update:modelValue", false);
    },
    /** 阻止事件冒泡 */
    onTouchStart(e) {
      this.stopPropagation = true;
    },
  },
  mounted() {},
};
</script>

<template>
  <view
    :style="{
      opacity: modelValue ? 1 : 0,
      zIndex: modelValue ? 10086 : -10086,
    }"
    class="k-bottom-popup"
    @click="maskClick"
  >
    <view
      :class="{
        fade: !modelValue,
        'fade-out': modelValue,
      }"
      class="tran-content"
      @touchstart="onTouchStart"
    >
      <slot></slot>
    </view>
  </view>
</template>

<style lang="scss">
.k-bottom-popup {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: -10086;
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
  .fade {
    transform: translateY(100%);
  }
  .fade-out {
    transform: translateY(0);
  }
  .tran-content {
    position: absolute;
    right: 0;
    left: 0;
    bottom: 0;
    transition: transform 0.3s ease-in-out;
  }
}
</style>
