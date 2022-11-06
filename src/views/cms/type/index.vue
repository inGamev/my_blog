<template>
  <div class="app-container">
    <el-form
        :model="queryParams"
        ref="queryForm"
        :inline="true"
        v-show="showSearch"
        label-width="68px">
      <el-form-item label="分类名称" prop="typeName">
        <el-input
            v-model:value="queryParams.typeName"
            placeholder="请输入分类名称"
            clearable
            size="small"
            @keyup.enter="handleQuery"
        />
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
            v-hasPermi="['cms:type:add']"
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
            v-hasPermi="['cms:type:edit']"
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
            v-hasPermi="['cms:type:remove']"
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
            v-hasPermi="['cms:type:export']"
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
        :data="typeList"
        @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center"/>
      <!-- <el-table-column label="分类ID" align="center" prop="typeId" /> -->
      <el-table-column
          label="分类图像"
          align="center"
          prop="typePic"
          width="100">
        <template v-slot="scope">
          <el-image
              style="
              width: 28px;
              height: 28px;
              border-radius: 50%;
              margin-right: 10px;
            "
              :src="scope.row.typePic"
              lazy
              :preview-src-list="[scope.row.typePic]"
          >
            <template v-slot:error>
              <div class="image-slot">
                <i class="el-icon-collection"></i>
              </div>
            </template>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="分类名称" align="center" prop="typeName"/>
      <el-table-column label="博客数量" align="center" prop="blogNum"/>
      <el-table-column label="创建者" align="center" prop="createBy"/>
      <el-table-column
          label="创建时间"
          align="center"
          prop="createTime"
          width="100"
      >
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
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
              v-hasPermi="['cms:type:edit']"
          >修改
          </el-button
          >
          <el-button

              type="text"
              icon="el-icon-delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:type:remove']"
          >删除
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

    <!-- 添加或修改分类管理对话框 -->
    <el-dialog
        :title="title"
        v-model:visible="open"
        :before-close="cancel"
        width="500px"
        append-to-body
    >
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="分类名称" prop="typeName">
          <el-input
              v-model:value="form.typeName"
              placeholder="请输入分类名称"
          />
        </el-form-item>
        <el-form-item label="分类图像">
          <imageUpload v-model:value="form.typePic" :limit="1"/>
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
import {
  listType,
  getType,
  delType,
  addType,
  updateType,
  cancelType,
} from '@/api/cms/type'

const {proxy} = getCurrentInstance();

const loading = ref(true)
const ids = ref([])
const names = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const typeList = ref([])
const title = ref('')
const open = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  typeName: null,
  typePic: null,
  createBy: null,
})
const form = ref({})
const rules = {
  typeName: [
    {
      required: true,
      message: '分类名称不能为空',
      trigger: 'blur',
    },
  ],
}

/** 查询分类管理列表 */
function getList() {
  loading.value = true
  listType(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let typeInfo = response.rows[i]
      if (typeInfo.typePic.length > 0) {
        response.rows[i].typePic = import.meta.env.VUE_APP_BASE_API + typeInfo.typePic
      }
    }
    typeList.value = response.rows
    total.value = response.total
    loading.value = false
  })
}

function cancel() {
  cancelType(form.value).then((response) => {
    open.value = false
    reset()
  })
}

function reset() {
  form.value = {
    typeId: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
    typeName: null,
    typePic: null,
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
  ids.value = selection.map((item) => item.typeId)
  names.value = selection.map((item) => item.typeName)
  single.value = selection.length !== 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = '添加分类管理'
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const typeId = row.typeId || ids.value
  getType(typeId).then((response) => {
    form.value = response.data
    open.value = true
    title.value = '修改分类管理'
  })
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs['form'].validate((valid) => {
    if (valid) {
      if (form.value.typeId != null) {
        updateType(form.value).then((response) => {
          proxy.$modal.msgSuccess('修改成功')
          open.value = false
          getList()
        })
      } else {
        addType(form.value).then((response) => {
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
  const typeIds = row.typeId || ids.value
  let name = row.typeName || names.value
  proxy.$modal
      .confirm('是否确认删除"' + name + '"？')
      .then(function () {
        return delType(typeIds)
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
      'cms/type/export',
      {
        ...queryParams.value,
      },
      `type_${new Date().getTime()}.xlsx`
  )
}


getList()


</script>
