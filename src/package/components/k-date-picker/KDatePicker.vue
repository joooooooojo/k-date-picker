<script>
import { computed, onBeforeUnmount, onMounted, ref, watch } from "vue";
import KButton from "../k-bottom-button/KBottomButton.vue";
import dayjs from "dayjs";
import KToast from "../k-toast/KToast.vue";
import { left, left_double, closable } from "./icons";

export default {
  name: "KPicker",
  components: { KButton, KToast },
  options: {
    virtualHost: true,
  },
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
      type: String, //"date" | "month" | "year" | "daterange" | "monthrange" | "yearrange"
      default: "date",
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
  },
  setup(props, { emit }) {
    const kToast = ref();
    const startDay = ref();
    const endDay = ref();
    const selectedDay = ref();
    const weekList = ["日", "一", "二", "三", "四", "五", "六"];
    const monthsChinese = [
      "一月",
      "二月",
      "三月",
      "四月",
      "五月",
      "六月",
      "七月",
      "八月",
      "九月",
      "十月",
      "十一月",
      "十二月",
    ];
    const currentDay = ref(dayjs(new Date()));
    const now = ref(dayjs());
    const briefDate = computed(() => {
      if (props.type === "date" || props.type === "daterange") {
        return currentDay.value.format("YYYY年MM月");
      }
      if (props.type === "month" || props.type === "monthrange") {
        return currentDay.value.format("YYYY年");
      }
      if (props.type === "year" || props.type === "yearrange") {
        return `${startYear.value.format("YYYY")} - ${startYear.value.add(9, "year").format("YYYY")}`;
      }
    });

    const days = computed(() => {
      //强制触发依赖收集
      [startDay.value, endDay.value, selectedDay.value];
      const result = [];
      if (props.type === "date" || props.type === "daterange") {
        let start = currentDay.value.startOf("month");
        const end = currentDay.value.endOf("month");
        //计算当前月的第一天是星期几，然后在result头部补上空字符串
        const firstDayWeek = start.day();
        for (let i = 0; i < firstDayWeek; i++) {
          result.push("");
        }
        while (start.isBefore(end)) {
          const obj = {
            date: start,
            format: start.format("D"),
            isToday: start.isSame(now.value.startOf("day")),
            isStart: false,
            isEnd: false,
            isInRange: false,
            isSelect: false,
            isLimit:
              (props.limitStartDate &&
                start.isBefore(dayjs(props.limitStartDate))) ||
              (props.limitEndDate && start.isAfter(dayjs(props.limitEndDate))),
          };
          //如果type是daterange，需要判断是否在选择范围内和是否是开始或结束日期
          if (props.type === "daterange") {
            obj.isStart = startDay.value && start.isSame(startDay.value, "day");
            obj.isEnd = endDay.value && start.isSame(endDay.value, "day");
            obj.isInRange =
              startDay.value &&
              endDay.value &&
              start.isBefore(endDay.value) &&
              start.isAfter(startDay.value);
          }
          // 如果既是开始点又是结束点，那么设置选中此处
          if (obj.isStart && obj.isEnd) {
            obj.isSelect = true;
          }

          if (props.type === "date") {
            obj.isSelect =
              selectedDay.value &&
              start.isSame(selectedDay.value.startOf("day").valueOf());
          }

          result.push(obj);
          start = start.add(1, "day");
        }
      }
      return result;
    });

    const popup = ref();
    const watcher = watch(
      () => props.modelValue,
      (value) => {
        if (value) {
          popup.value.open();
        } else {
          popup.value.close();
        }
      },
    );

    const close = () => {
      emit("update:modelValue", false);
    };
    const onClick = () => {
      if (
        props.type === "daterange" ||
        props.type === "monthrange" ||
        props.type === "yearrange"
      ) {
        if (startDay.value && endDay.value) {
          emit("update:modelValue", false);
          emit("change", [
            props.formatter
              ? startDay.value.format(props.formatter)
              : startDay.value.valueOf(),
            props.formatter
              ? endDay.value.format(props.formatter)
              : endDay.value.valueOf(),
          ]);
        } else {
          kToast.value.show({
            type: "warning",
            message: "请选择开始日期和结束日期",
          });
        }
      } else {
        if (selectedDay.value) {
          emit("update:modelValue", false);
          emit(
            "change",
            props.formatter
              ? selectedDay.value.format(props.formatter)
              : selectedDay.value.valueOf(),
          );
        } else {
          kToast.value.show({ type: "warning", message: "请选择日期" });
        }
      }
    };
    const handleClickItem = (day) => {
      /**当day为空字符串或者在限制开始时间之前，限制开始时间之后 不允许点击 **/
      if (day === "") return;
      if (
        props.limitStartDate &&
        day.date.isBefore(dayjs(props.limitStartDate))
      ) {
        return;
      }
      if (props.limitEndDate && day.date.isAfter(dayjs(props.limitEndDate))) {
        return;
      }

      if (
        props.type === "daterange" ||
        props.type === "monthrange" ||
        props.type === "yearrange"
      ) {
        //如果有结束日期和开始日期，并且点击的不是开始日期和结束日期，则将其设置为开始日期并清除结束日期
        if (startDay.value && endDay.value && !day.isStart && !day.isEnd) {
          startDay.value = day.date;
          endDay.value = null;
          return;
        }
        // 如果点击的是选择的日期，那么就清楚这些选项
        if (day.isSelect) {
          startDay.value = null;
          endDay.value = null;
          return;
        }

        // 如果重复点击开始日期或者结束日期则取消选择
        if (day.isStart) {
          // 如果没有结束日期 那么将结束日期设置为开始日期
          if (!endDay.value) {
            endDay.value = day.date;
            return;
          }
          startDay.value = null;
          return;
        }
        if (day.isEnd) {
          endDay.value = null;
          return;
        }
        const setStartAndEndDay = (start) => {
          if (day.date.isBefore(start)) {
            endDay.value = start;
            startDay.value = day.date;
          } else {
            endDay.value = day.date;
          }
        };
        if (startDay.value) {
          setStartAndEndDay(startDay.value);
        } else {
          // 如果有结束日期，需要判断是否在结束日期之前
          if (endDay.value) {
            setStartAndEndDay(endDay.value);
          } else {
            startDay.value = day.date;
          }
        }
      } else {
        // 如果重复点击则取消选择
        if (day.isSelect) {
          selectedDay.value = null;
          return;
        }
        selectedDay.value = day.date;
      }
    };

    const toggle = (type, direction) => {
      if (props.type === "date" || props.type === "daterange") {
        currentDay.value = currentDay.value.add(
          direction === "left" ? -1 : 1,
          type === "single" ? "month" : "year",
        );
      }
      if (props.type === "month" || props.type === "monthrange") {
        currentDay.value = currentDay.value.add(
          direction === "left" ? -1 : 1,
          "year",
        );
      }
      if (props.type === "year" || props.type === "yearrange") {
        startYear.value = startYear.value.add(
          direction === "left" ? -10 : 10,
          "year",
        );
      }
    };

    const selectedDate = computed(() => {
      const formatMap = {
        daterange: "YYYY-MM-DD",
        monthrange: "YYYY-MM",
        yearrange: "YYYY年",
        date: "YYYY-MM-DD",
        month: "YYYY-MM",
        year: "YYYY年",
      };
      const format = formatMap[props.type];
      if (["daterange", "monthrange", "yearrange"].includes(props.type)) {
        return `${startDay.value ? startDay.value.format(format) : "-"} 至 ${endDay.value ? endDay.value.format(format) : "-"}`;
      } else {
        return selectedDay.value ? selectedDay.value.format(format) : "-";
      }
    });

    const backText = computed(() => {
      if (props.type === "daterange" || props.type === "date") {
        return currentDay.value.format("M");
      }
    });

    const months = computed(() => {
      const result = [];
      // 根据currentDay生成12个月份，如果是当前月份则加上is-current
      for (let i = 0; i < 12; i++) {
        const date = currentDay.value.startOf("year").add(i, "month");
        const obj = {
          date,
          isLimit:
            (props.limitStartDate &&
              date.isBefore(dayjs(props.limitStartDate))) ||
            (props.limitEndDate && date.isAfter(dayjs(props.limitEndDate))),
          chinese: monthsChinese[i],
          isCurrent: date.startOf("month").isSame(now.value.startOf("month")),
          isSelect:
            selectedDay.value &&
            date.startOf("month").isSame(selectedDay.value.startOf("month")),
          isStart:
            startDay.value &&
            date.startOf("month").isSame(startDay.value.startOf("month")),
          isEnd:
            endDay.value &&
            date.startOf("month").isSame(endDay.value.startOf("month")),
          isInRange:
            startDay.value &&
            endDay.value &&
            date.startOf("month").isAfter(startDay.value.startOf("month")) &&
            date.startOf("month").isBefore(endDay.value.startOf("month")),
        };
        if (obj.isStart && obj.isEnd) {
          obj.isSelect = true;
        }
        result.push(obj);
      }
      return result;
    });
    const startYear = ref(
      currentDay.value.subtract(currentDay.value.year() % 10, "year"),
    );
    const years = computed(() => {
      // 根据currentDay生成10个年份，如果是当前年则加上is-current
      const result = [];
      // 从startYear开始生成一个10年的数组
      for (let i = 0; i < 10; i++) {
        const date = startYear.value.add(i, "year");
        const obj = {
          date,
          chinese: date.format("YYYY年"),
          isLimit:
            (props.limitStartDate &&
              date.isBefore(dayjs(props.limitStartDate))) ||
            (props.limitEndDate && date.isAfter(dayjs(props.limitEndDate))),
          isCurrent: date.startOf("year").isSame(now.value.startOf("year")),
          isSelect:
            selectedDay.value &&
            date.startOf("year").isSame(selectedDay.value.startOf("year")),
          isStart:
            startDay.value &&
            date.startOf("year").isSame(startDay.value.startOf("year")),
          isEnd:
            endDay.value &&
            date.startOf("year").isSame(endDay.value.startOf("year")),
          isInRange:
            startDay.value &&
            endDay.value &&
            date.startOf("year").isAfter(startDay.value.startOf("year")) &&
            date.startOf("year").isBefore(endDay.value.startOf("year")),
        };
        if (obj.isStart && obj.isEnd) {
          obj.isSelect = true;
        }
        result.push(obj);
      }
      return result;
    });

    const init = (type) => {
      // 清空原数据
      startDay.value = null;
      endDay.value = null;
      selectedDay.value = null;
      // 初始化数据
      if (!props.defaultValue) return;
      if (
        Array.isArray(props.defaultValue) &&
        ["daterange", "monthrange", "yearrange"].includes(type)
      ) {
        startDay.value = dayjs(props.defaultValue[0]);
        endDay.value = dayjs(props.defaultValue[1]);
      } else if (!Array.isArray(props.defaultValue)) {
        selectedDay.value = dayjs(props.defaultValue);
      }
    };

    const typeWatcher = watch(
      () => [props.type],
      () => {
        init(props.type);
      },
    );

    const defaultValueWatcher = watch(
      () => props.defaultValue,
      () => {
        init(props.type);
      },
      {
        deep: true,
      },
    );

    onMounted(() => {
      init(props.type);
    });
    onBeforeUnmount(() => {
      watcher();
      typeWatcher();
      defaultValueWatcher();
    });
    return {
      popup,
      close,
      onClick,
      briefDate,
      weekList,
      days,
      left,

      left_double,
      closable,
      handleClickItem,
      toggle,
      selectedDate,
      backText,
      months,
      years,
      kToast,
    };
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
            v-if="type === 'date' || type === 'daterange'"
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
            v-if="type === 'date' || type === 'daterange'"
            class="image-arrow right-arrow"
            :src="left_double"
            @click="toggle('double', 'right')"
          />
        </view>
        <view
          class="inner-body__calendar"
          v-if="type === 'date' || type === 'daterange'"
        >
          <view class="inner-body__calendar_week">
            <view
              class="inner-body__calendar_week--item"
              v-for="(item, index) in weekList"
              :key="index"
              >{{ item }}</view
            >
          </view>
          <view class="inner-body__calendar_day" :data-text="backText">
            <view
              class="inner-body__calendar_day--item"
              :class="{
                'is-today': day.isToday,
                'is-start': day.isStart,
                'is-end': day.isEnd,
                'is-in-range': day.isInRange,
                'is-selected': day.isSelect,
                'is-limit': day.isLimit,
              }"
              v-for="(day, index) in days"
              :key="index"
              @click="handleClickItem(day)"
            >
              {{ day.format }}
            </view>
          </view>
        </view>
        <view
          class="inner-body__months"
          v-else-if="type === 'month' || type === 'monthrange'"
        >
          <view
            class="inner-body__months_item"
            :class="{
              'is-month-current': month.isCurrent,
              'is-selected': month.isSelect,
              'is-start': month.isStart,
              'is-end': month.isEnd,
              'is-in-range': month.isInRange,
              'is-limit': month.isLimit,
            }"
            v-for="(month, index) in months"
            :key="index"
            @click="handleClickItem(month)"
          >
            {{ month.chinese }}
          </view>
        </view>
        <view class="inner-body__years" v-else>
          <view
            class="inner-body__years_item"
            :class="{
              'is-year-current': year.isCurrent,
              'is-selected': year.isSelect,
              'is-start': year.isStart,
              'is-end': year.isEnd,
              'is-in-range': year.isInRange,
              'is-limit': year.isLimit,
            }"
            v-for="(year, index) in years"
            :key="index"
            @click="handleClickItem(year)"
          >
            {{ year.chinese }}
          </view>
        </view>
        <view class="selected-date">
          {{ selectedDate }}
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
  box-sizing: border-box;
  width: 100%;
  padding: 30rpx 30rpx 20rpx;
  background-color: white;
  border-radius: 10px 10px 0 0;
  position: relative;
  display: flex;
  flex-direction: column;
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
    &__years {
      display: grid;
      grid-template-columns: repeat(4, minmax(0, 1fr));
      margin-top: 24rpx;
      &_item {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        aspect-ratio: 1;
        font-size: 30rpx;
        color: #303133;
        /* #ifdef MP-WEIXIN */
        font-weight: bolder;
        /* #endif */
        /* #ifdef APP-PLUS */
        font-weight: 500;
        /* #endif */
      }
    }
    &__months {
      display: grid;
      grid-template-columns: repeat(4, minmax(0, 1fr));
      margin-top: 24rpx;
      &_item {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        aspect-ratio: 1;
        font-size: 30rpx;
        color: #303133;
        /* #ifdef MP-WEIXIN */
        font-weight: bolder;
        /* #endif */
        /* #ifdef APP-PLUS */
        font-weight: 500;
        /* #endif */
      }
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
      &_day {
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
        grid-template-columns: repeat(7, minmax(0, 1fr));
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
}

.is-selected {
  &:after {
    content: "已选" !important;
    color: white !important;
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 18rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
  --color: white;
  position: relative;
  background-color: #005ceeff !important;
  color: white !important;
  border-radius: 8rpx !important;
}

.is-today {
  border-radius: 8rpx;
  background-color: #e4edfe;
  position: relative;
  --color: #7994b2;
  &:after {
    content: "今天";
    color: var(--color);
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 18rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
}

.is-start {
  &:after {
    content: "开始";
    color: white !important;
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 18rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
  position: relative;
  background-color: #005ceeff !important;
  color: white !important;
  border-radius: 8rpx 0 0 8rpx;
}
.is-end {
  &:after {
    content: "结束";
    color: white !important;
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 18rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
  position: relative;
  background-color: #005ceeff !important;
  color: white !important;
  border-radius: 0 8rpx 8rpx 0;
}
.is-in-range {
  --color: #005ceeff !important;
  background-color: #f2f6fc !important;
  border-radius: 0 !important;
  color: #005ceeff !important;
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
}

.is-month-current {
  border-radius: 8rpx;
  background-color: #e4edfe;
  position: relative;
  --color: #7994b2;
  &:after {
    content: "本月";
    color: var(--color);
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 22rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
}
.is-year-current {
  border-radius: 8rpx;
  background-color: #e4edfe;
  position: relative;
  --color: #7994b2;
  &:after {
    content: "本年";
    color: var(--color);
    position: absolute;
    left: 50%;
    bottom: 10%;
    transform: translate(-50%);
    font-size: 22rpx;
    /* #ifdef APP-PLUS */
    font-weight: 500;
    /* #endif */
  }
}
.is-limit {
  color: #a8abb2 !important;
  background-color: #f5f7fa;
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
</style>
