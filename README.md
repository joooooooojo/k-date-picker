## 本插件有多个依赖项，使用前需要先行安装，后续更新会逐步去除依赖

- ~~[uni-popup 弹出层 ](https://ext.dcloud.net.cn/plugin?id=329)~~
- ~~[uni-icons 图标](https://ext.dcloud.net.cn/plugin?id=28)~~
- ~~dayjs~~

## 使用方法

```vue
<template>
  <view>
    <button @click="open">打开选择器</button>
    <KDatePicker v-model="show" type="day" is-range @change="handleChange" formatter="YYYY-MM-DD" />
  </view>
</template>

<script setup>
  import KDatePicker from '../../components/k-date-picker/KDatePicker.vue'
  const show = ref(false)
  const date = ref('')
  const open = () => {
    show.value = true
  }
  const handleChange = (value) => {
    date.value = value
  }
</script>
```

## 注意 ！

###### 本插件原使用Vue3编写，经过语法降级后。作者测试Vue2也可以正常使用，但是您可能会遇到一些兼容性问题，您可以尝试自行解决，如无能为力也可以提在评论区。

### 属性

| 属性名       | 类型                   | 默认值   | 可选值                       | 说明                                                   |
| ------------ | ---------------------- |-------|---------------------------|------------------------------------------------------|
| modelValue | Boolean                | -     |                           | 控制选择器显示                                              |
| confirmText  | String                 | 确认    |                           | 底部按钮文字                                               |
| title        | String                 | 请选择   |                           | 选择器标题                                                |
| defaultValue | String\|Number\|Arrary | -     | 凡是可被dayjs初始化的值都可传入        | 选择器初始化时默认值（不允许传入Number类型的年份 如：2020 改为 '2020'）        |
| type         | String                 | -     | "day" \| "month" \| "year" | 选择器类型                                                |
| isRange | Boolean | false |                           | 选择器是否为范围选择器                                          |
| limitStartDate | Number \| String | -     | 凡是可被dayjs初始化的值都可传入        | 限制选择器起始日期（不允许传入Number类型的年份 如：2020 改为 '2020'）         |
| limitEndDate | Number \| String | -     | 凡是可被dayjs初始化的值都可传入        | 限制选择器结束日期（不允许传入Number类型的年份 如：2020 改为 '2020'）         |
| multiple | Boolean | false |                           | 多选（目前暂不支持多选时间段。）                                     |
| formatter | String | -     |                           | change返回的数据默认为时间戳格式，通过此项配置可以格式化返回的数据。例如 'YYYY-MM-DD' |

### 事件

| 事件名  | 类型                                    | 说明             |
| ------- | --------------------------------------- | ---------------- |
| @change | (value: number\| [number,number])=>void | 选择器确认时触发 |

### CSS变量

###### 控制插件主题请直接修改CSS变量。

```scss
  /** 控制提示文字的位置 **/
  --bottom-offset: 10%;
  /** 控制提示文字的大小 **/
  --bottom-font-size: 18rpx;
  /** 选中时字体的颜色**/
  --seletct--text-color: white;
  /** 选中时背景颜色 **/
  --seletct--background-color: #005ceeff;
  /** 处于区间的字体颜色 **/
  --in-range--text-color: #005ceeff;
  /** 处于区间的背景颜色 **/
  --in-range--background-color: #f2f6fc;
  /** 限制的字体颜色 **/
  --limit--text-color: #a8abb2;
  /** 限制的背景颜色 **/
  --limit--background-color: #f5f7fa;
  /** current的背景颜色 **/
  --current--background-color: #e4edfe;
  /** current的字体颜色 **/
  --current--text-color: #7994b2;
```


### 如果这个插件有帮助到你，不妨给个好评吧！
### 你的好评是给作者的最大鼓励🫡

