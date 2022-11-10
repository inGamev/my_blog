<template>
  <div class="editor">
    <!-- props传递有延迟,有内容再渲染-->
    <!-- 自定义toolbar 使其内容为空不显示-->
    <QuillEditor
        v-if="props.content"
        toolbar="#my-toolbar"
        v-model:content="props.content"
        :readOnly="true"
        contentType="html"/>
    <template>
      <div id="my-toolbar"></div>
    </template>
  </div>
</template>

<script setup>
import {QuillEditor, Quill} from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css';
import {getCurrentInstance, ref, watch} from "vue";

const {proxy} = getCurrentInstance();
const props = defineProps({
  /* 编辑器的内容   */
  content: {
    type: String,
    default: '',
  },
})

</script>
<style>
.editor,
.ql-toolbar {
  white-space: pre-wrap !important;
  line-height: normal !important;
}

</style>
