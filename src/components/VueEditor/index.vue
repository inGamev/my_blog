<template>
  <el-upload
      :action="uploadUrl"
      :before-upload="handleBeforeUpload"
      :on-success="handleUploadSuccess"
      :on-error="handleUploadError"
      name="file"
      :show-file-list="false"
      :headers="headers"
      class="editor-img-uploader"
      v-if="props.type === 'url'"
  >
    <i ref="uploadRef" class="Plus editor-img-uploader"></i>
  </el-upload>
  <div class="editor">

    <QuillEditor
        id="editorId"
        ref="myQuillEditor"
        v-model:content="editorContent"
        contentType="html"
        :options="options"
    />
    <!--文本改变时触发-->
    <!--    @update:content="onContentChange($event)"-->
  </div>


</template>

<script setup>
import {QuillEditor, Quill} from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css';
import {getToken} from "@/utils/auth";
import {getCurrentInstance, reactive, ref, toRaw} from "vue";

const {proxy} = getCurrentInstance();
const emit = defineEmits(['update:content', 'getFileId'])
const props = defineProps({
  /* 编辑器的内容 */
  content: {
    type: String,
    default: '',
  },
  /* 只读 */
  readOnly: {
    type: Boolean,
    default: false,
  },
  // 上传文件大小限制(MB)
  fileSize: {
    type: Number,
    default: 5,
  },
  /* 类型（base64格式、url格式） */
  type: {
    type: String,
    default: 'url',
  },
})

const editorContent = computed({
  get: () => props.content,
  set: (val) => {
    emit('update:content', val)
  }
});
//
// const onContentChange = (val) => {
//   emit('update', val)
//   console.log('onContentChange!', val)
// }

const myQuillEditor = ref(null)
const uploadUrl = ref(import.meta.env.VITE_APP_BASE_API + '/common/upload') // 上传的图片服务器地址
const headers = reactive({
  Authorization: 'Bearer ' + getToken(),
})

const options = reactive({
  theme: 'snow',
  debug: 'warn',
  modules: {
    // 工具栏配置
    toolbar: {
      container: [
        ['bold', 'italic', 'underline', 'strike'], // 加粗 斜体 下划线 删除线
        ['blockquote', 'code-block'], // 引用  代码块
        [{list: 'ordered'}, {list: 'bullet'}], // 有序、无序列表
        [{indent: '-1'}, {indent: '+1'}], // 缩进
        [{size: ['small', false, 'large', 'huge']}], // 字体大小
        [{header: [1, 2, 3, 4, 5, 6, false]}], // 标题
        [{color: []}, {background: []}], // 字体颜色、字体背景颜色
        [{align: []}], // 对齐方式
        ['clean'], // 清除文本格式
        ['link', 'image', 'video'], // 链接、图片、视频
      ],
      handlers: {
        // 重写图片上传事件
        image: function (value) {
          if (value) {
            //调用图片上传
            proxy.$refs.uploadRef.click()
          } else {
            Quill.format("image", true);
          }
        },
      },
    }
  },
  placeholder: '请输入内容',
  readOnly: props.readOnly,
})

// 上传前校检格式和大小
function handleBeforeUpload(file) {
  const type = ["image/jpeg", "image/jpg", "image/png", "image/svg"];
  const isJPG = type.includes(file.type);
  //检验文件格式
  if (!isJPG) {
    proxy.$modal.msgError(`图片格式错误!`)
    return false
  }
  // 校检文件大小
  if (props.fileSize) {
    const isLt = file.size / 1024 / 1024 < props.fileSize
    if (!isLt) {
      proxy.$modal.msgError(`上传文件大小不能超过 ${props.fileSize} MB!`)
      return false
    }
  }
  return true
}
// 上传成功处理
function handleUploadSuccess(res, file) {
  // 如果上传成功
  if (res.code == 200) {
    // 获取富文本实例
    let quill = toRaw(myQuillEditor.value).getQuill();
    // 获取光标位置
    let length = quill.selection.savedRange.index;
    // 插入图片，res为服务器返回的图片链接地址
    let imageUrl = import.meta.env.VITE_APP_BASE_API + res.fileName
    quill.insertEmbed(length, "image", imageUrl);
    // 调整光标到最后
    quill.setSelection(length + 1);
    emit('getFileId', res.fileId)
  } else {
    proxy.$modal.msgError('图片插入失败')
  }

}
// 上传失败处理
function handleUploadError() {
  proxy.$modal.msgError('图片插入失败')
}
</script>
<style>
.editor,
.ql-toolbar {
  white-space: pre-wrap !important;
  line-height: normal !important;
}

.editor-img-uploader {
  display: none;
}

.ql-editor {
  min-height: 200px;
}

.ql-snow .ql-tooltip[data-mode='link']::before {
  content: '请输入链接地址:';
}

.ql-snow .ql-tooltip.ql-editing a.ql-action::after {
  border-right: 0px;
  content: '保存';
  padding-right: 0px;
}

.ql-snow .ql-tooltip[data-mode='video']::before {
  content: '请输入视频地址:';
}

.ql-snow .ql-picker.ql-size .ql-picker-label::before,
.ql-snow .ql-picker.ql-size .ql-picker-item::before {
  content: '14px';
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value='small']::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value='small']::before {
  content: '10px';
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value='large']::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value='large']::before {
  content: '18px';
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value='huge']::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value='huge']::before {
  content: '32px';
}

.ql-snow .ql-picker.ql-header .ql-picker-label::before,
.ql-snow .ql-picker.ql-header .ql-picker-item::before {
  content: '文本';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='1']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='1']::before {
  content: '标题1';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='2']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='2']::before {
  content: '标题2';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='3']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='3']::before {
  content: '标题3';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='4']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='4']::before {
  content: '标题4';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='5']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='5']::before {
  content: '标题5';
}

.ql-snow .ql-picker.ql-header .ql-picker-label[data-value='6']::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value='6']::before {
  content: '标题6';
}

.ql-snow .ql-picker.ql-font .ql-picker-label::before,
.ql-snow .ql-picker.ql-font .ql-picker-item::before {
  content: '标准字体';
}

.ql-snow .ql-picker.ql-font .ql-picker-label[data-value='serif']::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value='serif']::before {
  content: '衬线字体';
}

.ql-snow .ql-picker.ql-font .ql-picker-label[data-value='monospace']::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value='monospace']::before {
  content: '等宽字体';
}


</style>
