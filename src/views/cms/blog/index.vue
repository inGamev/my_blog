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
            v-model:value="queryParams.title"
            placeholder="请输入标题"
            clearable
            size="small"
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
            v-model:value="queryParams.status"
            placeholder="请选择状态"
            clearable
            size="small"
        >
          <el-option
              v-for="dict in cms_blog_status"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
            type="primary"
            icon="el-icon-search"

            @click="handleQuery"
        >搜索
        </el-button
        >
        <el-button icon="el-icon-refresh" @click="resetQuery"
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
            icon="el-icon-plus"
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
            icon="el-icon-edit"

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
            icon="el-icon-delete"

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
            icon="el-icon-download"

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
        :row-class-name="tableRowClassName"
    >
      <el-table-column type="selection" width="55" align="center"/>
      <!-- <el-table-column label="ID" align="center" prop="id" /> -->
      <el-table-column label="首图预览" align="center" prop="blogPic">
        <template v-slot="scope">
          <el-image
              style="width: 120px; height: 60px"
              :src="scope.row.blogPic"
              lazy
              :preview-src-list="[scope.row.blogPic]"
          >
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="标题" align="center" prop="title"/>
      <!-- <el-table-column label="内容" align="center" prop="content" /> -->
      <!-- <el-table-column label="置顶" align="center" prop="top" /> -->
      <el-table-column label="分类" align="center" prop="types">
        <template v-slot="scope">
          <el-tag

              v-for="tag in scope.row.types"
              :key="tag.typeId"
              type="info"
          >{{ tag.typeName }}
          </el-tag
          >
        </template>
      </el-table-column>
      <el-table-column label="标签" align="center" prop="tags">
        <template v-slot="scope">
          <el-tag
              effect="plain"

              v-for="tag in scope.row.tags"
              :key="tag.tagId"
              type="success">
            {{ tag.tagName }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="阅读量" align="center" prop="views"/>
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
          class-name="small-padding fixed-width"
      >
        <template v-slot="scope">
          <el-button

              type="text"
              icon="el-icon-edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['cms:blog:edit']"
          >修改
          </el-button
          >
          <el-button

              type="text"
              icon="el-icon-delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:blog:remove']"
          >删除
          </el-button
          >
          <el-button

              type="text"
              icon="el-icon-folder-opened"
              @click="blogFiles(scope.row)"
              v-hasPermi="['cms:blog:edit']"
          >附件管理
          </el-button
          >
          <el-button

              type="text"
              icon="el-icon-folder-opened"
              @click="fileList(scope.row)"
              v-hasPermi="['cms:blog:edit']"
          >资源列表
          </el-button
          >
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

    <!-- 添加或修改文章管理对话框 -->
    <el-dialog
        :title="title"
        v-model="open"
        :before-close="cancel"
        width="1200px"
        append-to-body
    >
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标题" prop="title">
          <el-input v-model:value="form.title" placeholder="请输入标题"/>
        </el-form-item>
        <el-row>
          <el-col :span="8">
            <el-form-item label="首图">
              <imageUpload v-model:value="form.blogPic" :limit="1"/>
            </el-form-item>
          </el-col>
          <el-col :span="16">
            <el-form-item label="简介">
              <el-input
                  type="textarea"
                  v-model:value="form.blogDesc"
                  :autosize="{ minRows: 7, maxRows: 7 }"
                  maxlength="50"
                  show-word-limit
                  placeholder="请输入简介"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="内容">
          <!-- 图片用base64存储,url方式移动端会显示异常 -->
          <cmsEditor
              v-model:value="form.content"
              @getFileId="getFileId"
              type="base64"
              :min-height="192"
          />
        </el-form-item>
        <el-form-item label="标签">
          <el-checkbox-group v-model:value="form.tagIds">
            <el-checkbox
                v-for="item in tagOptions"
                :label="item.tagId"
                :key="item.tagId"
                :value="item.tagId"
            >
              {{ item.tagName }}
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-row>
          <el-col :span="7">
            <el-form-item label="分类">
              <el-select
                  v-model:value="form.typeIds"
                  multiple
                  placeholder="请选择"
                  filterable
                  clearable
              >
                <el-option
                    v-for="item in typeOptions"
                    :key="item.typeId"
                    :label="item.typeName"
                    :value="item.typeId"
                >
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="17">
            <el-form-item label="置顶">
              <el-checkbox v-model:value="top"></el-checkbox>
            </el-form-item>
          </el-col>
        </el-row>
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
            prop="fileOriginName"
        />
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
            class-name="small-padding fixed-width"
        >
          <template v-slot="scope">
            <el-button
                type="text"
                icon="el-icon-download"
                @click="handleDownload(scope.row)">下载
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>

    <!-- 附件管理对话框 -->
    <el-dialog
        title="附件管理"
        v-model="blogFilesOpen"
        :before-close="cancel"
        width="1200px"
        append-to-body>
      <el-form
          ref="form"
          :model="form"
          :rules="rules"
          label-position="top"
          label-width="80px">
        <el-form-item>
          <el-row>
            <el-button type="primary" @click="addFiles"
            >添加文件
            </el-button
            >
            <el-table
                :data="form.blogFilesNew"
                :border="true"
                style="width: 99.99%">
              <el-table-column
                  align="center"
                  min-width="20%"
                  prop="pic"
                  label="附件">
                <template v-slot="scope">
                  <!--                  <filesUpload-->
                  <!--                      v-model="scope.row.fileId"-->
                  <!--                      @handleFilesSuccess="filesSuccess"-->
                  <!--                      :is-show-tip="false"-->
                  <!--                  />-->
                </template>
              </el-table-column>
              <el-table-column
                  align="center"
                  min-width="20%"
                  prop="remark"
                  label="文件信息">
                <template v-slot="scope">
                  <el-row>
                    <el-col :span="6">
                      <div class="blogFilesInfoName">名称：</div>
                    </el-col>
                    <el-col :span="18">
                      <el-input v-model:value="scope.row.fileOriginName" disabled/>
                    </el-col>
                  </el-row>
                  <el-row style="margin-top: 4px">
                    <el-col :span="6">
                      <div class="blogFilesInfoName">大小：</div>
                    </el-col>
                    <el-col :span="18">
                      <el-input v-model:value="scope.row.fileSize" disabled/>
                    </el-col>
                  </el-row>
                  <el-row style="margin-top: 4px">
                    <el-col :span="6">
                      <div class="blogFilesInfoName">类型：</div>
                    </el-col>
                    <el-col :span="18">
                      <el-input v-model:value="scope.row.fileSuffix" disabled/>
                    </el-col>
                  </el-row>
                </template>
              </el-table-column>
              <el-table-column
                  align="center"
                  min-width="40%"
                  prop="remark"
                  label="备注">
                <template v-slot="scope">
                  <el-input
                      v-model:value="scope.row.remark"
                      type="textarea"
                      :rows="6"
                      size="small"
                  />
                </template>
              </el-table-column>
              <el-table-column align="center" min-width="20%" label="操作">
                <template v-slot="scope">
                  <el-button
                      v-show="scope.row.fileId !== ''"
                      plain
                      @click="handleDownload(scope.row)"
                  >下载
                  </el-button>
                  <el-button
                      type="danger"
                      plain
                      @click="delFiles(scope.$index, scope.row)"
                  >删除
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-row>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="saveBlogFiles">保 存</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>


  </div>
</template>

<script setup>
import FilesUpload from './components/filesUpload'
import {
  listBlog,
  getBlog,
  delBlog,
  addBlog,
  updateBlog,
  cancelBlog,
} from '@/api/cms/blog'
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
const names = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const blogList = ref([])
const fileInfoList = ref([])
const title = ref('')
const open = ref(false)
const fileListOpen = ref(false)
const blogFilesOpen = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  title: null,
  type: 1,
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
const typeOptions = ref([])
const tagOptions = ref([])

// Maximum recursive updates exceeded in component <ElDialogContent>. This means you have a reactive effect that is mutating
// its own dependencies and thus recursively triggering itself. Possible sources include component template, render function,
// updated hook or watcher source function.
/** 查询文章管理列表 */
function getList() {
  loading.value = true
  listBlog(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let blogInfo = response.rows[i]
      if (blogInfo.blogPic.length > 0) {
        response.rows[i].blogPic =
            import.meta.env.VUE_APP_BASE_API + blogInfo.blogPic
      } else {
        response.rows[i].blogPic = '/errorImg.jpg'
      }
    }
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
        cancelBlog(form.value).then((response) => {
        })
        top.value = false
        open.value = false
        blogFilesOpen.value = false
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
    type: 1,
    content: null,
    top: '0',
    views: null,
    status: '0',
    blogDesc: null,
    blogFiles: null,
    blogPic: null,
    tagIds: [],
    typeIds: [],
    blogFilesNew: [],
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
  ids.value = selection.map((item) => item.id)
  names.value = selection.map((item) => item.title)
  single.value = selection.length !== 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  getBlog().then((response) => {
    typeOptions.value = response.types
    tagOptions.value = response.tags
    reset()
    open.value = true
    title.value = '添加文章'
  })
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const id = row.id || ids.value
  getBlog(id).then((response) => {
    typeOptions.value = response.types
    tagOptions.value = response.tags
    form.value = response.data
    if (form.value.top == 1) {
      top.value = true
    }
    open.value = true
    title.value = '修改文章'
  })
}

/** 发布按钮 */
function releaseForm() {
  proxy.$refs['form'].validate((valid) => {
    if (valid) {
      form.value.type = 1
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
  proxy.$refs['form'].validate((valid) => {
    if (valid) {
      form.value.type = 1
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
  let name = row.title || names.value
  proxy.$modal
      .confirm('是否确认删除"' + name + '"？')
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
              import.meta.env.VUE_APP_BASE_API + fileInfo.filePath
          break
        default:
          response.data[i].pic = image.bg1
          break
      }
    }
    fileInfoList.value = response.data
    fileListOpen.value = true
    title.value = '资源列表'
    setTimeout(() => {
      proxy.$modal.closeLoading()
    }, 100)
  })
}


/** 附件管理按钮操作 */
function blogFiles(row) {
  reset()
  const id = row.id || ids.value
  getBlog(id).then((response) => {
    typeOptions.value = response.types
    tagOptions.value = response.tags
    form.value = response.data
    form.value.blogFilesNew = []
    if (response.data.blogFiles !== null) {
      form.value.blogFilesNew = JSON.parse(response.data.blogFiles)
    }
    if (form.value.top == 1) {
      top.value = true
    }
    blogFilesOpen.value = true
  })
}


// 附件管理添加按钮
function addFiles() {
  form.value.blogFilesNew.push({
    id: uuid(),
    fileId: '',
    fileOriginName: '',
    fileSuffix: '',
    fileSize: '',
    filePath: '',
    remark: '',
  })
}


function delFiles(index, row) {
  proxy.$modal.confirm('确认删除吗?')
      .then(() => {
        // 点击确定的操作(调用接口)
        var hasmembers = form.value.blogFilesNew
        for (var i = 0; i < hasmembers.length; i++) {
          if (row.id === hasmembers[i].id) {
            form.value.blogFilesNew.splice(i, 1)
          }
        }
      })
      .catch(() => {
        // 点取消的提示
        return
      })
}


// 生成uuid
function uuid() {
  var d = new Date().getTime()
  if (window.performance && typeof window.performance.now === 'function') {
    d += performance.now() // use high-precision timer if available
  }
  var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(
      /[xy]/g,
      function (c) {
        var r = (d + Math.random() * 16) % 16 | 0 // d是随机种子
        d = Math.floor(d / 16)
        return (c === 'x' ? r : (r & 0x3) | 0x8).toString(16)
      }
  )
  return uuid
}


function filesSuccess(value) {
  form.value.blogFilesNew.forEach((item) => {
    if (item.fileId === value.fileId) {
      item.fileOriginName = value.fileOriginName
      item.fileSuffix = value.fileSuffix
      item.fileSize = value.fileSize
      item.filePath = value.filePath
    }
  })
}


//保存文件
function saveBlogFiles() {
  if (form.value.blogFilesNew.length > 0) {
    for (let i = 0; i < form.value.blogFilesNew.length; i++) {
      const fileInfo = form.value.blogFilesNew[i]
      if (fileInfo.fileId === '') {
        proxy.$message.warning('请添加文件或删除空行！')
        return false
      } else if (fileInfo.remark === '') {
        proxy.$message.warning('请添加文件备注！')
        return false
      }
    }
  }
  form.value.blogFiles = JSON.stringify(form.value.blogFilesNew)
  updateBlog(form.value).then((response) => {
    proxy.$modal.msgSuccess('保存成功')
    blogFilesOpen.value = false
    getList()
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
  a.setAttribute('href', import.meta.env.VUE_APP_BASE_API + url)
  a.click()
}

function tableRowClassName({row, rowIndex}) {
  if (row.top == 1) {
    return 'warning-row'
  }
  return ''
}

getList()


</script>

<style lang="scss" scoped>
.el-tag + .el-tag {
  margin-left: 10px;
}

.el-table .warning-row {
  background: #f8f8f9;
}

.blogFilesInfoName {
  text-align: center;
  padding-top: 5px;
}
</style>
