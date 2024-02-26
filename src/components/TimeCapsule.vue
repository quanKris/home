<template>
  <div class="time-capsule">
    <div class="title">
      <hourglass-full theme="two-tone" size="24" :fill="['#efefef', '#00000020']" />
      <span>时光胶囊</span>
    </div>
    <span class="text">距离下班还有&nbsp;{{ timeRemaining.hours }}&nbsp;小时&nbsp;{{ timeRemaining.minutes }}&nbsp;分钟&nbsp;{{ timeRemaining.seconds }}&nbsp;秒</span>
    <span class="text">今日已经度过了&nbsp;{{ timeData.day.elapsed }}&nbsp;小时</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.day.pass" />
    <span class="text">本周已经度过了&nbsp;{{ timeData.week.elapsed }}&nbsp;天</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.week.pass" />
    <span class="text">本月已经度过了&nbsp;{{ timeData.month.elapsed }}&nbsp;天</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.month.pass" />
    <span class="text">今年已经度过了&nbsp;{{ timeData.year.elapsed }}&nbsp;个月</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.year.pass" />
    <div v-if="startDate?.length >= 4 && store.siteStartShow">
      <span class="text" v-html="startDateText" />
      <!-- <el-progress
        :show-text="false"
        :indeterminate="true"
        :stroke-width="6"
        :percentage="80"
        :duration="2"
      /> -->
    </div>
  </div>
</template>

<script setup>
import { HourglassFull } from "@icon-park/vue-next";
import { getTimeCapsule, siteDateStatistics } from "@/utils/getTime.js";
import { mainStore } from "@/store";
const store = mainStore();

// 进度条数据
const timeData = ref(getTimeCapsule());
const startDate = ref(import.meta.env.VITE_SITE_START);
const startDateText = ref(null);
const timeInterval = ref(null);
const endHour = 17; // 设置下班的小时数（24小时制）

const endDate = computed(() => {
  const now = new Date();
  const endDateTime = new Date(now);
  endDateTime.setHours(endHour, 30, 0, 0); // 设置下班时间每天的下午五点半

  // 如果当前时间已经过了下班时间，则下班时间顺延到明天
  if (now >= endDateTime) {
    endDateTime.setDate(endDateTime.getDate() + 1);
  }

  return endDateTime;
});
const timeRemaining = ref({});
const updateTimeData = () => {
    timeRemaining.value = getCurrentTime();
};
const getCurrentTime = ()=>{
  const now = new Date();
  const timeDiff = endDate.value - now;

  if (timeDiff <= 0) {
    // 下班时间已过
    return { hours: 0, minutes: 0, seconds: 0 };
  }

  const hours = Math.floor(timeDiff / (1000 * 60 * 60));
  const minutes = Math.floor((timeDiff % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((timeDiff % (1000 * 60)) / 1000);
  let currentTime = {
    hours,
    minutes,
    seconds,
  };
  return currentTime;
};
onMounted(() => {
  timeInterval.value = setInterval(() => {
    timeData.value = getTimeCapsule();
    updateTimeData()
    if (startDate.value) startDateText.value = siteDateStatistics(new Date(startDate.value));
  }, 1000);
});

onBeforeUnmount(() => {
  clearInterval(timeInterval.value);
});
</script>

<style lang="scss" scoped>
.time-capsule {
  width: 100%;
  .title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2rem 0 1.5rem;
    font-size: 1.1rem;
    .i-icon {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-right: 6px;
    }
  }
  .text {
    display: block;
    margin: 1rem 0rem 0.5rem 0rem;
    font-size: 0.95rem;
  }
}
</style>
