<template>
  <view
    class="k-button-group"
    :style="{
      paddingBottom: safeArea && `${bottomSafeArea || 10}px`,
      boxShadow: boxShadow ? '0 2px 12px 0 rgba(0, 0, 0, 0.1)' : '',
    }"
  >
    <button
      v-for="(button, index) in list"
      class="k-button"
      :style="{
        color: button.color,
        backgroundColor: button.background,
        border: `2rpx solid ${button.borderColor || 'transparent'}`,
      }"
      @click="onClick(index)"
      :key="index"
    >
      {{ button.label }}
    </button>
  </view>
</template>

<script>
export default {
  options: {
    virtualHost: true,
  },
  props: {
    safeArea: {
      type: Boolean,
      default: true,
    },
    boxShadow: {
      type: Boolean,
      default: true,
    },
    options: {
      type: [Array, Object],
      default: () => [],
    },
    label: {
      type: String,
      default: "",
    }
  },
  computed: {
    list() {
      if (JSON.stringify(this.options) !== "{}" && this.options.length > 0) {
        return Array.isArray(this.options) ? this.options : [this.options];
      } else {
        return [
          {
            label: this.label,
            color: "white",
            background: "#005CEEFF",
          },
        ];
      }
    },
  },
  data() {
    return {
      bottomSafeArea: uni.getSystemInfoSync().safeAreaInsets.bottom,
    };
  },
  methods: {
    onClick(index) {
      this.$emit("onClick", index);
    },
  },
};
</script>
<style lang="scss">
.k-button-group {
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
  background-color: white;
  width: 100%;
  padding: 20rpx 30rpx;
  column-gap: 24rpx;
  .k-button {
    width: 100%;
    font-size: 32rpx;
  }
}
</style>
