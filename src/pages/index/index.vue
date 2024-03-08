<script setup>
import KDatePicker from "../../package/components/k-date-picker/KDatePicker.vue";
import { ref, reactive } from "vue";
const prop = reactive({
  start: false,
});
const defaultValue = ref(["2021", "2022"]);
const cp = ["2020", "2028"];
function onShow() {
  prop.start = true;
}
function onChangeValue() {
  defaultValue.value = ["2021", "2022"];
}
</script>

<template>
  <view class="content">
    <view class="text-area">
      <button @click="onShow">打开选择器</button>
      <button @click="onChangeValue">更改起始值</button>
    </view>
    <KDatePicker
      v-model="prop.start"
      type="date"
      is-range
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
