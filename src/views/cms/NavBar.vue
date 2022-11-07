<template>
  <el-header :style="'margin-bottom:' + headerBottom + 'px'">
    <h2 class="logo">
      <!--      <svg-icon icon-class="EarOfWheat"/>-->
      inGamev
    </h2>


    <div class="bg-purple-light">
      <el-menu
          :default-active="activeIndex"
          router
          ellipsis
          class="el-menu-demo"
          mode="horizontal"
          style="border: none"
          background-color="rgba(0,0,0,0)"
          text-color="#fff"
          active-text-color="#ffd04b">
        <el-menu-item index="/cms/main/cmsIndex">
          首页
        </el-menu-item>
        <el-menu-item
            :index="item.path"
            v-for="item in menuList"
            :key="item.id">
          {{ item.menuName }}
        </el-menu-item>
      </el-menu>
    </div>
    <!--小屏菜单-->
    <div class="bg-purple-light el-menu-hidden" v-if="menuHiddenVisiable">
      <el-menu
          :default-active="activeIndex"
          router
          background-color="rgba(84,92,100,0.5)"
          text-color="#fff"
          active-text-color="#ffd04b">
        <el-menu-item index="/cms/main/cmsIndex" @click="menuAway">首页</el-menu-item>
        <el-menu-item
            :index="item.path"
            v-for="item in menuList"
            :key="item.id"
            @click="menuAway">
          {{ item.menuName }}
        </el-menu-item>
      </el-menu>
    </div>


    <div class="menu-expend" @click="menuExpend">
      <el-icon style="color: rgba(255, 255, 255)">
        <Menu/>
      </el-icon>
    </div>

    <div v-if="searchInput" class="search_input">
      <el-input
          @focus="checkInput"
          @blur="notSearching()"
          class="search"
          placeholder="搜索博客"
          prefix-icon="Search"
          v-model="queryInfo.queryKey">
      </el-input>
      <ul v-if="searching">
        <li class="animate__animated animate__fadeInDown search-blog"
            v-for="blog in searchList"
            :key="blog.id"
            @click="getBlogInfo(blog.id)">
          <a><span v-html="blog.title"></span></a>
        </li>
      </ul>
    </div>

    <!--    <div v-if="isLogin" class="bg-purple">-->
    <!--      <el-dropdown class="avatar-container right-menu-item hover-effect" trigger="click">-->
    <!--        <div class="avatar-wrapper">-->
    <!--          <el-avatar class="user-avatar" :src="userStore.avatar" @error="errorHandler">-->
    <!--            <i class="el-icon-s-custom"/>-->
    <!--          </el-avatar>-->
    <!--          <p class="avatar-Name">{{ userStore.name }}</p>-->
    <!--        </div>-->
    <!--        <template v-slot:dropdown>-->
    <!--          <el-dropdown-menu>-->
    <!--            <router-link target="_blank" to="/index">-->
    <!--              <el-dropdown-item>管理博客</el-dropdown-item>-->
    <!--            </router-link>-->
    <!--            <el-dropdown-item divided @click="logout">-->
    <!--              <span>退出登录</span>-->
    <!--            </el-dropdown-item>-->
    <!--          </el-dropdown-menu>-->
    <!--        </template>-->
    <!--      </el-dropdown>-->
    <!--    </div>-->
    <!--    &lt;!&ndash;开放登录注册&ndash;&gt;-->
    <!--    <div v-else class="bg-purple">-->
    <!--      <div class="avatar-wrapper">-->
    <!--        <p class="avatar-Name" @click.prevent="toLogin">登录 | 注册</p>-->
    <!--      </div>-->
    <!--    </div>-->
  </el-header>
</template>

<script setup>

import {getToken} from '@/utils/auth'
import {cmsListBlog} from '@/api/cms/blog'
import useUserStore from "@/store/modules/user";
import {ElMessageBox} from "element-plus";
import {watch} from "vue";

const router = useRouter();
const userStore = useUserStore()

const activeIndex = ref('router.path')
const isLogin = ref(false)
const searchInput = ref(true)
const menuHiddenVisiable = ref(false)
const headerBottom = ref(0)

const queryInfo = ref({
  queryKey: "",
})

const searchList = ref([])
const searching = ref(false)
const menuList = [
  {
    id: 1,
    menuName: '时间轴',
    path: '/cms/main/essay',
    icon: 'el-icon-edit',
  },
  // {
  //   id: 2,
  //   menuName: '留言',
  //   path: '/cms/main/message',
  //   icon: 'el-icon-chat-dot-round',
  // },
  // {
  //   id: 3,
  //   menuName: '文档',
  //   path: '/cms/doucument',
  //   icon: 'el-icon-document',
  // },
]

// 查询参数
const queryParams = ref({
  pageNum: 1,
  pageSize: 10,
  title: null,
  type: 1,
  content: null,
  top: null,
  views: null,
  status: null,
})


function menuListAdd() {
  //push()方法一般是添加到数组的最后的位置；unshift()方法是往最前面的位置添加。
  // this.menulist.push({id:"",menuName:""})
  menuList.value.unshift({
    id: '',
    menuName: '',
  })
}

//响应式布局
function ResponsiveLayout() {
  //浏览器窗口的内部高度
  var w = window.innerWidth || document.body.clientWidth
  //浏览器窗口的内部宽度
  var h = window.innerHeight || document.body.clientHeight
  console.log(w, h)
}

// 展开菜单栏
function menuExpend() {
  menuHiddenVisiable.value = !menuHiddenVisiable.value
  if (menuHiddenVisiable.value === true) {
    headerBottom.value = (menuList.length + 1) * 56
  } else {
    headerBottom.value = 0
  }
}


//收起菜单
function menuAway() {
  menuHiddenVisiable.value = false
  headerBottom.value = 0
}


function notSearching() {
  setTimeout(() => {
    searching.value = false
  }, 100)
}

function checkInput() {
  searching.value = queryInfo.value.queryKey !== ''
}

function toLogin() {
  router.push("/cmsLogin");
}

function checkLogin() {
  isLogin.value = getToken();
}


function errorHandler() {
  return true
}


async function searchBlog() {
  if (queryInfo.value.queryKey === '') {
    searching.value = false
    return
  }
  queryParams.value.title = queryInfo.value.queryKey
  cmsListBlog(queryParams.value).then((response) => {
    let listSize = response.rows.length
    if (listSize > 0) {
      for (let i = 0; i < listSize; i++) {
        let redTitle = brightenKeyword(
            response.rows[i].title,
            queryInfo.value.queryKey
        )
        response.rows[i].title = redTitle
      }
    }
    searchList.value = response.rows
    if (searchList.value.length !== 0) {
      searching.value = true
    }
  })
}

//搜索关建字标红
function brightenKeyword(val, keyword) {
  const Reg = new RegExp(keyword, 'i')
  let res = ''
  if (val) {
    res = val.replace(Reg, `<span style="color: red;">${keyword}</span>`)
    return res
  }
}

// 跳转到博客详情页
function getBlogInfo(blogId) {
  let routeUrl = router.resolve({
    path: '/cms/main/blog',
    query: {
      id: blogId,
    }
  })
  window.open(routeUrl.href, '_blank')
}

function logout() {
  ElMessageBox.confirm('确定注销并退出系统吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    userStore.logOut().then(() => {
      location.href = '/cms/main/cmsIndex'
    })
  }).catch(() => {
  });
}

watch(
    () => queryInfo.value.queryKey,
    (value) => {
      searchBlog()
    }
)

checkLogin()

</script>

<style scoped>
.el-header {
  display: flex;
  justify-content: space-between;
  background-color: rgba(0, 0, 0, 0.2);
  align-items: center;
  transition: 0.2s;
}

.el-header:hover {
  opacity: 1 !important;
}

.el-menu {
  flex-shrink: 0;
  background-color: rgba(0, 0, 0, 0) !important;
}

.el-menu :deep .el-menu-item {
  background-color: rgba(0, 0, 0, 0) !important;
}

.el-menu :deep .el-menu-item i {
  color: rgba(0, 0, 0, 0);
}

.el-menu :deep .el-menu-item:hover i {
  color: rgba(0, 0, 0, 0);
}

.el-menu :deep .el-menu-item:hover {
  background-color: rgba(0, 0, 0, 0.5) !important;
}

.search_input {
  position: relative;
  box-sizing: border-box;
}

.search_input ul {
  position: absolute;
  top: 30px;
  width: 100%;
  border: 1px solid #e5e9ef;
  margin-top: 1px;
  background: #fff;
  z-index: 10000;
  border-radius: 2px;
  box-shadow: 0 2px 4px rgba(58, 118, 142, 0.16);
  padding: 10px 0;
  font-size: 14px;
  box-sizing: border-box;
}

.search_input ul li {
  padding: 0 16px;
  height: 32px;
  line-height: 32px;
  cursor: pointer;
  word-wrap: break-word;
  word-break: break-all;
  display: block;
  color: #222;
  position: relative;
}

.search_input ul li:hover {
  background-color: #e8f3ff;
}

.search-blog {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  padding-left: 20px;
  padding-right: 20px;
}

.bg-purple-light {
  width: 100%;
  align-content: center;
  float: right;
}

.bg-purple {
  float: right;
}

.user-avatar {
  float: left;
  cursor: pointer;
  /*border: dashed rgba($ color: #f6f683, $ alpha: 0.5);*/
}

.avatar-container {
  margin-right: 30px;
}

.avatar-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.avatar-Name {
  margin-left: 10px;
  cursor: pointer;
  float: right;
  font-size: 16px;
  color: #ffffff;
}

.logo {
  float: left;
  color: #ffd04b;
  font-weight: bold;
}

.logo:hover {
  cursor: pointer;
}

.el-menu-hidden {
  position: absolute;
  top: 60px;
  left: 0;
  border-top: 1px solid #ccc;
  border-right: none;
  width: 100%;
}

.menu-expend {
  display: none !important;
}

@media screen and (max-width: 780px) {
  .search_input {
    display: none;
  }
}

@media screen and (max-width: 768px) {
  .el-menu :deep .el-menu-item {
    background-color: rgba(0, 0, 0, 0.3) !important;
  }

  .el-menu-demo {
    display: none;
  }

  .menu-expend {
    display: block !important;
    float: right;
  }

  .menu-expend:hover {
    color: #ffd04b;
    cursor: pointer;
  }
}
</style>
