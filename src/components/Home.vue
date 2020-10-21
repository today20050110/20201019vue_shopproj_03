<template>
  <!-- 頭部區域 -->
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>電商後台管理系統</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 頁面主體區域 -->
    <el-container>
      <!-- 側邊攔 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 側邊攔菜單區域 -->
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409eff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath">
          <!-- 1級菜單 -->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <!-- 1級菜單模板區域 -->
            <template slot="title">
              <!-- 圖標 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二級菜單 -->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <template slot="title">
              <!-- 圖標 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右側內容主體 -->
      <el-main>
        <!-- 路由佔位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      // 左側菜單數據
      menulist: [],
      iconsObj: {
        // eslint-disable-next-line quote-props
        '125': 'iconfont icon-user',
        // eslint-disable-next-line quote-props
        '103': 'iconfont icon-tijikongjian',
        // eslint-disable-next-line quote-props
        '101': 'iconfont icon-shangpin',
        // eslint-disable-next-line quote-props
        '102': 'iconfont icon-danju',
        // eslint-disable-next-line quote-props
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      // 被激活的鏈接地址
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 獲取所有菜單
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meat.msg)
      this.menulist = res.data
      // console.log(res)
    },
    // 點擊按鈕,切換菜單的折疊與展開
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    // 保存練街的激活狀態
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
