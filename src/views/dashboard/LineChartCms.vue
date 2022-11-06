<template>
  <div :class="className" :style="{ height: height, width: width }"/>
</template>

<script setup>
import * as echarts from 'echarts';
import resize from './mixins/resize'
import {lineChart} from '@/api/cms/charts'
import {nextTick, onBeforeUnmount, onMounted, watch} from "vue";

const {proxy} = getCurrentInstance();

const mixins = ref([resize])
const props = defineProps({
      className: {
        type: String,
        default: 'chart',
      },
      width: {
        type: String,
        default: '100%',
      },
      height: {
        type: String,
        default: '350px',
      },
      autoResize: {
        type: Boolean,
        default: true,
      },
      chartData: {
        type: Object,
        required: true,
      },
    }
)
const chart = ref(null)
const datex = ref(['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'])
const blogData = ref([100, 120, 161, 134, 105, 160, 165])
const commentData = ref([120, 82, 91, 154, 162, 140, 145])
const messageData = ref([100, 72, 191, 54, 62, 100, 105])
watch(() => props.chartData, value => setOptions(value))

onMounted(
    nextTick(
        initChart()
    )
)
onBeforeUnmount(
    () => {
      if (!chart.value) {
        return
      }
      chart.value.dispose()
      chart.value = null
    }
)

function initChart() {
  getLineChartData()
}

function getLineChartData() {
  lineChart().then((response) => {
    datex.value = response.datex
    blogData.value = response.blogData
    commentData.value = response.commentData
    messageData.value = response.messageData
    chart.value = echarts.init(proxy.$el, 'macarons')
    setOptions(chart.valueData)
  })
}

function setOptions({blogColor, commentColor, messageColor} = {}) {
  chart.value.setOption({
    xAxis: {
      data: datex.value,
      boundaryGap: false,
      axisTick: {
        show: false,
      },
      // axisLabel: {
      //    interval:0,
      //    rotate:40
      // }
    },
    grid: {
      left: 10,
      right: 10,
      bottom: 20,
      top: 30,
      containLabel: true,
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'cross',
      },
      padding: [5, 10],
    },
    yAxis: {
      axisTick: {
        show: false,
      },
    },
    legend: {
      data: ['文章', '评论', '留言'],
    },
    series: [
      {
        name: '文章',
        smooth: true,
        type: 'line',
        itemStyle: {

          color: '#3888fa',
          lineStyle: {
            color: '#3888fa',
            width: 2,
          },
          areaStyle: {
            color: blogColor,
          },

        },
        data: blogData.value,
        animationDuration: 2800,
        animationEasing: 'quadraticOut',
      },
      {
        name: '评论',
        itemStyle: {
          color: '#FF005A',
          lineStyle: {
            color: '#FF005A',
            width: 2,
          },
          areaStyle: {
            color: commentColor,
          },
        },
        smooth: true,
        type: 'line',
        data: commentData.value,
        animationDuration: 2800,
        animationEasing: 'cubicInOut',
      },
      {
        name: '留言',
        itemStyle: {
          color: '#34bfa3',
          lineStyle: {
            color: '#34bfa3',
            width: 2,
          },
          areaStyle: {
            color: messageColor,
          },
        },
        smooth: true,
        type: 'line',
        data: messageData.value,
        animationDuration: 2800,
        animationEasing: 'cubicInOut',
      },
    ],
  })
}
</script>
