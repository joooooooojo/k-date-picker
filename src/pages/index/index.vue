<script>
import KDatePicker from "../../package/components/k-date-picker/KDatePicker.vue";
export default {
  components: {
    KDatePicker,
  },
  data() {
    return {
      title: "Hello",
      show: false,
      defaultValue: ["2020", "2028"],
    };
  },
  onLoad() {},
  methods: {
    onShow() {
      this.show = true;
    },
    onChangeValue() {
      this.defaultValue = ["2021", "2022"];
    },
  },
};
</script>

<template>
  <view class="content">
    <image class="logo" src="/static/logo.png"></image>
    <view class="text-area">
      <text class="title">{{ title }}</text>
      <button @click="onShow">点我</button>
      <button @click="onChangeValue">点我</button>
    </view>
    <KDatePicker
      v-model="show"
      type="yearrange"
      :default-value="defaultValue"
      limit-start-date="2020"
      limit-end-date="2028"
    ></KDatePicker>
  </view>
</template>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 200rpx;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 50rpx;
}

.text-area {
  display: flex;
  justify-content: center;
}

.title {
  font-size: 36rpx;
  color: #8f8f94;
}
</style>
