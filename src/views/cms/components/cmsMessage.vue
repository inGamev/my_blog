<template>
  <el-row :gutter="20">
    <el-col :sm="5" class="hidden-xs-only" style="opacity: 0">左侧占位</el-col>
    <el-col :xs="24" :sm="14">
      <el-container style="opacity: 0.9" class="message">
        <el-card class="animate__animated animate__fadeInLeft publish">
          <div class="author">
            <el-avatar
                v-if="userStore.token == null"
                icon="el-icon-user-solid"
                size="large">
              <!-- style="background-color: #666" -->
            </el-avatar>
            <el-avatar v-else :src="userStore.avatar" size="large"></el-avatar>
            <div>
              <div class="nkname">
                <span class="name" v-if="userStore.token == null">匿名用户</span>
                <span class="name" v-else>{{ userStore.name }} </span>
              </div>
            </div>
          </div>
          <el-form
              :model="messageForm"
              :rules="messageFormRules"
              ref="messageFormRef"
          >
            <el-form-item prop="content">
              <el-input
                  :rows="5"
                  v-model:value="messageForm.content"
                  type="textarea"
                  maxlength="100"
                  show-word-limit
                  placeholder="请输入你的留言"
              ></el-input>
            </el-form-item>
            <el-form-item style="text-align: right">
              <el-button type="primary" @click="publish">点击发表</el-button>
            </el-form-item>
          </el-form>
        </el-card>

        <el-card
            v-if="messageList.length > 0"
            class="animate__animated animate__fadeInLeft"
        >
          <comment
              :comments="messageList"
              @replyConfirm="commitComment"
          ></comment>
          <pagination
              v-show="total > 0"
              :total="total"
              v-model:page="queryParams.pageNum"
              v-model:limit="queryParams.pageSize"
              @pagination="getMessageList"
          />
        </el-card>
      </el-container>
    </el-col>
    <el-col :sm="5" class="hidden-xs-only" style="opacity: 0">右侧占位</el-col>
    <!-- 设置底部距离的 -->
    <el-backtop :bottom="60">
      <div style="
           {
            height: 50px;
            width: 50px;
            background-color: rgba(240, 239, 241, 1);
            box-shadow: 0 0 6px rgba(0, 0, 0, 0.12);
            text-align: center;
            line-height: 40px;
            border-radius: 2px;
            color: #1989fa;
          }
        ">
        <svg-icon icon-class="top"/>
      </div>
    </el-backtop>
  </el-row>
</template>

<script setup>

import {cmsListMessage, cmsAddMessage} from '@/api/cms/message'
import comment from './messages/messages.vue'
import useUserStore from "@/store/modules/user";
import {nextTick, onUpdated} from "vue";

const userStore = useUserStore()
const router = useRouter();
const {proxy} = getCurrentInstance();

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
  blogId: null,
  userId: null,
  delFlag: null,
  createBy: null,
})
const messageFormRules = {
  content: [
    {
      min: 0,
      max: 100,
      message: '留言内容不超过100字！',
    },
  ],
}

onUpdated(
    nextTick(() => {
      to()
    })
)

// 表单重置
function reset() {
  messageForm.value = {
    id: null,
    parentId: null,
    mainId: null,
    likeNum: null,
    content: null,
    type: null,
    blogId: null,
    userId: null,
    delFlag: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
  }
  proxy.resetForm('messageForm')
}

// 留言发表
function publish() {
  proxy.$refs['messageFormRef'].validate(async (valid) => {
    if (!valid) return
    if (messageForm.value.content == null || messageForm.value.content == '') {
      proxy.$modal.msgError('留言内容不能为空！')
      return
    }
    if (userStore.token == null || userStore.token == '') {
      messageForm.value.createBy = '匿名用户'
      messageForm.value.type = '0'
    } else {
      messageForm.value.createBy = userStore.name
      messageForm.value.type = '0'
    }
    cmsAddMessage(messageForm.value).then((response) => {
      proxy.$modal.msgSuccess('留言发表成功')
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
  this.$refs['messageFormRef'].validate(async (valid) => {
    if (!valid) return
    if (
        messageForm.value.content == null ||
        messageForm.value.content == ''
    ) {
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
    cmsAddMessage(messageForm.value).then((response) => {
      proxy.$modal.msgSuccess('评论发表成功')
      reset()
      getMessageList()
    })
  })
}

// 获取留言列表
async function getMessageList() {
  let token = userStore.token
  if (token != null && token != '') {
    queryParams.value.createBy = userStore.name
  }
  cmsListMessage(queryParams.value).then((response) => {
    for (let i = 0; i < response.rows.length; i++) {
      let mesInfo = response.rows[i]
      if (mesInfo.avatar != null && mesInfo.avatar != '') {
        response.rows[i].avatar =
            import.meta.env.VUE_APP_BASE_API + mesInfo.avatar
      }
      if (mesInfo.children != null && mesInfo.children != '') {
        for (let j = 0; j < response.rows[i].children.length; j++) {
          let children = response.rows[i].children
          if (children.avatar != null && children.avatar != '') {
            response.rows[i].children[j].avatar =
                import.meta.env.VUE_APP_BASE_API + children.avatar
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
  if (router.currentRoute.value.id != null) {
    var toEl = document.getElementById(router.currentRoute.value.id)
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

.publish {
  margin-bottom: 20px;
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
