<template>
  <view
      class="k-button-group"
      :style="{
      paddingBottom: safeArea && `${bottomSafeArea || 10}px`,
      boxShadow: boxShadow ? '0 2px 12px 0 rgba(0, 0, 0, 0.1)' : ''
    }"
  >
    <button
        v-for="(button, index) in list"
        class="k-button"
        :style="{
        color: button.color,
        backgroundColor: button.background,
        border: '2rpx solid ' + (button.borderColor ?? 'transparent')
      }"
        @click="button.onClick"
        :key="index"
    >
      {{ button.label }}
    </button>
  </view>
</template>

<script>
import { computed } from "vue"
export default {
  options: {
    virtualHost: true
  },
  props: {
    safeArea: {
      type: Boolean,
      default: true
    },
    boxShadow: {
      type: Boolean,
      default: true
    },
    options: {
      type: [Array, Object],
      default: () => []
    },
    label: {
      type: String,
      default: ""
    },
    onClick: {
      type: Function,
      default: () => {}
    }
  },
  setup(props, { emit }) {
    const bottomSafeArea = uni.getSystemInfoSync().safeAreaInsets.bottom
    const list = computed(() => {
      if (JSON.stringify(props.options) !== "{}" && props.options.length > 0) {
        return Array.isArray(props.options) ? props.options : [props.options]
      } else {
        return [
          {
            label: props.label,
            color: "white",
            background: "#005CEEFF",
            onClick: props.onClick
          }
        ]
      }
    })
    return {
      bottomSafeArea,
      list
    }
  }
}
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
