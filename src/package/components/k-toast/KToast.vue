<template>
  <uni-transition :show="r.show" mode-class="fade" :styles="r.styles">
    <view class="k-toast__content" :class="'k-type-' + r.config.type">
      <uni-icons :type="r.iconName" size="17" :color="r.iconColor"></uni-icons>
      <text
        class="k-toast__content__text"
        :class="['k-toast__content__text--' + r.config.type]"
        style="max-width: 400rpx"
      >
        {{ r.config.message }}
      </text>
    </view>
  </uni-transition>
</template>

<script>
import { reactive } from "vue";

export default {
  setup(props, { expose }) {
    const r = reactive({
      show: false,
      styles: {
        position: "absolute",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        top: "0",
        left: "0",
        zIndex: "-10086",
        width: "100%",
        height: "100%",
        textAlign: "center",
        backgroundColor: "transparent",
      },
      iconName: "",
      iconColor: "",
      config: {
        type: "",
        message: "",
      },
    });
    function show(params) {
      r.config.type = params.type;
      switch (params.type) {
        case "error":
          r.iconName = "close";
          r.iconColor = "#f56c6c";
          r.config.message = params.message;
          break;
        case "success":
          r.iconName = "checkbox";
          r.iconColor = "#5ac725";
          r.config.message = params.message;
          break;
        case "warning":
          r.iconName = "info";
          r.iconColor = "#f9ae3d";
          r.config.message = params.message;
          break;
      }
      r.show = true;
      r.styles.zIndex = "10086";
      setTimeout(() => {
        r.show = false;
        setTimeout(() => {
          r.styles.zIndex = "-10086";
        }, 300);
        //结束动画时 调用事件
        try {
          if (params.event) {
            params.event();
          }
        } catch (e) {}
      }, 2000);
    }

    function error(message) {
      r.config.type = "error";
      r.iconName = "close";
      r.iconColor = "#f56c6c";
      r.config.message = message;
      r.show = true;
      r.styles.zIndex = "10086";
      setTimeout(() => {
        r.show = false;
        setTimeout(() => {
          r.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    }

    function success(message) {
      r.config.type = "success";
      r.iconName = "checkbox";
      r.iconColor = "#5ac725";
      r.config.message = message;
      r.show = true;
      r.styles.zIndex = "10086";
      setTimeout(() => {
        r.show = false;
        setTimeout(() => {
          r.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    }

    function warning(message) {
      r.config.type = "warning";
      r.iconName = "info";
      r.iconColor = "#f9ae3d";
      r.config.message = message;
      r.show = true;
      r.styles.zIndex = "10086";
      setTimeout(() => {
        r.show = false;
        setTimeout(() => {
          r.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    }
    expose({ show, success, warning, error });
    return {
      r,
    };
  },
};
</script>
<style lang="scss" scoped>
$k-success: #5ac725;
$k-error: #f56c6c;
$k-warning: #f9ae3d;

$k-toast-k-type-success-color: $k-success !default;
$k-toast-k-type-success-background-color: #dbf1e1 !default;
$k-toast-k-type-success-border-color: #bef5c8 !default;
$k-toast-k-type-success-border-width: 1px !default;

$k-toast-k-type-error-color: $k-error !default;
$k-toast-k-type-error-background-color: #fef0f0 !default;
$k-toast-k-type-error-border-color: #fde2e2 !default;
$k-toast-k-type-error-border-width: 1px !default;

$k-toast-k-type-warning-color: $k-warning !default;
$k-toast-k-type-warning-background-color: #fdf6ec !default;
$k-toast-k-type-warning-border-color: #faecd8 !default;
$k-toast-k-type-warning-border-width: 1px !default;

$k-toast-k-type-default-color: #fff !default;
$k-toast-k-type-default-background-color: #585858 !default;

.text {
  font-size: 14px;
  color: #fff;
}
.k-toast__content {
  display: flex;
  flex-direction: row;
  justify-content: center;
  padding: 12px 20px;
  border-radius: 4px;
  color: #fff;
  align-items: center;
  max-width: 600rpx;
  position: relative;
  &__text {
    color: #fff;
    font-size: 15px;
    line-height: 15px;
    margin-left: 5px;

    &--default {
      color: #fff;
    }

    &--error {
      color: $k-error;
    }

    &--success {
      color: $k-success;
    }

    &--warning {
      color: $k-warning;
    }
  }
}
.k-type-success {
  color: $k-toast-k-type-success-color;
  background-color: $k-toast-k-type-success-background-color;
  border-color: $k-toast-k-type-success-border-color;
  border-width: 1px;
}

.k-type-error {
  color: $k-toast-k-type-error-color;
  background-color: $k-toast-k-type-error-background-color;
  border-color: $k-toast-k-type-error-border-color;
  border-width: $k-toast-k-type-error-border-width;
}

.k-type-warning {
  color: $k-toast-k-type-warning-color;
  background-color: $k-toast-k-type-warning-background-color;
  border-color: $k-toast-k-type-warning-border-color;
  border-width: 1px;
}

.k-type-default {
  color: $k-toast-k-type-default-color;
  background-color: $k-toast-k-type-default-background-color;
}
</style>
