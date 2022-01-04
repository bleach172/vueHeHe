<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <h1>vue</h1>
      <el-button
        @click="logout"
        type="info"
      >退出</el-button>
    </el-header>
    <!-- 页面主题区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div
          class="toggle-button"
          @click="toggleCollapse"
        >|||</div>
        <el-menu
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          :router="true"
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id+''"
            v-for="item in menuList"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item
              :index="'/'+subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNavState('/'+subItem.path)"
            >
              <i class="el-icon-location"></i>
              <span>{{subItem.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容主体区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        125: 'iconfont icon-yonghuguanli',
        103: 'iconfont icon-quanxianguanli',
        101: 'iconfont icon-shangchengxuanzhong',
        102: 'iconfont icon-dingdanguanli',
        145: 'iconfont icon-shujutongji'
      },
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created () {
    // 执行获取菜单数据的函数
    this.getMenuList()
    // 获取会话存储空间中的激活链接地址
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(res)
    },
    // 点击按钮控制侧边栏折叠
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #b3c0d1;
  color: #333;
  line-height: 60px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.el-aside {
  background-color: #545c64;
  color: #333;
  line-height: 200px;
}
.el-main {
  background-color: #e9eef3;
  color: #333;
  line-height: 160px;
}
.el-menu {
  border-right: none;
}
.toggle-button {
  height: 40px;
  line-height: 40px;
  text-align: center;
  color: white;
  letter-spacing: 4px;
  font-size: 10px;
  cursor: pointer;
}
.iconfont {
  margin-right: 12px;
}
</style>
