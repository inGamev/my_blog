<template>
  <div class="app-container">
    <el-form
        :model="queryParams"
        ref="queryForm"
        :inline="true"
        v-show="showSearch"
        label-width="68px">
      <el-form-item label="标题" prop="title">
        <el-input
            v-model="queryParams.title"
            placeholder="请输入标题"
            clearable
            @keyup.enter="handleQuery" />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
            v-model="queryParams.status"
            placeholder="请选择状态"
            clearable  >
          <el-option
              v-for="dict in cms_blog_status"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"  />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
            type="primary"
            icon="Search"
            @click="handleQuery"
        >搜索
        </el-button
        >
        <el-button icon="Refresh" @click="resetQuery"
        >重置
        </el-button
        >
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="Plus"
            @click="handleAdd"
            v-hasPermi="['cms:blog:add']"
        >新增
        </el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="success"
            plain
            icon="Edit"

            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['cms:blog:edit']"
        >修改
        </el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="danger"
            plain
            icon="Delete"

            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['cms:blog:remove']"
        >删除
        </el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="warning"
            plain
            icon="Download"

            @click="handleExport"
            v-hasPermi="['cms:blog:export']"
        >导出
        </el-button
        >
      </el-col>
      <right-toolbar
          v-model:showSearch="showSearch"
          @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
        v-loading="loading"
        :data="blogList"
        @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center"/>
      <!-- <el-table-column label="ID" align="center" prop="id" /> -->
      <el-table-column label="标题" align="center" prop="title"/>
      <!-- <el-table-column label="内容" align="center" prop="content" /> -->
      <!-- <el-table-column label="置顶" align="center" prop="top" /> -->
      <!-- <el-table-column label="阅读量" align="center" prop="views" /> -->
      <el-table-column label="状态" align="center" prop="status">
        <template v-slot="scope">
          <dict-tag
              :options="cms_blog_status"
              :value="scope.row.status"
          />
        </template>
      </el-table-column>
      <el-table-column label="创建者" align="center" prop="createBy"/>
      <el-table-column
          label="创建时间"
          align="center"
          prop="createTime"
          width="100"
      >
        <template v-slot="scope">
          <span>{{
              parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}')
            }}</span>
        </template>
      </el-table-column>
      <el-table-column
          label="操作"
          align="center"
          class-name="small-padding fixed-width">
        <template v-slot="scope">
          <el-button
              type="text"
              icon="Edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['cms:blog:edit']">
            修改
          </el-button>
          <el-button
              type="text"
              icon="Delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:blog:remove']"
          >删除
          </el-button>
          <el-button
              type="text"
              icon="FolderOpened"
              @click="fileList(scope.row)"
              v-hasPermi="['system:notice:query']"
          >资源列表
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

    <!-- 添加或修改随笔管理对话框 -->
    <el-dialog
        :title="title"
        v-model="open"
        :before-close="cancel"
        width="1200px"
        append-to-body>
      <el-form ref="esform" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入标题"/>
        </el-form-item>
        <el-form-item label="内容" prop="content">
          <!-- 图片用base64存储,url方式移动端会显示异常 -->
          <vue-editor v-if="open" v-model:content="form.content" @getFileId="getFileId"/>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="releaseForm">发 布</el-button>
          <el-button type="info" @click="saveForm">暂 存</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 资源列表对话框 -->
    <el-dialog
        :title="title"
        v-model="fileListOpen"
        width="1000px"
        append-to-body>
      <el-table class="file-list" :data="fileInfoList">
        <el-table-column type="selection" width="55" align="center"/>
        <!-- <el-table-column label="文件主键id" align="center" prop="fileId" /> -->
        <el-table-column label="图片预览" align="center" prop="pic">
          <template v-slot="scope">
            <el-image
                style="width: 120px; height: 60px"
                :src="scope.row.pic"
                lazy
                :preview-src-list="[scope.row.pic]">
            </el-image>
          </template>
        </el-table-column>
        <el-table-column
            label="文件名称"
            align="center"
            prop="fileOriginName"/>
        <el-table-column label="文件类型" align="center" prop="fileSuffix"/>
        <el-table-column label="文件大小" align="center" prop="fileSizeInfo"/>
        <!-- <el-table-column label="存储文件名称" align="center" prop="fileObjectName" /> -->
        <!-- <el-table-column label="存储路径" align="center" prop="filePath" /> -->
        <!-- <el-table-column label="创建者" align="center" prop="createBy" /> -->
        <!-- <el-table-column label="创建时间" align="center" prop="createTime" width="100">
            <template slot-scope="scope">
              <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
            </template>
          </el-table-column> -->
        <el-table-column
            label="操作"
            align="center"
            class-name="small-padding fixed-width">
          <template v-slot="scope">
            <el-button
                type="text"
                icon="Download"
                @click="handleDownload(scope.row)">下载
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script setup>
import {listBlog, getBlog, delBlog, addBlog, updateBlog} from '@/api/cms/blog'
import {delFileInfo} from '@/api/cms/fileInfo'
import {
  addFileBlogInfo,
  delFileBlogInfo,
  getFileList,
} from '@/api/cms/fileBlogInfo'
import {ref} from "vue";

const {proxy} = getCurrentInstance();
const {cms_blog_status} = proxy.useDict("cms_blog_status");

const loading = ref(true)
const ids = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const blogList = ref([])
const fileInfoList = ref([])
const title = ref('')
const open = ref(false)
const fileListOpen = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  title: null,
  type: 2,
  content: null,
  top: null,
  views: null,
  status: null,
  createBy: null,
})
const form = ref({})
const top = ref(false)
const rules = {
  title: [
    {
      required: true,
      message: '标题不能为空',
      trigger: 'blur',
    },
  ],
  type: [
    {
      required: true,
      message: '类型不能为空',
      trigger: 'change',
    },
  ],
}
const fileIds = ref([])

/** 查询随笔管理列表 */
function getList() {
  loading.value = true
  listBlog(queryParams.value).then((response) => {
    blogList.value = response.rows
    total.value = response.total
    loading.value = false
  })
}

// 取消按钮
function cancel() {
  proxy.$modal.confirm('是否放弃此次编辑？')
      .then(() => {
        let fileids = fileIds.value
        if (fileids.length > 0) {
          delFileInfo(fileids)
        }
        fileIds.value.length = 0
        top.value = false
        open.value = false
        reset()
      })
      .catch(() => {
      })
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
    title: null,
    type: 2,
    content: null,
    top: '0',
    views: null,
    status: '0',
  }
  proxy.resetForm('esform')
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
  ids.value = selection.map((item) => item.id)
  single.value = selection.length !== 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = '添加随笔'
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const id = row.id || ids.value
  getBlog(id).then((response) => {
    form.value = response.data
    if (form.value.top == 1) {
      top.value = true
    }
    open.value = true
    title.value = '修改随笔'
  })
}

/** 发布按钮 */
function releaseForm() {
  proxy.$refs['esform'].validate((valid) => {
    if (valid) {
      form.value.type = 2
      form.value.status = 1
      if (top.value) {
        form.value.top = 1
      } else {
        form.value.top = 0
      }
      if (form.value.id != null) {
        updateBlog(form.value).then((response) => {
          if (fileIds.value.length > 0) {
            let fileBlogInfo = {
              blogId: form.value.id,
              fileIds: fileIds.value,
            }
            addFileBlogInfo(fileBlogInfo).then((response) => {
            })
          }
          proxy.$modal.msgSuccess('修改成功')
          fileIds.value.length = 0
          open.value = false
          getList()
        })
      } else {
        addBlog(form.value).then((response) => {
          if (fileIds.value.length > 0) {
            let fileBlogInfo = {
              blogId: response.data,
              fileIds: fileIds.value,
            }
            addFileBlogInfo(fileBlogInfo).then((response) => {
            })
          }
          proxy.$modal.msgSuccess('新增成功')
          fileIds.value.length = 0
          open.value = false
          getList()
        })
      }
    }
  })
}

/** 暂存按钮 */
function saveForm() {
  proxy.$refs['esform'].validate((valid) => {
    if (valid) {
      form.value.type = 2
      form.value.status = 0
      if (top.value) {
        form.value.top = 1
      } else {
        form.value.top = 0
      }
      if (form.value.id != null) {
        updateBlog(form.value).then((response) => {
          if (fileIds.value.length > 0) {
            let fileBlogInfo = {
              blogId: form.value.id,
              fileIds: fileIds.value,
            }
            addFileBlogInfo(fileBlogInfo).then((response) => {
            })
          }
          proxy.$modal.msgSuccess('修改成功')
          fileIds.value.length = 0
          open.value = false
          getList()
        })
      } else {
        addBlog(form.value).then((response) => {
          if (fileIds.value.length > 0) {
            let fileBlogInfo = {
              blogId: response.data,
              fileIds: fileIds.value,
            }
            addFileBlogInfo(fileBlogInfo).then((response) => {
            })
          }
          proxy.$modal.msgSuccess('新增成功')
          fileIds.value.length = 0
          open.value = false
          getList()
        })
      }
    }
  })
}

/** 删除按钮操作 */
function handleDelete(row) {
  const id = row.id || ids.value
  proxy.$modal
      .confirm('是否确认删除随笔管理编号为"' + id + '"的数据项？')
      .then(function () {
        delFileBlogInfo(id)
            .then()
            .then((response) => {
            })
        return delBlog(id)
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
      'cms/blog/export',
      {
        ...queryParams.value,
      },
      `blog_${new Date().getTime()}.xlsx`
  )
}


function getFileId(data) {
  fileIds.value.push(data)
}


/** 资源列表按钮操作 */
function fileList(row) {
  proxy.$modal.loading("加载中")
  reset()
  const blogId = row.id || ids.value
  getFileList(blogId).then((response) => {
    for (let i = 0; i < response.data.length; i++) {
      let fileInfo = response.data[i]
      switch (fileInfo.fileSuffix) {
        case 'png':
        case 'jpg':
        case 'jpeg':
        case 'bmp':
        case 'gif':
          response.data[i].pic =
              import.meta.env.VITE_APP_BASE_API + fileInfo.filePath
          break
        default:
          response.data[i].pic = image.bg1
          break
      }
    }
    fileInfoList.value = response.data
    fileListOpen.value = true
    title.value = '资源列表'
    proxy.$modal.closeLoading()
  })
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


getList()

</script>
