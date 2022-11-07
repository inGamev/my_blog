<template>
  <div class="app-container">
    <el-form
        :model="queryParams"
        ref="queryForm"
        :inline="true"
        v-show="showSearch"
        label-width="68px"
        style="padding-left: 20px">
      <el-form-item label="留言者" prop="content" v-if="isAdmin">
        <el-input
            v-model="queryParams.createBy"
            placeholder="请输入留言者"
            clearable
            @keyup.enter="handleQuery"/>
      </el-form-item>
      <el-form-item label="留言内容" prop="content">
        <el-input
            v-model="queryParams.content"
            placeholder="请输入留言内容"
            clearable
            @keyup.enter="handleQuery"/>
      </el-form-item>
      <el-form-item>
        <el-button
            type="primary"
            icon="Search"
            @click="handleQuery">搜索
        </el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- <el-row :gutter="10" class="mb8">
        <el-col :span="1.5">
          <el-button
            type="primary"
            plain
            icon="Plus"

            @click="handleAdd"
            v-hasPermi="['cms:message:add']"
          >新增</el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="success"
            plain
            icon="Edit"

            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['cms:message:edit']"
          >修改</el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="danger"
            plain
            icon="Delete"

            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['cms:message:remove']"
          >删除</el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="warning"
            plain
            icon="Download"

            @click="handleExport"
            v-hasPermi="['cms:message:export']"
          >导出</el-button>
        </el-col>
        <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
      </el-row>

      <el-table v-loading="loading" :data="messageList" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="ID" align="center" prop="id" />
        <el-table-column label="父留言id" align="center" prop="parentId" />
        <el-table-column label="点赞数量" align="center" prop="likeNum" />
        <el-table-column label="内容" align="center" prop="content" />
        <el-table-column label="留言类型：对人评论，对项目评论，对资源评论" align="center" prop="type" />
        <el-table-column label="被留言者id，可以是人、项目、资源" align="center" prop="blogId" />
        <el-table-column label="留言者id" align="center" prop="userId" />
        <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
          <template slot-scope="scope">
            <el-button

              type="text"
              icon="Edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['cms:message:edit']"
            >修改</el-button>
            <el-button

              type="text"
              icon="Delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['cms:message:remove']"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table> -->
    <el-card>
      <div class="el-card__header" style="text-align: left; padding: 0">
        <h3 style="float: left; margin: 0">留言列表</h3>
        <right-toolbar
            style="float: right"
            v-model:showSearch="showSearch"
            @queryTable="getList"
        ></right-toolbar>
      </div>
      <ul style="padding: 0" class="comment-list">
        <li v-show="messageList.length == 0"
            style="
            text-align: center;
            border-bottom: 1px dashed #ccc;
            margin: 10px 0;
            list-style-type: none;
          ">
          <span class="el-table__empty-text">暂无数据</span>
        </li>
        <li class="comment" v-for="mes in messageList" :key="mes.id">
          <el-avatar
              v-if="mes.avatar !== '' && mes.avatar != null"
              :src="mes.avatar"
          ></el-avatar>
          <el-avatar v-else icon="el-icon-user-solid"></el-avatar>
          <div class="content">
            <div style="display: flex; justify-content: space-between; width: 100%">
              <div class="nkname">
                <span class="name">{{ mes.createBy }} </span>
                <span class="date">{{ mes.createTime }}</span>
                <span v-show="mes.type == '0'" class="rp">给你留言</span>
                <span v-show="mes.type == '1'" class="rp">回复了</span>
                <span v-show="mes.type == '1'" class="pcreateBy">{{
                    mes.pcreateBy
                  }}</span>
                <span v-show="mes.type == '1'" class="rp">的留言</span>
              </div>
              <div class="op">
                <span @click="getBlogInfo(mes.id)" class="blog">查看</span>
                <span> | </span>
                <el-button
                    type="text"
                    @click="handleAdd(mes)"
                    v-hasPermi="['cms:message:add']" >回复
                </el-button >
                <span
                    v-show="!isAdmin && mes.createBy != userStore.name"
                    style="margin-right: 39.43px" ></span>
                <span v-show="!(!isAdmin && mes.createBy != userStore.name)"> | </span>
                <span
                    v-show="!(!isAdmin && mes.createBy != userStore.name)"
                    class="del"
                    @click="handleDelete(mes)">删除</span>
              </div>
            </div>
            <p class="reply">{{ mes.content }}</p>
          </div>
        </li>
      </ul>
    </el-card>

    <pagination
        v-show="total > 0"
        :total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="getList"/>
    <!-- 添加或修改留言管理对话框 -->
    <el-dialog
        :title="title"
        v-model="open"
        width="500px"
        append-to-body>
      <el-form ref="msgBackform" :model="form" :rules="rules" label-width="80px">
        <!-- <el-form-item label="父留言id" prop="parentId">
            <el-input v-model="form.parentId" placeholder="请输入父留言id" />
          </el-form-item>
          <el-form-item label="点赞数量" prop="likeNum">
            <el-input v-model="form.likeNum" placeholder="请输入点赞数量" />
          </el-form-item> -->
<!--         <el-form-item  prop="content">-->
          <el-input
            v-model="form.content"
            type="textarea"
            maxlength="100"
            show-word-limit
            :placeholder="toName" />
<!--         </el-form-item>-->
        <!-- <el-form-item label="被留言者id，可以是人、项目、资源" prop="blogId">
            <el-input v-model="form.blogId" placeholder="请输入被留言者id，可以是人、项目、资源" />
          </el-form-item>
          <el-form-item label="留言者id" prop="userId">
            <el-input v-model="form.userId" placeholder="请输入留言者id" />
          </el-form-item> -->
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
  listMessage,
  getMessage,
  delMessage,
  addMessage,
  updateMessage,
} from '@/api/cms/message'
import useUserStore from "@/store/modules/user";
import {getCurrentInstance, nextTick, onMounted, ref} from "vue";
import {useRouter} from "vue-router";

const {proxy} = getCurrentInstance();
const userStore = useUserStore()
const router = useRouter();

const loading = ref(true)
const ids = ref([])
const names = ref([])
const single = ref(true)
const multiple = ref(true)
const showSearch = ref(true)
const total = ref(0)
const messageList = ref([])
const title = ref('')
const open = ref(false)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  parentId: null,
  likeNum: null,
  content: null,
  type: null,
  blogId: null,
  userId: null,
  delFlag: null,
  createBy: null,
})
const form = ref({})
const rules = {}
const toName = ref('')
const isAdmin = ref(false)

onMounted(
    nextTick(() => {
      //仅在整个视图都被渲染之后才会运行的代码
      getList()
      isAdminRole()
    })
)

/** 查询留言管理列表 */
function getList() {
  proxy.$modal.loading("加载中")
  listMessage(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let mesInfo = response.rows[i]
      if (mesInfo.avatar != null && mesInfo.avatar != '') {
        response.rows[i].avatar = import.meta.env.VITE_APP_BASE_API + mesInfo.avatar
      }
    }
    messageList.value = response.rows
    total.value = response.total
  proxy.$modal.closeLoading()
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
    id: null,
    parentId: null,
    mainId: null,
    likeNum: null,
    content: null,
    type: null,
    blogId: null,
    userId: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
  }
  proxy.resetForm('msgBackform')
}


/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1
  getList()
}

/** 重置按钮操作 */
function resetQuery() {
  queryParams.value.createBy = ''
  queryParams.value.content = ''
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
function handleAdd(mes) {
  reset()
  if (mes.mainId != null) {
    form.value.mainId = mes.mainId
  } else {
    form.value.mainId = mes.id
  }
  form.value.parentId = mes.id
  form.value.type = '1'
  form.value.createBy = userStore.name
  toName.value = '@' + mes.createBy
  open.value = true
  title.value = '回复留言'
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const id = row.id || ids.value
  getMessage(id).then((response) => {
    form.value = response.data
    open.value = true
    title.value = '修改留言管理'
  })
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs['msgBackform'].validate((valid) => {
    if (valid) {
      if (form.value.id != null) {
        updateMessage(form.value).then((response) => {
          proxy.$modal.msgSuccess('修改成功')
          open.value = false
          getList()
        })
      } else {
        addMessage(form.value).then((response) => {
          proxy.$modal.msgSuccess('回复成功')
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
  let name = row.content || names.value
  proxy.$modal
      .confirm('是否确认删除 "' + name + '" ？')
      .then(function () {
        return delMessage(id)
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
      'cms/message/export',
      {
        ...queryParams.value,
      },
      `message_${new Date().getTime()}.xlsx`
  )
}

// 跳转到留言页
function getBlogInfo(blogId) {
  let routeUrl = router.resolve({
    path: '/cms/main/message',
    query: {
      id: blogId,
    },
  })
  window.open(routeUrl.href, '_blank')
}

function isAdminRole() {
  // 验证用户是否具备某角色
  if (proxy.$auth.hasRole('admin') || proxy.$auth.hasRole('cms')) {
    isAdmin.value = true
  }
}


</script>

<style scoped>
.comment {
  border-bottom: 1px dashed #ccc;
  margin: 10px 0;
  display: flex;
}

.el-avatar {
  width: 35px;
  height: 35px;
  box-shadow: 0 0 10px 2px rgba(0, 0, 0, 0.06);
  flex-shrink: 0;
}

.content {
  text-align: left;
  font-size: 14px;
  flex-grow: 1;
}

.nkname {
  margin-left: 10px;
  max-width: 530px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.rp,
.date {
  color: #999;
  margin-left: 10px;
}

.pcreateBy {
  margin-left: 10px;
}

.blog {
  color: #349edf;
  margin-left: 10px;
  cursor: pointer;
}

.reply {
  margin-left: 10px;
}

.op {
  color: #ddd;
  font-weight: lighter;
}

.rep {
  color: #349edf;
}

.del {
  color: red;
}

.op:hover {
  cursor: pointer;
}

.el-table__empty-text {
  line-height: 60px;
  width: 50%;
  color: #909399;
}
</style>
