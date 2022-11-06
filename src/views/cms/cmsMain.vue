<template>
  <el-main>
    <router-view :key="key"/>
  </el-main>
</template>

<script setup>

import {computed} from "vue";

const router = useRouter();
const key = computed(() => {
  return router.path
})
</script>

<style scoped>
.el-main {
  min-height: 650px;
}
</style>
