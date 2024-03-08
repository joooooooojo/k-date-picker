<template>
  <uni-transition :show="isShow" mode-class="fade" :styles="styles">
    <view class="k-toast__content" :class="'k-type-' + config.type">
      <uni-icons :type="iconName" size="17" :color="iconColor"></uni-icons>
      <text
        class="k-toast__content__text"
        :class="['k-toast__content__text--' + config.type]"
        style="max-width: 400rpx"
      >
        {{ config.message }}
      </text>
    </view>
  </uni-transition>
</template>

<script>
import { reactive } from "vue";

export default {
  data() {
    return {
      isShow: false,
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
    };
  },
  methods: {
    show(params) {
      this.config.type = params.type;
      switch (params.type) {
        case "error":
          this.iconName = "close";
          this.iconColor = "#f56c6c";
          this.config.message = params.message;
          break;
        case "success":
          this.iconName = "checkbox";
          this.iconColor = "#5ac725";
          this.config.message = params.message;
          break;
        case "warning":
          this.iconName = "info";
          this.iconColor = "#f9ae3d";
          this.config.message = params.message;
          break;
      }
      this.isShow = true;
      this.styles.zIndex = "10086";
      setTimeout(() => {
        this.isShow = false;
        setTimeout(() => {
          this.styles.zIndex = "-10086";
        }, 300);
        //结束动画时 调用事件
        try {
          if (params.event) {
            params.event();
          }
        } catch (e) {}
      }, 2000);
    },

    error(message) {
      this.config.type = "error";
      this.iconName = "close";
      this.iconColor = "#f56c6c";
      this.config.message = message;
      this.isShow = true;
      this.styles.zIndex = "10086";
      setTimeout(() => {
        this.isShow = false;
        setTimeout(() => {
          this.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    },

    success(message) {
      this.config.type = "success";
      this.iconName = "checkbox";
      this.iconColor = "#5ac725";
      this.config.message = message;
      this.isShow = true;
      this.styles.zIndex = "10086";
      setTimeout(() => {
        this.isShow = false;
        setTimeout(() => {
          this.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    },

    warning(message) {
      this.config.type = "warning";
      this.iconName = "info";
      this.iconColor = "#f9ae3d";
      this.config.message = message;
      this.isShow = true;
      this.styles.zIndex = "10086";
      setTimeout(() => {
        this.isShow = false;
        setTimeout(() => {
          this.styles.zIndex = "-10086";
        }, 300);
      }, 2000);
    },
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
