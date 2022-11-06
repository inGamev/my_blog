<template>
  <div class="dashboard-editor-container">

    <panel-group-cms @handleSetLineChartData="handleSetLineChartData"></panel-group-cms>
    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <line-chart-cms :chart-data="lineChartData"></line-chart-cms>
    </el-row>
    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <TagChart></TagChart>
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <PieChartCms></PieChartCms>
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <BarChartCms></BarChartCms>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>


import BarChartCms from "@/views/dashboard/BarChartCms";
import PieChartCms from "@/views/dashboard/PieChartCms";
import TagChart from "@/views/dashboard/TagChart";
import LineChartCms from "@/views/dashboard/LineChartCms";
import PanelGroupCms from "@/views/dashboard/PanelGroupCms";

const lineChartDataList = {
  views: {
    blogColor: 'rgba(243, 248, 255, 0)',
    commentColor: 'rgba(243, 248, 255, 0)',
    messageColor: 'rgba(243, 248, 255, 0)',
  },
  blog: {
    blogColor: 'rgba(56, 136, 250, 0.2)',
    commentColor: 'rgba(243, 248, 255, 0)',
    messageColor: 'rgba(243, 248, 255, 0)',
  },
  comment: {
    blogColor: 'rgba(243, 248, 255, 0)',
    commentColor: 'rgba(255, 0, 90, 0.2)',
    messageColor: 'rgba(243, 248, 255, 0)',
  },
  message: {
    blogColor: 'rgba(243, 248, 255, 0)',
    commentColor: 'rgba(243, 248, 255, 0)',
    messageColor: 'rgba(52, 191, 163, 0.2)',
  }
}
const lineChartData = ref(lineChartDataList.views)

function handleSetLineChartData(type) {
  lineChartData.value = lineChartDataList[type]
}
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width: 1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
