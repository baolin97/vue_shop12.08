<template>
    <div>
            <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
       </el-breadcrumb> 

     <!-- 卡片视图区 -->
     <el-card class="box-card">
         <!-- 按钮 -->
       <el-row>
          <el-col>
            <el-button  type="primary" >添加角色</el-button>
          </el-col>
       </el-row>
           <!-- 表格 -->
         <el-table :data="rolelist" border stripe>
           <!-- 展开列 -->
           <el-table-column type="expand" label="拓展" width="100">
             <template slot-scope="scope">
               <el-row :class="['bdbottom', i1===0? 'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                 <!-- 一级权限 -->
                  <el-col :span="5">
                    <el-tag closable
                                @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                   <!-- 二级权限 -->
                  <el-col :span="19">
                      <el-row :class="[i2===0? '': 'bdtop','vcenter']" v-for="(item2 ,i2) in item1.children" :key="item2.id">
                        <!-- 二级权限 -->
                         <el-col :span="6">
                           <el-tag type='success' closable
                                @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                           <i class="el-icon-caret-right"></i>
                         </el-col>
                         <!--三级权限 -->
                         <el-col :span="18">
                                <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" 
                                closable
                                @close="removeRightById(scope.row,item3.id)"
                                >
                                  {{item3.authName}}
                                </el-tag>
                         </el-col>
                      </el-row>
                  </el-col>
                  
               </el-row>
                <!-- <pre>
                     {{scope.row}}
                </pre> -->

             </template>
           </el-table-column>

            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="400">
                 <template slot-scope="scope">
                       <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                       <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                       <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">权限</el-button>
                 </template>
            </el-table-column>
         </el-table>

    </el-card>

    <!-- 展示分配权限对话框 -->
    <el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="50%"
   @close="setRightDialogClosed">
  <!-- 树形结构 -->
  <el-tree :data="rightslist" show-checkbox node-key="id"
   default-expand-all=true
  :default-checked-keys="defkeys"
  :props="treeProps"
  ref="treeRef">
</el-tree>

  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>

  

    </div>
</template>

<script>
export default {
  // 生命周期函数-
  created(){
    this.getRolesList();
  },

   data(){
     return{
      //  所有角色列表数据
      rolelist:[],
      // 分配权限id
      roleId:'',
      // 控制分配权限框显示隐藏
      setRightDialogVisible:false,
      //所有权限数据
      rightslist:[],
      // 树形控件的属性绑定对象
      treeProps:{
          label: 'authName',
          children: 'children',
      },
      // 默认选中的节点
      defkeys:[]
     }
   },

   methods:{
    //  获取角色列表
    async getRolesList(){
      const {data:res}= await this.$http.get('roles');
      if(res.meta.status !==200){  return this.$message.error('角色列表获取失败！')}
      this.rolelist= res.data;
        // console.log(this.rolelist);
     },

    //  根据id删除对应权限
    async removeRightById(role ,rightId){
    const confirmResult= await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult !=='confirm'){
          return this.$message.info('取消删除')
        }
      const {data:res}= await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        // console.log('确认删除');
        if(res.meta.status !==200){
          return this.$message.error("删除取消失败！")
        }
        this.$message.success("删除成功");
        role.children =res.data
     },
    //  展示分配权限的对话框
    async showSetRightDialog(role){
      this.roleId=role.id;
      // 获取权限数据
      const {data:res}=await this.$http.get('rights/tree')
        if(res.meta.status !==200){
           return this.$message.eror("获取权限数据失败")
        };
        // 获取到的数据保存到
        this.rightslist=res.data;
        // 获取三级节点
        this.getLeafKeys(role,this.defkeys);

        // console.log( this.rightslist);
       this.setRightDialogVisible =true; 
    },

    // 获取所有三级节点的id(提供递归的方式，获取角色下所有三级权限的id,并且保存到数据中defkeys)
    getLeafKeys(node,arr){
        if(!node.children){
          return arr.push(node.id)
        }
        node.children.forEach(item=>this.getLeafKeys(item,arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed(){
          this.defkeys=[];
    },
    // 点击分配权限
   async allotRights(){
      const keys=[
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ];
      const idStr=keys.join(',');
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr});
         if(res.meta.status!== 200){ return this.$message.error("权限分配更新失败")};
        this.$message.success("权限分配更新成功");
        this.getRolesList();

      // console.log(keys);
      // 隐藏权限框
       this.setRightDialogVisible = false
    }
   }

    
}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.bdtop{
  border-top: 1px solid #eee;
}
.bdbottom{
  border-bottom:1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}

</style>