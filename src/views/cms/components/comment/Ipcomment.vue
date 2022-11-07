<template>
  <el-container style="opacity: 0.9">
    <div class="author">
      <el-avatar v-if="userStore.token" icon="el-icon-user-solid" size="large">
        <!-- style="background-color: #666" -->
      </el-avatar>
      <el-avatar v-else :src="avatar" size="large"></el-avatar>
      <div>
        <div class="nkname">
          <span class="name" v-if="userStore.token">匿名用户</span>
          <span class="name" v-else>{{ userStore.name }} </span>
        </div>
      </div>
    </div>
    <el-form
        :model="messageForm"
        :rules="messageFormRules"
        ref="messageFormRef">
      <el-form-item prop="content">
        <el-input
            :rows="5"
            v-model="messageForm.content"
            type="textarea"
            maxlength="100"
            show-word-limit
            placeholder="请输入你的评论"
        ></el-input>
      </el-form-item>
      <el-form-item style="text-align: right">
        <el-button type="primary" @click="publish">发表评论</el-button>
      </el-form-item>
    </el-form>
    <el-divider v-if="messageList.length > 0"><span style="color: #999; font-size: small">最新评论</span></el-divider>
    <comments :comments="messageList" @replyConfirm="commitComment"></comments>
    <pagination
        v-show="total > 0"
        :total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="getMessageList"
    />
  </el-container>
</template>

<script setup>
import {cmsListComment, cmsAddComment} from '@/api/cms/comment'
import useUserStore from "@/store/modules/user";
import {nextTick, onUpdated} from "vue";
import Comments from "@/views/cms/components/comment/comments";


const {proxy} = getCurrentInstance();
const router = useRouter();
const userStore = useUserStore()
const picList = ref([])
const editing = ref(false)
const messageList = ref([])
const message = ref({
  userId: -1,
  content: '',
})
const messageForm = ref({})
const total = ref(0)
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  parentId: null,
  mainId: null,
  likeNum: null,
  content: null,
  type: null,
  blogId: router.currentRoute.value.id,
  userId: null,
  delFlag: null,
  createBy: null,
})
const messageFormRules = ref({
  content: [
    {
      min: 0,
      max: 100,
      message: '评论内容不超过100字！',
    },
  ],
})


onUpdated(
    nextTick(() => to())
)

function reset() {
  messageForm.value = {
    id: null,
    parentId: null,
    mainId: null,
    likeNum: null,
    content: null,
    type: null,
    blogId: router.currentRoute.value.id,
    userId: null,
    delFlag: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
  }
  proxy.resetForm('messageForm')
}

function publish() {
  proxy.$refs['messageFormRef'].validate(async (valid) => {
    if (!valid) return
    if (messageForm.value.content == null || messageForm.value.content == '') {
      proxy.$modal.msgError('评论内容不能为空！')
      return
    }
    if (useUserStore.token == null || useUserStore.token == '') {
      messageForm.value.createBy = '匿名用户'
      messageForm.value.type = '0'
    } else {
      messageForm.value.createBy = userStore.name
      messageForm.value.type = '0'
    }
    cmsAddComment(messageForm.value).then((response) => {
      proxy.$modal.msgSuccess('评论发表成功')
      reset()
      getMessageList()
    })
  })
}

/**
 * 提交评论
 */
function commitComment(value) {
  reset()
  messageForm.value.content = value.inputComment
  messageForm.value.parentId = value.id

  proxy.$refs['messageFormRef'].validate(async (valid) => {
    if (!valid) return
    if (messageForm.value.content == null || messageForm.value.content == '') {
      proxy.$modal.msgError('评论内容不能为空！')
      return
    }
    if (userStore.token == null || userStore.token == '') {
      messageForm.value.createBy = '匿名用户'
      messageForm.value.type = '1'
    } else {
      messageForm.value.createBy = userStore.name
      messageForm.value.type = '1'
    }
    cmsAddComment(messageForm.value).then((response) => {
      proxy.$modal.msgSuccess('评论发表成功')
      reset()
      getMessageList()
    })
  })
}

// 获取评论列表
async function getMessageList() {
  if (userStore.token != null && userStore.token != '') {
    queryParams.value.createBy = userStore.name
  }
  cmsListComment(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let mesInfo = response.rows[i]
      if (mesInfo.avatar != null && mesInfo.avatar != '') {
        response.rows[i].avatar = import.meta.env.VITE_APP_BASE_API + mesInfo.avatar
      }
      if (mesInfo.children != null && mesInfo.children != '') {
        for (let j = 0; j < response.rows[i].children.length; j++) {
          let children = response.rows[i].children
          if (children.avatar != null && children.avatar != '') {
            response.rows[i].children[j].avatar =
                import.meta.env.VITE_APP_BASE_API + children.avatar
          }
        }
      }
    }
    messageList.value = response.rows
    total.value = response.total
  })
}


//跳转到相应位置
function to() {
  if (router.currentRoute.value.commentId != null) {
    let toEl = document.getElementById(router.currentRoute.value.commentId);
    if (toEl != null) {
      if (toEl != null && toEl != '') {
        // toEl 为指定跳转到该位置的DOM节点
        let bridgeCms = toEl
        let bodyTop = document.body
        let heightCms = 0
        // 计算该 DOM 节点到 bodyTop 顶部距离
        do {
          heightCms += bridgeCms.offsetTop
          bridgeCms = bridgeCms.offsetParent
        } while (bridgeCms !== bodyTop)
        // 滚动到指定位置
        window.scrollTo({
          top: heightCms,
          behavior: 'smooth',
        })
      }
    }
  }
}

getMessageList()
reset()

</script>

<style scoped>
.el-container {
  display: block;
}

.author {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  width: 100%;
  margin-bottom: 20px;
}

.comment {
  border-bottom: 1px dashed #ccc;
  margin: 30px 0;
  display: flex;
}

.content {
  text-align: left;
  font-size: 14px;
  flex-grow: 1;
}

.nkname {
  margin: 10px;
  max-width: 530px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.date {
  color: #999;
  margin-left: 10px;
}

.reply {
  margin-left: 10px;
}
</style>
