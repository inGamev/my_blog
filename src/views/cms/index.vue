<template>
  <el-container>
    <div class="background" :style="backgroundImage"></div>
    <nav-bar></nav-bar>
    <keep-alive>
      <cms-main></cms-main>
    </keep-alive>
    <el-footer height="40px">
      <cms-footer></cms-footer>
    </el-footer>
  </el-container>
</template>

<script setup>

// 引入背景图
import backgroundImages from './backgroundImages.js'
import {computed} from "vue";
import CmsFooter from "@/views/cms/Footer";
import CmsMain from "@/views/cms/cmsMain";
import NavBar from "@/views/cms/NavBar";
//
const backgroundImage = computed(() => {
      // 根据背景图数组的长度随机选择索引
      const randIndex = Math.floor(Math.random() * backgroundImages.length)
      return {
        // 获取对应的图片资源并将其设置到`background-image`属性上
        backgroundImage: `url(${backgroundImages[randIndex]})`,
      }
    }
)

function handleSelect(key, keyPath) {
  console.log(key, keyPath)
}

</script>

<style scoped>
.el-footer {
  background-color: rgba(84, 92, 100, 0.5);
}

.background {
  background-repeat: no-repeat;
  background-size: cover;
  margin: 0px;
  padding: 0px;
  top: 0;
  width: 100%;
  height: 120vh;
  position: fixed;
  z-index: -1;
}

@media screen and (max-width: 768px) {
  .title {
    width: 100%;
    background-position-x: center;
    background-position-y: 0;
  }
}
</style>
