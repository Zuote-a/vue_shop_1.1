<template>
  <el-container class="home-container">
    <!--头部-->
    <el-header>
      <div class="header-life">
        <img src="../assets/logo/OIP.png" alt style="width: 100px;height: 60px" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--主题区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <!--侧边栏菜单区-->
        <div class="toggle-button" @click="toggleCollp">|||</div>
        <el-menu background-color="#323743" text-color="#fff" active-text-color="#409FFF" unique-opened
        :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id +''" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children"
            :key="subItem.id" @click="saveNavState('/'+subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!--      右侧类容-->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'Home',
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data () {
    return {
      isCollapse: false,
      iconsObj: {
        125: 'iconfont icon-users',

        103: 'iconfont icon-tijikongjian',

        101: 'iconfont icon-shangpin',

        102: 'iconfont icon-shangpin',

        145: 'iconfont icon-shangpin'
      },
      menulist: [],
      // 保存激活的连接地址
      activePath: ''
    }
  },
  methods: {
    // 保存连接的激活状态解决刷新后导航失效的问题
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    },
    // 菜单折叠与展开
    toggleCollp () {
      this.isCollapse = !this.isCollapse
    },
    // 获取所有菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(res)
    },
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped>
.home-container {
  width: 100%;
  height: 100%;
}
.el-header {
  background-color: #373d3f;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #ffffff;
  font-size: 20px;
}
.header-life {
  display: flex;
  align-items: center;
}
.header-life span {
  margin-left: 15px;
}
.el-aside {
  background-color: #323743;
}
.el-main {
  background-color: #e9edf0;
}
.iconfont {
  padding-right: 20px;
}
.el-menu {
  border-right: none;
}
.toggle-button {
  background-color: #4A5065;
  font-size: 10px;
  line-height: 24px;
  text-align: center;
  color: #fff;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
