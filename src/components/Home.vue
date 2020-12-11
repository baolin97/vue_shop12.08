<template>
<el-container class="home-container">
  <!-- 头部 -->
  <el-header height="80px">
   <div>
     <h1 class="fl"><img src="../assets/img/login/logo.png" alt=""></h1>
     <p class="fl">云商城后台管理系统</p>
  </div>
    <div >
      <el-button type="info" @click="logout">退出</el-button>
    </div>
  </el-header>

<!-- 主体区 -->
  <el-container>
   
    <!-- 侧边栏 -->
    <el-aside :width=" isCollapse ? '64px': '240px'">
       <!-- 显示隐藏 -->
    <div class="toggle-button" @click="toggleCollapse">
          <i class="el-icon-s-unfold"></i>
    </div>
      <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409bff"
      :unique-opened="true"
      :collapse="isCollapse"
      :collapse-transition="false"
      :router="true"
      :default-active="activePath"
            >
      <!-- 一级菜单 -->
      <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
        <!-- 一级菜单模板区 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconsObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
        </template>

       <!-- 二级菜单 -->
        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" 
        :key="subItem.id" @click="saveNavState('/'+subItem.path)">
          <template slot="title">
           <i class="el-icon-s-grid"></i>
          <!-- 文本 -->
            <span>{{subItem.authName}}</span>
          </template>
        </el-menu-item>
      </el-submenu>
          
    
     
    </el-menu>
    </el-aside>

    <!-- 右侧内容 -->
    <el-main>
      <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
    // 生命周期函数
    created () {
      this.getMenuList(),
      this.activePath =window.sessionStorage.getItem('activePath')
    },
  data(){
      return{
        // 菜单数据
        menulist:[],
        iconsObj:{
          '125':'iconfont icon-user-group-fill',
          '103':'iconfont icon-databaseset-fill',
          '101':'iconfont icon-shopping-cart-fill',
          '102':'iconfont icon-database-fill',
          '145':'iconfont icon-chart-line',
        },
        // 是否折叠
        isCollapse:false,
        // 激活的链接地址
        activePath:''
      }
  },
  methods:{
    // 退出
      logout(){
          window.sessionStorage.clear();
          this.$router.push('/login')
        //   alert("退出")
      },
      // 获取菜单
    async getMenuList(){
        const {data:res} = await this.$http.get('menus');
        //  console.log(res);
        if(res.meta.status !== 200) return this.$message.error(res.meta.msg);
       this.menulist =res.data;
        // console.log(res.data);
      },
      // 隐藏菜单效果
      toggleCollapse(){
             this.isCollapse=!this.isCollapse;
      },
      // 保存连接激活状态
      saveNavState(activePath){
          window.sessionStorage.setItem('activePath' ,activePath)
          this.activePath=activePath;
      },

  }
    
}
</script>

<style lang="less" scoped>
*{
  margin: 0;
  padding:0;
}

.home-container{
  height:100%;
}
.el-menu{
    border-right:none;
}

// 头部样式
  .el-header{
    display: flex;
    justify-content: space-between;
    color:white;
    background-color: #373d41;
    align-items: center; 
  }
 
 .el-header>div img{
    width: 3.75rem;
    height: 3.75rem;
    margin: .625rem;
}

 .el-header>div>p{
   display: flex;
   align-items: center;
    color: #fff;
    font-size:1.75rem;
    line-height:5rem;
    font-weight: bold; 
}

  // 侧边栏样式
.el-aside{
  background-color: #333744;
}

.iconfont{
  margin-right: .75rem;
  color: #EAEDF1;
  font-size: 1.125rem;
}
.toggle-button{
   height:1.875rem;
   line-height:1.875rem;
   background-color: #4a5064;
   color:#fff;
   text-align: center;
   cursor: pointer;
}

  // 右侧样式
.el-main{
  background-color: #EAEDF1;
}
</style>