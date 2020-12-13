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
            <!-- 权限表格 -->
           <el-table :data="rightsList" border stripe>
                    <el-table-column label="#" type="index"  > </el-table-column>
                    <el-table-column label="权限名称" prop="authName" > </el-table-column>
                    <el-table-column label="路径" prop="path" > </el-table-column>
                    <el-table-column label="权限等级" prop="level">
                        <template slot-scope="scope">
                            <el-tag v-if="scope.row.level === '0'">一级</el-tag>
                            <el-tag v-else-if="scope.row.level==='1'" type="success">二级</el-tag>
                            <el-tag v-else type="warning">三级</el-tag>
                        </template>
                    </el-table-column>
            </el-table>

  <!-- 分页区 -->
     <!-- <el-pagination @size-change="SizeChange" @current-change="CurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 20, 40]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination> -->

    </el-card>
    </div>
</template>

<script>
export default {
    // 生命周期函数-创建后
    created(){
        //获取所有的权限数据
        this.getRightsList()
    },

    // 数据
    data(){
        return{
            rightsList:[],
            total:0
        }
    },
   // 方法-函数
    methods:{
        // 获取权限列表
        async getRightsList(){
          const {data:res}=await this.$http.get('/rights/list')
           if(res.meta.status !==200 ){
                return this.$message.error("获取用户权限列表失败！")
           }
           this.rightsList=res.data;
           this.total=res.data.length;
        //    console.log(this.rightsList);
        },

           // 监听pagesize改变的事件
    //   SizeChange(newSize){
    //     // console.log(newSize);
    //     this.queryInfo.pagesize=newSize;
    //     this.getRightsList();
    //     // console.log(111);
    //     // console.log(this.queryInfo.pagesize);
    //   },

    // // 监听页面值改变的事件  
    // CurrentChange(newPage){
    //     // console.log(newPage);
    //     this.queryInfo.pagenum=newPage;
    //     this.getRightsList();
    //     // console.log(222);
    //  },


    },
    
}
</script>

<style lang="less" scoped>

</style>