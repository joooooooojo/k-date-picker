<script>
export default {
  name: "KBottomPopup",
  options: {
    virtualHost: true,
  },
  // #ifndef VUE3
  model: {
    prop: "value",
    event: "input",
  },
  // #endif
  props: {
    // #ifndef VUE3
    value: {
      type: Boolean,
      default: false,
    },
    // #endif
    // #ifdef VUE3
    modelValue: {
      type: Boolean,
      default: false,
    },
    // #endif
  },
  data() {
    return {
      stopPropagation: false,
      zIndex: -10086,
    };
  },
  computed: {
    show() {
      // #ifdef VUE3
      return this.modelValue;
      // #endif
      // #ifndef VUE3
      return this.value;
      // #endif
    },
  },
  methods: {
    maskClick() {
      if (this.stopPropagation) {
        this.stopPropagation = false;
        return;
      }
      // #ifndef VUE3
      this.$emit("input", false);
      // #endif
      // #ifdef VUE3
      this.$emit("update:modelValue", false);
      // #endif
    },
    /** 阻止事件冒泡 */
    onTouchStart(e) {
      this.stopPropagation = true;
    },
  },
  watch: {
    show: {
      handler(n) {
        if (n) {
          this.zIndex = 10086;
        } else {
          setTimeout(() => {
            this.zIndex = -10086;
          }, 300);
        }
      },
    },
  },
  mounted() {},
};
</script>

<template>
  <view
    :style="[
      {
        opacity: show ? 1 : 0,
        zIndex: zIndex,
      },
    ]"
    class="k-bottom-popup"
    @click="maskClick"
  >
    <view
      :class="{
        fade: !show,
        'fade-out': show,
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
