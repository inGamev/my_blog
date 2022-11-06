<template>
  <div class="component-upload-image">
    <el-upload
        :disabled="disabled"
        :action="uploadImgUrl"
        list-type="picture-card"
        :on-success="handleUploadSuccess"
        :before-upload="handleBeforeUpload"
        :limit="limit"
        :on-error="handleUploadError"
        name="file"
        :on-remove="handleRemove"
        :show-file-list="true"
        :headers="headers"
        :file-list="fileList"
        :class="{ hide: fileList.length >= limit }">
      <i class="el-icon-plus"></i>
    </el-upload>

    <!-- 上传提示 -->
    <template>
      <div class="el-upload__tip" v-if="showTip">
        请上传
        <template v-if="fileSize">
          大小不超过 <b style="color: #f56c6c">{{ fileSize }}MB</b>
        </template>
        <template v-if="fileType">
          格式为 <b style="color: #f56c6c">{{ fileType.join('/') }}</b>
        </template>
        的文件
      </div>
    </template>
  </div>
</template>

<script setup>

import {getToken} from "@/utils/auth";
import {computed, watch} from "vue";

const props = defineProps({
  modelValue: [String, Object, Array],
  // 大小限制(MB)
  fileSize: {
    type: Number,
    default: 10,
  },
  // 文件类型, 例如['png', 'jpg', 'jpeg']
  fileType: {
    type: Array,
    default: () => [
      'doc',
      'xls',
      'ppt',
      'pdf',
      'png',
      'jpg',
      'jpeg',
      'zip',
      '7z',
      'wps',
    ],
  },
  // 是否显示提示
  isShowTip: {
    type: Boolean,
    default: true,
  },
  // 是否不可编辑
  disabled: {
    type: Boolean,
    default: false,
  },
})
const {proxy} = getCurrentInstance();
const emit = defineEmits(['update:value', 'handleFilesSuccess'])

const limit = ref(1)
const hideUpload = ref(false)
const baseUrl = import.meta.env.VUE_APP_BASE_API
const uploadImgUrl = ref(import.meta.env.VUE_APP_BASE_API + '/common/upload')
const headers = ref({
  Authorization: 'Bearer ' + getToken(),
})
const fileList = ref([])
watch(() => props.modelValue, val => {
  if (val) {
    // 首先将值转为数组
    const list = Array.isArray(val) ? val : props.modelValue.split(',')
    // 然后将数组转为对象数组
    fileList.value = list.map((item) => {
      if (typeof item === 'string') {
        if (item.indexOf(baseUrl) === -1) {
          item = {name: baseUrl + item, url: '/file.png'}
        } else {
          item = {name: item, url: item}
        }
      }
      return item
    })
  } else {
    fileList.value = []
    return []
  }
}, {deep: true, immediate: true})


const showTip = computed(() => {
  return props.isShowTip && (props.fileType || props.fileSize)
})

// 删除图片
function handleRemove(file, fileList) {
  const findex = fileList.value.map((f) => f.name).indexOf(file.name)
  if (findex > -1) {
    fileList.value.splice(findex, 1)
    emit('update:value', listToString(fileList.value))
  }
}

// 上传成功回调
function handleUploadSuccess(res) {
  fileList.value.push({name: res.fileName, url: res.fileName})
  emit('update:value', listToString(fileList.value))
  const fileInfo = {
    fileId: listToString(fileList.value),
    fileOriginName: res.fileOriginName,
    fileSuffix: res.fileSuffix,
    fileSize: res.fileSize,
    filePath: res.fileName,
  }
  emit('handleFilesSuccess', fileInfo)
  proxy.$modal.closeLoading();
}

// 上传前loading加载
function handleBeforeUpload(file) {
  let isImg = false
  if (props.fileType.length) {
    let fileExtension = ''
    if (file.name.lastIndexOf('.') > -1) {
      fileExtension = file.name.slice(file.name.lastIndexOf('.') + 1)
    }
    isImg = props.fileType.some((type) => {
      if (file.type.indexOf(type) > -1) return true
      if (fileExtension && fileExtension.indexOf(type) > -1) return true
      return false
    })
  } else {
    isImg = file.type.indexOf('image') > -1
  }

  if (!isImg) {
    proxy.$modal.msgError(
        `文件格式不正确, 请上传${props.fileType.join('/')}格式文件!`
    )
    return false
  }
  if (props.fileSize) {
    const isLt = file.size / 1024 / 1024 < props.fileSize
    if (!isLt) {
      proxy.$modal.msgError(`上传文件大小不能超过 ${props.fileSize} MB!`)
      return false
    }
  }
  proxy.$modal.loading("正在上传文件，请稍候...");
}

// 上传失败
function handleUploadError() {
  proxy.$modal.msgError("上传文件失败");
  proxy.$modal.closeLoading();
}


// 对象转成指定字符串分隔
function listToString(list, separator) {
  let strs = ''
  separator = separator || ','
  for (let i in list) {
    strs += list[i].url.replace(baseUrl, '') + separator
  }
  return strs != '' ? strs.substr(0, strs.length - 1) : ''
}


</script>

<style lang="scss" scoped>
/*// .el-upload--picture-card 控制加号部分
*/
:deep( .el-upload--picture-card).hide {
  display: none;
}

/*// 去掉动画效果
*/
:deep(.el-list-enter-active),
:deep(.el-list-leave-active) {
  transition: all 0s;
}

:deep(.el-list-enter),
.el-list-leave-active {
  opacity: 0;
  transform: translateY(0);
}
</style>
