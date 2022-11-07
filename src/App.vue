<template>
  <!--  <phy-live2d></phy-live2d>-->
  <router-view/>
</template>

<script setup>
import useSettingsStore from '@/store/modules/settings'
import {handleThemeStyle} from '@/utils/theme'
import '/public/live2d/autoload'


onMounted(() => {
  nextTick(() => {
    // 初始化主题样式
    handleThemeStyle(useSettingsStore().theme)
  })
})
</script>
