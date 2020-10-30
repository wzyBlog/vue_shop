<template>
  <el-container class="home-contanier">
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="" />
        <span>电商管理网站</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409EFF"
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          :router="true"
          :default-active="active_path"
        >
          <el-submenu :index="v.id + ''" v-for="(v, k) in menulist" :key="v.id">
            <template slot="title">
              <i :class="iconlist[k]"></i>
              <span>{{ v.authName }}</span>
            </template>
            <el-menu-item
              :index="'/' + subItem.path"
              v-for="subItem in v.children"
              :key="subItem.id"
              @click="saveNavStatus('/' + subItem.path)"
            >
              <i class="el-icon-menu"></i>
              <span>{{ subItem.authName }}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main><router-view></router-view></el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data() {
    return {
      menulist: [],
      iconlist: [
        'iconfont icon-user',
        'iconfont icon-tijikongjian',
        'iconfont icon-shangpin',
        'iconfont icon-danju',
        'iconfont icon-baobiao'
      ],
      isCollapse: false,
      active_path: ''
    }
  },
  created() {
    this.getMenuList()
    this.active_path = window.sessionStorage.getItem('active_path')
  },
  methods: {
    // 退出只需清空sessionStorage，即销毁token
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取左侧菜单数据
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavStatus(path) {
      window.sessionStorage.active_path = path
      this.active_path = path
    }
  }
}
</script>
<style lang="less" scoped>
.home-contanier {
  height: 100%;
  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    > div {
      display: flex;
      align-items: center;
      span {
        margin-left: 10px;
        color: #fff;
        font-size: 20px;
      }
    }
  }
  .el-aside {
    width: 200px;
    background-color: #333744;
    .toggle-button {
      text-align: center;
      color: #fff;
      letter-spacing: 0.2em;
      background-color: gray;
      font-size: 14px;
      line-height: 24px;
      cursor: pointer;
    }
    .iconfont {
      margin-right: 10px;
    }
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: #eaedf1;
  }
}
</style>
