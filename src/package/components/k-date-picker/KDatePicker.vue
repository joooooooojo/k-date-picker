<script>
import dayjs from "./day";
import {
  monthsChinese,
  weeksChinese,
  closable,
  left,
  left_double,
} from "./utils";
import KToast from "../k-toast/KToast.vue";
import KButton from "../k-bottom-button/KBottomButton.vue";
export default {
  components: { KButton, KToast },
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
    confirmText: {
      type: String,
      default: "确定",
    },
    title: {
      type: String,
      default: "请选择",
    },
    modelValue: {
      type: Boolean,
      default: false,
    },
    defaultValue: {
      type: [String, Number, Array],
    },
    type: {
      type: String, //"day" | "month" | "year"
      default: "day",
    },
    safeArea: {
      type: Boolean,
      default: true,
    },
    /** 限制开始时间 **/
    limitStartDate: {
      type: [String, Number],
    },
    /** 限制结束时间 **/
    limitEndDate: {
      type: [String, Number],
    },
    /** 格式化数据 **/
    formatter: {
      type: String,
    },
    /** 是否为范围选择器 **/
    isRange: {
      type: Boolean,
      default: () => false,
    },
    /** 是否为多选 **/
    multiple: {
      type: Boolean,
      default: () => false,
    },
  },
  computed: {
    briefDate() {
      if (this.type === "day") {
        return this.currentDay.format("YYYY年MM月");
      }
      if (this.type === "month") {
        return this.currentDay.format("YYYY年");
      }
      if (this.type === "year") {
        return `${this.startYear.format("YYYY")} - ${this.startYear.add(9, "year").format("YYYY")}`;
      }
    },
    selectedDate() {
      const formatMap = {
        day: "YYYY-MM-DD",
        month: "YYYY-MM",
        year: "YYYY年",
      };
      const format = formatMap[this.type];
      if (this.isRange) {
        return `${this.rangeStart ? this.rangeStart.format(format) : "-"} 至 ${this.rangeEnd ? this.rangeEnd.format(format) : "-"}`;
      } else {
        if (this.checked.length > 0) {
          return this.checked.map((item) => item.format(format)).join("、");
        } else {
          return "-";
        }
      }
    },
    backText() {
      if (this.type === "day") {
        return this.currentDay.format("M");
      }
      return "";
    },
    blocks() {
      const dates =
        this.type === "day"
          ? this.days
          : this.type === "month"
            ? this.months
            : this.years;
      const result = [];
      const chineseMap = {
        day: (date) => {
          return date.format("D");
        },
        month: (date) => {
          return monthsChinese[date.month()];
        },
        year: (date) => {
          return date.format("YYYY年");
        },
      };
      for (const date of dates) {
        if (!date) {
          result.push(date);
        } else {
          const obj = {
            date,
            chinese: chineseMap[this.type](date),
            isLimit:
              (this.limitStartDate &&
                date.isBefore(dayjs(this.limitStartDate))) ||
              (this.limitEndDate && date.isAfter(dayjs(this.limitEndDate))),
            isCurrent: date
              .startOf(this.type)
              .isSame(this.sameDay.startOf(this.type)),
            isSelect:
              this.checked.findIndex(
                (item) => item.valueOf() === date.valueOf(),
              ) !== -1,
            isStart:
              this.rangeStart &&
              date
                .startOf(this.type)
                .isSame(this.rangeStart.startOf(this.type)),
            isEnd:
              this.rangeEnd &&
              date.startOf(this.type).isSame(this.rangeEnd.startOf(this.type)),
            isInRange:
              this.rangeStart &&
              this.rangeEnd &&
              date
                .startOf(this.type)
                .isAfter(this.rangeStart.startOf(this.type)) &&
              date
                .startOf(this.type)
                .isBefore(this.rangeEnd.startOf(this.type)),
          };
          // 若果既是范围起点又是范围终点，那么设置为选中
          if (this.isRange) {
            obj.isSelect = obj.isStart && obj.isEnd;
          }
          result.push(obj);
        }
      }
      return result;
    },
    days() {
      const result = [];
      let start = this.currentDay.startOf("month");
      const end = this.currentDay.endOf("month");
      //计算当前月的第一天是星期几，然后在result头部补上空字符串
      const firstDayWeek = start.day();
      for (let i = 0; i < firstDayWeek; i++) {
        result.push("");
      }
      while (start.isBefore(end)) {
        result.push(start);
        start = start.add(1, "day");
      }
      return result;
    },
    months() {
      const result = [];
      for (let i = 0; i < 12; i++) {
        const date = this.currentDay.startOf("year").add(i, "month");
        result.push(date);
      }
      return result;
    },
    startYear() {
      return this.currentDay.subtract(this.currentDay.year() % 10, "year");
    },
    years() {
      // 根据currentDay生成10个年份，如果是当前年则加上is-current
      const result = [];
      // 从startYear开始生成一个10年的数组
      for (let i = 0; i < 10; i++) {
        const date = this.startYear.add(i, "year");
        result.push(date);
      }
      return result;
    },
  },
  data() {
    return {
      sameDay: dayjs(), // 当日
      rangeStart: null, // 范围选择器开始日期
      rangeEnd: null, // 范围选择器结束日期
      checked: [], // 选中的日期
      currentDay: dayjs(), // 当前选择器显示的日期
      weeksChinese,
      left,
      closable,
      left_double,
    };
  },
  methods: {
    onClick() {
      if (this.isRange) {
        if (this.rangeStart && this.rangeEnd) {
          this.$emit("update:modelValue", false);
          this.$emit("change", [
            this.formatter
              ? this.rangeStart.format(this.formatter)
              : this.rangeStart.valueOf(),
            this.formatter
              ? this.rangeEnd.format(this.formatter)
              : this.rangeEnd.valueOf(),
          ]);
        } else {
          this.$refs.kToast.show({
            type: "warning",
            message: "请选择开始日期和结束日期",
          });
        }
      } else {
        if (this.selectedDate) {
          this.$emit("update:modelValue", false);
          if (this.multiple) {
            this.$emit(
              "change",
              this.checked.map((item) =>
                this.formatter ? item.format(this.formatter) : item.valueOf(),
              ),
            );
          } else {
            this.$emit(
              "change",
              this.formatter
                ? this.checked[0].format(this.formatter)
                : this.checked[0].valueOf(),
            );
          }
        } else {
          this.$refs.kToast.show({ type: "warning", message: "请选择日期" });
        }
      }
    },
    handleClickItem(block) {
      /**当day为空字符串或者在限制开始时间之前，限制开始时间之后 不允许点击 **/
      if (
        block === "" ||
        (this.limitEndDate && block.date.isAfter(dayjs(this.limitEndDate))) ||
        (this.limitStartDate && block.date.isBefore(dayjs(this.limitStartDate)))
      )
        return;
      if (this.isRange) {
        //如果有结束日期和开始日期，并且点击的不是开始日期和结束日期，则将其设置为开始日期并清除结束日期
        if (
          this.rangeStart &&
          this.rangeEnd &&
          !block.isStart &&
          !block.isEnd
        ) {
          this.rangeStart = block.date;
          this.rangeEnd = null;
          return;
        }
        // 如果点击的是选择的日期，那么就清除这些选项
        if (block.isSelect) {
          this.rangeStart = null;
          this.rangeEnd = null;
          return;
        }

        // 如果重复点击开始日期或者结束日期则取消选择
        if (block.isStart) {
          // 如果没有结束日期 那么将结束日期设置为开始日期
          if (!this.rangeEnd) {
            this.rangeEnd = block.date;
            return;
          }
          this.rangeStart = null;
          return;
        }
        if (block.isEnd) {
          this.rangeEnd = null;
          return;
        }
        const setStartAndEndDay = (start) => {
          if (block.date.isBefore(start)) {
            this.rangeEnd = start;
            this.rangeStart = block.date;
          } else {
            this.rangeEnd = block.date;
          }
        };
        if (this.rangeStart) {
          setStartAndEndDay(this.rangeStart);
        } else {
          // 如果有结束日期，需要判断是否在结束日期之前
          if (this.rangeEnd) {
            setStartAndEndDay(this.rangeEnd);
          } else {
            this.rangeStart = block.date;
          }
        }
      } else {
        // 如果重复点击则取消选择
        if (block.isSelect) {
          this.checked = this.checked.filter(
            (item) => item.valueOf() !== block.date.valueOf(),
          );
          return;
        }
        this.multiple
          ? this.checked.push(block.date)
          : (this.checked = [block.date]);
      }
    },
    toggle(type, direction) {
      if (this.type === "day") {
        this.currentDay = this.currentDay.add(
          direction === "left" ? -1 : 1,
          type === "single" ? "month" : "year",
        );
      }
      if (this.type === "month") {
        this.currentDay = this.currentDay.add(
          direction === "left" ? -1 : 1,
          "year",
        );
      }
      if (this.type === "year") {
        this.currentDay = this.currentDay.add(
          direction === "left" ? -10 : 10,
          "year",
        );
      }
    },
    close() {
      this.$emit("update:modelValue", false);
    },
    init() {
      // 检查类型是否为 day | month | year
      if (!["day", "month", "year"].includes(this.type)) {
        console.error("类型必须为 day | month | year");
        return;
      }
      // 检查 isRange和 multiple 是否同时为 true
      if (this.isRange && this.multiple) {
        console.error("目前暂不支持时间段的多选");
        return;
      }

      // 清空原数据
      this.checked = [];
      this.rangeStart = null;
      this.rangeEnd = null;
      // 初始化数据
      if (!this.defaultValue) return;
      if (this.isRange) {
        if (
          !Array.isArray(this.defaultValue) ||
          this.defaultValue.length !== 2
        ) {
          this.$refs.kToast.show("范围选择器的默认值必须为长度为2的数组");
          return;
        }
        this.currentDay = dayjs(this.defaultValue[0]);
        this.rangeStart = dayjs(this.defaultValue[0]);
        this.rangeEnd = dayjs(this.defaultValue[1]);
      } else {
        if (this.multiple && Array.isArray(this.defaultValue)) {
          this.checked = this.defaultValue.map((item) => dayjs(item));
        } else {
          this.checked = [dayjs(this.defaultValue)];
        }
        this.currentDay = this.checked[0] ? dayjs(this.checked[0]) : dayjs();
      }
    },
  },
  mounted() {
    this.init();
  },
  watch: {
    modelValue: {
      handler(value) {
        if (value) {
          this.$refs.popup.open();
        } else {
          this.$refs.popup.close();
        }
      },
    },
    type: {
      handler() {
        this.init();
      },
    },
    defaultValue: {
      handler() {
        this.init();
      },
      deep: true,
    },
  },
};
</script>

<template>
  <uni-popup type="bottom" ref="popup" :safe-area="false" @maskClick="close">
    <view class="k-date-picker">
      <view class="inner-top">
        <view>
          <slot name="left">
            <text>{{ title }}</text>
          </slot>
        </view>
        <view @click="close">
          <slot name="right">
            <image :src="closable" class="image-closable"></image>
          </slot>
        </view>
      </view>
      <view class="inner-body">
        <view class="inner-body__arrow">
          <image
            v-if="type === 'day'"
            class="image-arrow"
            :src="left_double"
            @click="toggle('double', 'left')"
          />
          <image
            class="image-arrow"
            :src="left"
            @click="toggle('single', 'left')"
          />
          <view class="brief">{{ briefDate }}</view>
          <image
            class="image-arrow right-arrow"
            :src="left"
            @click="toggle('single', 'right')"
          />
          <image
            v-if="type === 'day'"
            class="image-arrow right-arrow"
            :src="left_double"
            @click="toggle('double', 'right')"
          />
        </view>
        <view class="inner-body__calendar">
          <view
            class="inner-body__calendar_week"
            :style="{ columnGap: isRange ? '0' : '12rpx' }"
            v-if="type === 'day'"
          >
            <view
              class="inner-body__calendar_week--item"
              v-for="(item, index) in weeksChinese"
              :key="index"
              >{{ item }}</view
            >
          </view>
          <view
            class="inner-body__calendar_block"
            :class="{
              'is-day': type === 'day',
              'is-month': type === 'month',
              'is-year': type === 'year',
            }"
            :data-text="backText"
            :style="{ columnGap: isRange ? '0' : '12rpx' }"
          >
            <view
              class="inner-body__calendar_block--item"
              :class="{
                'is-current': block.isCurrent,
                'is-start': block.isStart,
                'is-end': block.isEnd,
                'is-in-range': block.isInRange,
                'is-selected': block.isSelect,
                'is-limit': block.isLimit,
              }"
              v-for="(block, index) in blocks"
              :key="index"
              @click="handleClickItem(block)"
            >
              {{ block.chinese }}
            </view>
          </view>
        </view>
        <view class="selected-wrap" v-if="!multiple">
          <view class="selected-date">
            {{ selectedDate }}
          </view>
        </view>
      </view>
    </view>
    <KButton
      :label="confirmText"
      :box-shadow="false"
      :safe-area="safeArea"
      :on-click="onClick"
    ></KButton>
    <KToast ref="kToast"></KToast>
  </uni-popup>
</template>

<style lang="scss">
.k-date-picker {
  /** 控制选中文字的位置 **/
  --bottom-offset: 10%;
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

  box-sizing: border-box;
  width: 100%;
  padding: 30rpx 30rpx 20rpx;
  background-color: white;
  border-radius: 10px 10px 0 0;
  position: relative;
  display: flex;
  flex-direction: column;
  .is-current {
    border-radius: 8rpx;
    background-color: var(--current--background-color);
    position: relative;
    --color: var(--current--text-color);
    &:after {
      color: var(--color);
      position: absolute;
      left: 50%;
      bottom: var(--bottom-offset);
      transform: translate(-50%);
      font-size: 22rpx;
      /* #ifdef APP-PLUS */
      font-weight: 500;
      /* #endif */
    }
  }
  .is-day {
    grid-template-columns: repeat(7, minmax(0, 1fr));
    .is-current:after {
      content: "本日";
      bottom: 6%;
    }
  }
  .is-month {
    grid-template-columns: repeat(4, minmax(0, 1fr));
    margin-top: 24rpx;
    .is-current:after {
      content: "本月";
    }
  }
  .is-year {
    grid-template-columns: repeat(4, minmax(0, 1fr));
    margin-top: 24rpx;
    .is-current:after {
      content: "本年";
    }
  }
  .inner-top {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 30rpx;
  }
  .inner-body {
    &__arrow {
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: center;
    }
    &__calendar {
      &_week {
        display: grid;
        grid-template-columns: repeat(7, minmax(0, 1fr));
        &--item {
          /* #ifdef MP-WEIXIN */
          font-weight: bolder;
          /* #endif */
          /* #ifdef APP-PLUS */
          font-weight: 500;
          /* #endif */
          color: #303133;
          font-size: 30rpx;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          height: 88rpx;
        }
      }

      &_block {
        &:after {
          display: block;
          content: attr(data-text);
          color: #eef2f8ff;
          font-size: 280rpx;
          position: absolute;
          left: 50%;
          top: 50%;
          transform: translate(-50%, -50%);
          z-index: 1;
        }
        position: relative;
        display: grid;
        row-gap: 20rpx;
        &--item {
          z-index: 2;
          font-size: 30rpx;
          color: #7994b2;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          aspect-ratio: 1;
        }
      }
    }
  }
  .is-limit {
    color: var(--limit--text-color) !important;
    background-color: var(--limit--background-color);
  }
  .is-in-range {
    --color: var(--in-range--text-color) !important;
    background-color: var(--in-range--background-color) !important;
    border-radius: 0 !important;
    color: var(--in-range--text-color) !important;
  }
  .is-selected {
    &:after {
      content: "已选" !important;
      color: var(--seletct--text-color) !important;
      position: absolute;
      left: 50%;
      bottom: var(--bottom-offset);
      transform: translate(-50%);
      font-size: 18rpx;
      /* #ifdef APP-PLUS */
      font-weight: 500;
      /* #endif */
    }
    position: relative;
    background-color: var(--seletct--background-color) !important;
    color: var(--seletct--text-color) !important;
    border-radius: 8rpx !important;
  }
  .is-start {
    &:after {
      content: "开始";
      color: var(--seletct--text-color) !important;
      position: absolute;
      left: 50%;
      bottom: var(--bottom-offset);
      transform: translate(-50%);
      font-size: 18rpx;
      /* #ifdef APP-PLUS */
      font-weight: 500;
      /* #endif */
    }
    position: relative;
    background-color: var(--seletct--background-color) !important;
    color: var(--seletct--text-color) !important;
    border-radius: 8rpx 0 0 8rpx;
  }
  .is-end {
    &:after {
      content: "结束";
      color: var(--seletct--text-color) !important;
      position: absolute;
      left: 50%;
      bottom: var(--bottom-offset);
      transform: translate(-50%);
      font-size: 18rpx;
      /* #ifdef APP-PLUS */
      font-weight: 500;
      /* #endif */
    }
    position: relative;
    background-color: var(--seletct--background-color) !important;
    color: var(--seletct--text-color) !important;
    border-radius: 0 8rpx 8rpx 0;
  }
  .image-closable {
    width: 36rpx;
    height: 36rpx;
  }
  .image-arrow {
    width: 36rpx;
    height: 36rpx;
  }
  .right-arrow {
    transform: rotate(180deg);
  }
  .brief {
    margin: 0 30rpx;
    color: #000000d9;
    font-size: 30rpx;
    /* #ifdef MP-WEIXIN */
    font-weight: bolder;
    /* #endif */
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
  .selected-date {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    padding: 10rpx 0;
    color: #94a0b4ff;
    font-size: 30rpx;
    margin-top: 20rpx;
    width: 600rpx;
  }
  .selected-wrap {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    width: 100%;
  }
}
</style>
