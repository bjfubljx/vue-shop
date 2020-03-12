<template>
  <div>
    <el-container class="home-container">
      <!-- 头部区域 -->
      <el-header>
        <div>
          <img src="../assets/heima.png" alt />
          <span>电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <!-- 页面主体区域 -->
      <el-container>
        <el-aside :width="isCollapse?'64px':'200px'">
          <!-- 侧边栏菜单区 -->
          <div class = "toggle-button" @click="toggleCollapse">|||</div>
          <el-menu background-color="#333744" text-color="#fff" 
          active-text-color="#409EFF" unique-opened :collapse ="isCollapse" 
          :collapse-transition = "false" router :default-active="activePath">
            <!-- 一级菜单 -->
            <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
              <!-- 一级菜单模板区 -->
              <template slot="title">
                <i :class="iconsObj[item.id]"></i>
                <span>{{item.authName}}</span>
              </template>
              <!-- 二级菜单 -->
              <el-menu-item
                v-for="subItem in item.children"
                :index="'/'+subItem.path"
                :key="subItem.id"
                @click="saveNavState('/'+subItem.path)"
              >
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>{{subItem.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>

        <el-main>
            <!-- 路由占位符 -->
            <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      menulist: [],
      iconsObj: {
        "125": "el-icon-user-solid",
        "103": "el-icon-s-management",
        "101": "el-icon-s-goods",
        "102": "el-icon-document",
        "145": "el-icon-data-analysis"
      },
      //是否折叠
      isCollapse : false,
    //   被激活的链接地址
      activePath : ''
    };
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem('activePath');
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    async getMenuList() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.menulist = res.data;
    //   console.log(res);
    },
    // 菜单栏折叠与展开
    toggleCollapse () {
        this.isCollapse = !this.isCollapse;
    },
    saveNavState(activePath){
        window.sessionStorage.setItem('activePath',activePath);
    }
  }
};
</script>

<style lang="less" scoped>
.home-container {
  height: 100vh;
}
.el-header {
  padding-left: 0%;
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
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
  .el-menu{
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.toggle-button{
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>