<template>
  <div class="app-container">
    <el-form
        :model="queryParams"
        ref="queryForm"
        :inline="true"
        v-show="showSearch"
        label-width="68px">
      <el-form-item label="标签名称" prop="tagName">
        <el-input
            v-model="queryParams.tagName"
            placeholder="请输入标签名称"
            clearable
            @keyup.enter="handleQuery"  />
      </el-form-item>
      <el-form-item>
        <el-button
            type="primary"
            icon="Search"
            @click="handleQuery"
        >搜索
        </el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="Plus"
            @click="handleAdd"
            v-hasPermi="['cms:tag:add']"
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
            v-hasPermi="['cms:tag:edit']"
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
            v-hasPermi="['cms:tag:remove']"
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
            v-hasPermi="['cms:tag:export']"
        >导出
        </el-button
        >
      </el-col>
      <right-toolbar
          v-model:showSearch="showSearch"
          @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table  v-loading="loading" :data="tagList"  @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center"/>
      <!-- <el-table-column label="标签ID" align="center" prop="tagId" /> -->
      <el-table-column label="标签名称" align="center" prop="tagName"/>
      <el-table-column label="博客数量" align="center" prop="blogNum"/>
      <el-table-column label="创建者" align="center" prop="createBy"/>
      <el-table-column
          label="创建时间"
          align="center"
          prop="createTime"
          width="100" >
        <template v-slot="scope">
          <span>
            {{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}
          </span>
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
              v-hasPermi="['cms:tag:edit']">修改
          </el-button>
          <el-button
              type="text"
              icon="Delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:tag:remove']">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
        v-show="total > 0"
        :total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="getList"/>

    <!-- 添加或修改标签管理对话框 -->
    <el-dialog
        :title="title"
        v-model="open"
        width="500px"
        :before-close="cancel"
        append-to-body>
      <el-form ref="tagForm" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标签名称" prop="tagName">
          <el-input v-model="form.tagName" placeholder="请输入标签名称"/>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {listTag, getTag, delTag, addTag, updateTag} from '@/api/cms/tag'
import {getCurrentInstance, ref} from "vue";

const {proxy} = getCurrentInstance();

const loading = ref(false)
const ids = ref([])
const names = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const tagList = ref([])
const title = ref('')
const open = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  tagName: null,
  createBy: null,
})
const form = ref({})
const rules = {
  tagName: [
    {required: true, message: '标签名称不能为空', trigger: 'blur'},
  ],
}

/** 查询标签管理列表 */
function getList() {
  loading.value = true
  listTag(queryParams.value).then((response) => {
    tagList.value = response.rows
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
    tagId: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
    tagName: null,
  }
  proxy.resetForm('tagForm')
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
  ids.value = selection.map((item) => item.tagId)
  names.value = selection.map((item) => item.tagName)
  single.value = selection.length !== 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = '添加标签管理'
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const tagId = row.tagId || ids.value
  getTag(tagId).then((response) => {
    form.value = response.data
    open.value = true
    title.value = '修改标签管理'
  })
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs['tagForm'].validate((valid) => {
    if (valid) {
      if (form.value.tagId != null) {
        updateTag(form.value).then((response) => {
          proxy.$modal.msgSuccess('修改成功')
          open.value = false
          getList()
        })
      } else {
        addTag(form.value).then((response) => {
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
  const tagIds = row.tagId || ids.value
  let name = row.tagName || names.value
  proxy.$modal
      .confirm('是否确认删除标签"' + name + '"？')
      .then(function () {
        return delTag(tagIds)
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
      'cms/tag/export',
      {
        ...queryParams.value,
      },
      `tag_${new Date().getTime()}.xlsx`
  )
}

getList()

</script>
