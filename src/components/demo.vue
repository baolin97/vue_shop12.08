<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
       </el-breadcrumb> 
          <!-- 卡片视图区 -->
          
           <el-card class="box-card">
           </el-card>

    </div>
</template>

<script>
export default {
  // 生命周期函数
    created(){},
    // 数据
    data(){
        return{

        }
    },
    // 方法/函数
    methods:{}
    
}
</script>

<style lang="less" scoped>

</style>