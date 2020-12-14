<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
       </el-breadcrumb> 

          <!-- 卡片视图区 -->
        <el-card class="box-card">
          <!-- 按钮 -->
              <el-row>
                   <el-col>
                     <el-button  type="primary" @click="showAddCateDialog()">添加分类</el-button>
                 </el-col>
             </el-row>
             <!-- 表格 tree-table -->
             <tree-table :data="catelist" 
             :columns="columns" 
             :selection-type="false"
             :expand-type="false"
             show-index index-text="#"
             border
             class="tree-table"
             >
             <!-- 是否有效——插槽 -->
             <template slot="isok" slot-scope="scope">
                    <i class="el-icon-success" style="color:lightgreen ;font-size:20px" v-if="scope.row.cat_deleted===false"></i>
                    <i class="el-icon-error" style="color:lightgreen ; font-size:20px" v-else></i>
             </template>
             <!-- 是否有效——排序 -->
              <template slot="islevel" slot-scope="scope">
                    <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" v-else>三级</el-tag>
              </template>
               <!-- 是否有效——操作 -->
              <template slot="isbtn" slot-scope="scope">
                     <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                       <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
              </template>


             </tree-table>

           <!-- 表格 -->
         <!-- <el-table :data="catelist" border stripe>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="分类名称" prop="cat_name"></el-table-column>
            <el-table-column label="是否有效" prop="cat_deleted"></el-table-column>
            <el-table-column label="排序"  prop="cat_level"></el-table-column>
            <el-table-column label="操作" width="400">
                 <template slot-scope="scope">
                       <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                       <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                 </template>
            </el-table-column>
         </el-table> -->

         <!-- 分页 -->
          <el-pagination  @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5, 10, 20]"
                 :page-size="queryInfo.pagesize"
                 layout="total, sizes, prev, pager, next, jumper"
                 :total="total">
        </el-pagination>


        </el-card>

        <!-- 添加对话框 -->

                <el-dialog
                @close="addCateDialogClosed"
            title="添加分类"
           :visible.sync="addCateDialogVisible" width="50%">
           <!-- 添加分类表单 -->
         <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
             <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类：">
                  <!-- 级联选择器 -->
             <el-cascader style="width:100%"
                 popper-class="flotage"
                 expand-trigger="hover"
                 :options="parentCateList"
                 :props="cascaderProps"
                  v-model="selectedKeys"
                 @change="parentCateChanged"
                 clearable>
             </el-cascader>

            </el-form-item>

         </el-form>
            <span slot="footer" class="dialog-footer">
             <el-button @click="addCateDialogVisible = false">取 消</el-button>
                 <el-button type="primary" @click="addCate">确 定</el-button>
                 </span>
        </el-dialog>

    </div>
</template>

<script>
export default {
    // 生命周期函数
    created(){
        this.getCateList()
    },
    // 数据
    data(){
        return{
            // 商品分类的数据列表，默认为空
            catelist:[],
            //查询条件
            queryInfo:{type:3, pagenum:1, pagesize:5 },
            // 数据条数
            total:0,
            // 为table指定列的定义
            columns:[
                {
                    label:'分类名称',
                    prop:'cat_name'

            },{
                label:'是否有效',
                prop:'cat_deleted',
                type:'template',
                template:'isok'

            },{
                 label:"排序",
                 prop:"cat_level",
                  type:'template',
                 template:'islevel'

            },{
                label:"操作" ,
                width:"400",
                 type:'template',
                 template:'isbtn'
            }],
            // 控制添加框的显示
            addCateDialogVisible: false,
            // 添加分类的表单数据对象
            addCateForm:{
                // 分类名称
                cat_name:'',
                // 分类的父级id
                cat_pid:0,
                // 添加分类的等级
                cat_level:0
            },

            // 添加分类表单的验证规则对象
            addCateFormRules:{
                  cat_name: [
                      { required:true , message:'请输入类的名称' , trigger:'blur'}
                  ],
            },

            // 父级获取的分类的列表
            parentCateList:[],
            cascaderProps:{
                value: 'cat_id',
                label: 'cat_name',
                children: 'children',  
                checkStrictly:true
            },
            // 选中的父级分类的id数组
            selectedKeys: [],

        }
    },
    // 方法/函数
    methods:{
        // 获取商品分类列表
       async getCateList(){
          const {data:res}=await this.$http.get(`categories`,{params:this.queryInfo})
                if(res.meta.status !==200){
                    return this.$message.error('获取商品分类信息失败！')
                }
                // 数据列表赋值给catelist 
                this.catelist=res.data.result;
                // console.log(this.catelist);
                // 数据条数赋值
                this.total=res.data.total;

                
        },

         // 监听pagesize改变的事件
      handleSizeChange(newSize){
        // console.log(newSize);
        this.queryInfo.pagesize=newSize;
        this. getCateList();
      },

    // 监听页面值改变的事件  
    handleCurrentChange(newPage){
        //  console.log(newPage);
        this.queryInfo.pagenum=newPage;
         this. getCateList();
     }, 

    //  添加分类
    showAddCateDialog(){  
          this.getParentCateList()
         this.addCateDialogVisible=true;
    },
    // 获取父级分类的数据列表
  async  getParentCateList(){
      const{data:res}= await this.$http.get(`categories`,{params: { type:2 }});
        if(res.meta.status !==200){
           return this.$message.error("获取父级列表失败！")
        };

        this.parentCateList=res.data;
       console.log(this.parentCateList);

    },

    // 选择项发生变化
   parentCateChanged(){
        // console.log(this.selectedKeys);
     if(this.selectedKeys.length > 0){
     this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length-1];
     this.addCateForm.cat_level=this.selectedKeys.length;
     return 
     }else{
       this.addCateForm.cat_pid = 0;
       this.addCateForm.cat_level =0;
     }
    },
    // 点击确认按钮
    addCate(){
            this.$refs.addCateFormRef.validate(async valid=>{
                if(!valid) return
          const {data:res}=  await this.$http.post('categories',this.addCateForm);
          if(res.meta.status !==201){
              return this.$message.error("添加分类失败！")
          }
          this.$message.success("添加分类成功");
          this.getCateList();
          this.addCateDialogVisible =false;
            })
        },
        // 关闭事件
    addCateDialogClosed(){
            this.$refs.addCateFormRef.resetFields();
            this.selectedKeys=[];
            this.addCateForm.cat_level =0;
            this.addCateForm.cat_pid=0
},

    }
    
}
</script>

<style lang="less" scoped>
.el-cascader{
    width:100%
}

</style>