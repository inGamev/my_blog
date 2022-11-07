<template>
  <div class="app-container">
    <el-form
        :model="queryParams"
        ref="queryForm"
        :inline="true"
        v-show="showSearch"
        label-width="68px" >
      <el-form-item label="文件名称" prop="fileOriginName">
        <el-input
            v-model="queryParams.fileOriginName"
            placeholder="请输入文件名称"
            clearable
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="文件类型" prop="fileSuffix">
        <el-input
            v-model="queryParams.fileSuffix"
            placeholder="请输入文件类型，例如txt"
            clearable
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <!-- <el-form-item label="文件大小" prop="fileSizeInfo">
          <el-input
            v-model="queryParams.fileSizeInfo"
            placeholder="请输入文件大小"
            clearable
            @keyup.enter.native="handleQuery"
          />
        </el-form-item> -->
      <el-form-item label="存储名称" prop="fileObjectName">
        <el-input
            v-model="queryParams.fileObjectName"
            placeholder="请输入存储文件名称"
            clearable
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <!-- <el-form-item label="存储路径" prop="filePath">
          <el-input
            v-model="queryParams.filePath"
            placeholder="请输入存储路径"
            clearable
            @keyup.enter.native="handleQuery"
          />
        </el-form-item> -->
      <el-form-item>
        <el-button
            type="primary"
            icon="Search"
            @click="handleQuery">搜索
        </el-button >
        <el-button icon="Refresh" @click="resetQuery">重置</el-button >
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <!-- <el-col :span="1.5">
          <el-button
            type="primary"
            plain
            icon="Plus"

            @click="handleAdd"
            v-hasPermi="['cms:fileInfo:add']"
          >新增</el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="success"
            plain
            icon="Edit"

            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['cms:fileInfo:edit']"
          >修改</el-button>
        </el-col> -->
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="Upload"

            @click="uploadFile"
            v-hasPermi="['cms:fileInfo:add']"
        >上传文件
        </el-button >
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="danger"
            plain
            icon="Delete"
            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['cms:fileInfo:remove']"
        >删除
        </el-button  >
      </el-col>
      <!-- <el-col :span="1.5">
          <el-button
            type="warning"
            plain
            icon="Download"

            @click="handleExport"
            v-hasPermi="['cms:fileInfo:export']"
          >导出</el-button>
        </el-col> -->
      <right-toolbar
          v-model:showSearch="showSearch"
          @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
        v-loading="loading"
        :data="fileInfoList"
        @selection-change="handleSelectionChange"
        append-to-body
    >

      <el-table-column type="selection" width="55" align="center"/>
      <!-- <el-table-column label="文件主键id" align="center" prop="fileId" /> -->
      <el-table-column label="图片预览" align="center" prop="pic">
        <template v-slot="scope">
          <el-image
              style="width: 120px; height: 60px;"
              :src="scope.row.pic" lazy
              :preview-src-list="[scope.row.pic]"
              preview-teleported>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="文件名称" align="center" prop="fileOriginName"/>
      <el-table-column label="文件类型" align="center" prop="fileSuffix"/>
      <el-table-column label="文件大小" align="center" prop="fileSizeInfo"/>
      <el-table-column
          label="存储文件名称"
          align="center"
          prop="fileObjectName"
      />
      <!-- <el-table-column label="存储路径" align="center" prop="filePath" /> -->
      <el-table-column label="创建者" align="center" prop="createBy"/>
      <el-table-column
          label="创建时间"
          align="center"
          prop="createTime"
          width="100">
        <template v-slot="scope">
          <span>{{
              parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}')
            }}</span>
        </template>
      </el-table-column>
      <el-table-column
          label="操作"
          align="center"
          class-name="small-padding fixed-width" >
        <template v-slot="scope">
          <!-- <el-button

              type="text"
              icon="Edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['cms:fileInfo:edit']"
            >修改</el-button> -->
          <el-button
              type="text"
              icon="Download"
              @click="handleDownload(scope.row)" >下载
          </el-button  >
          <el-button
              type="text"
              icon="Delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:fileInfo:remove']" >删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
        v-show="total > 0"
        :total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="getList"
    />

    <!-- 添加或修改文件管理对话框 -->
    <!-- <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
        <el-form ref="form" :model="form" :rules="rules" label-width="80px">
          <el-form-item label="文件名称" prop="fileOriginName">
            <el-input v-model="form.fileOriginName" placeholder="请输入文件名称" />
          </el-form-item>
          <el-form-item label="文件类型" prop="fileSuffix">
            <el-input v-model="form.fileSuffix" placeholder="请输入文件类型，例如txt" />
          </el-form-item>
          <el-form-item label="文件大小" prop="fileSizeInfo">
            <el-input v-model="form.fileSizeInfo" placeholder="请输入文件大小" />
          </el-form-item>
          <el-form-item label="存储名称" prop="fileObjectName">
            <el-input v-model="form.fileObjectName" placeholder="请输入存储文件名称" />
          </el-form-item>
          <el-form-item label="存储路径" prop="filePath">
            <el-input v-model="form.filePath" placeholder="请输入存储路径" />
          </el-form-item>
          <el-form-item label="是否删除" prop="delFlag">
            <el-input v-model="form.delFlag" placeholder="请输入是否删除：Y-被删除，N-未删除" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </el-dialog> -->
    <el-dialog
        :title="title"
        v-model="open"
        width="500px"
        append-to-body>
      <el-upload
          style="min-height: 200px"
          ref="upload"
          :limit="5"
          accept=".jpg, .png,.txt,.doc,.docx,.xls,.xlsx,.ppt,.zip,.pdf"
          :action="upload.url"
          :headers="upload.headers"
          :file-list="upload.fileList"
          :on-progress="handleFileUploadProgress"
          :on-success="handleFileSuccess"
          :auto-upload="false">
        <template v-slot:trigger>
          <el-button size="small" type="primary">选取文件</el-button>
        </template>
        <el-button
            style="margin-left: 10px"
            size="small"
            type="success"
            :loading="upload.isUploading"
            @click="submitUpload"
        >上传到服务器
        </el-button
        >
        <template v-slot:tip>
          <div class="el-upload__tip">
            只能上传jpg/png/txt/doc/docx/xls/xlsx/ppt/zip/pdf文件，且单个文件不超过50Mb
          </div>
        </template>
      </el-upload>
    </el-dialog>
  </div>
</template>

<script setup>
import {
  listFileInfo,
  getFileInfo,
  delFileInfo,
  addFileInfo,
  updateFileInfo,
} from '@/api/cms/fileInfo'
import {getToken} from '@/utils/auth'
import image from './image'
import {getCurrentInstance, ref} from "vue";
import useUserStore from "@/store/modules/user";
import {useRouter} from "vue-router";

const {proxy} = getCurrentInstance();
const userStore = useUserStore()
const router = useRouter();

const loading = ref(true)
const ids = ref([])
const fileNames = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const fileInfoList = ref([])
const title = ref('')
const open = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  fileOriginName: null,
  fileSuffix: null,
  fileSizeInfo: null,
  fileObjectName: null,
  filePath: null,
  createBy: userStore.name,
})
const upload = ref({
  // 是否禁用上传
  isUploading: false,
  // 设置上传的请求头部
  headers: {Authorization: 'Bearer ' + getToken()},
  // 上传的地址
  url: import.meta.env.VITE_APP_BASE_API + '/common/upload',
  // 上传的文件列表
  fileList: [],
})
const form = ref({})
const rules = {
  fileOriginName: [
    {required: true, message: '文件名称不能为空', trigger: 'blur'},
  ],
  delFlag: [
    {
      required: true,
      message: '是否删除：Y-被删除，N-未删除不能为空',
      trigger: 'blur',
    },
  ],
}

// 文件下载处理
function handleDownload(row) {
  var name = row.fileOriginName
  var url = row.filePath
  var suffix = url.substring(url.lastIndexOf('.'), url.length)
  const a = document.createElement('a')
  a.setAttribute('download', name)
  a.setAttribute('target', '_blank')
  a.setAttribute('href', import.meta.env.VITE_APP_BASE_API + url)
  a.click()
}
/** 查询文件管理列表 */
function getList() {
  loading.value = true
  listFileInfo(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let fileInfo = response.rows[i]
      switch (fileInfo.fileSuffix) {
        case 'png':
        case 'jpg':
        case 'jpeg':
        case 'bmp':
        case 'gif':
          response.rows[i].pic =
              import.meta.env.VITE_APP_BASE_API + fileInfo.filePath
          break
        default:
          response.rows[i].pic = image.bg1
          break
      }
    }
    fileInfoList.value = response.rows
    total.value = response.total
    loading.value = false
  })
}
// 取消按钮
function cancel() {
  open.value = false
  reset()
}
// 表单重置
function reset() {
  form.value = {
    fileId: null,
    fileOriginName: null,
    fileSuffix: null,
    fileSizeInfo: null,
    fileObjectName: null,
    filePath: null,
    delFlag: null,
    createBy: userStore.name,
    createTime: null,
    updateBy: null,
    updateTime: null,
  }
  proxy.resetForm('form')
}
/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1
  getList()
}
/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm('queryForm')
  handleQuery()
}
// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map((item) => item.fileId)
  fileNames.value = selection.map((item) => item.fileOriginName)
  single.value = selection.length !== 1
  multiple.value = !selection.length
}
/** 上传文件按钮操作 */
function uploadFile() {
  reset()
  open.value = true
  title.value = '添加文件'
}
// 文件提交处理
function submitUpload() {
  proxy.$refs.upload.submit()
}
// 文件上传中处理
function handleFileUploadProgress(event, file, fileList) {
  upload.value.isUploading = true
}
// 文件上传成功处理
function handleFileSuccess(response, file, fileList) {
  upload.value.isUploading = false
  form.value.filePath = response.url
  proxy.$modal.msgSuccess(response.msg)
}
/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = '添加文件管理'
}
/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const fileId = row.fileId || ids.value
  getFileInfo(fileId).then((response) => {
    form.value = response.data
    open.value = true
    title.value = '修改文件管理'
  })
}
/** 提交按钮 */
function submitForm() {
  proxy.$refs['form'].validate((valid) => {
    if (valid) {
      if (form.value.fileId != null) {
        updateFileInfo(form.value).then((response) => {
          proxy.$modal.msgSuccess('修改成功')
          open.value = false
          getList()
        })
      } else {
        addFileInfo(form.value).then((response) => {
          proxy.$modal.msgSuccess('新增成功')
          open.value = false
          getList()
        })
      }
    }
  })
}
/** 删除按钮操作 */
function handleDelete(row) {
  const fileIds = row.fileId || ids.value
  let fileOriginName = row.fileOriginName || fileNames.value
  proxy.$modal
      .confirm('是否确认删除文件名称为"' + fileOriginName + '"的数据项？')
      .then(function () {
        return delFileInfo(fileIds)
      })
      .then(() => {
        getList()
        proxy.$modal.msgSuccess('删除成功')
      })
      .catch(() => {
      })
}
/** 导出按钮操作 */
function handleExport() {
  proxy.download(
      'cms/fileInfo/export',
      {
        ...queryParams.value,
      },
      `fileInfo_${new Date().getTime()}.xlsx`
  )
}

getList()


</script>
