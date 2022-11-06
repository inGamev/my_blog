<template>
  <el-container>
    <iframe
        :src="docHtml"
        id="qt"
        scrolling="no"
        frameborder="0"
        style="
        position: absolute;
        top: 64px;
        left: 0px;
        right: 0px;
        bottom: 100px;
        opacity: 1;
      "
    ></iframe>
  </el-container>
</template>

<script setup>

import {onMounted} from "vue";

const docHtml = ref('/docs/index.html')

onMounted(() => {
      changeIframe();
      window.onresize = function () {
        changeIframe()
      };
    }
)

function changeIframe() {
  const qt = document.getElementById('qt')
  const deviceWidth = document.body.clientWidth
  const deviceHeight = document.body.clientHeight
  qt.style.width = Number(deviceWidth) + 'px' // 数字是页面布局宽度差值
  qt.style.height = Number(deviceHeight) + 'px' // 数字是页面布局高度差
}

</script>

<style lang="scss" rel="stylesheet/scss" scoped></style>
