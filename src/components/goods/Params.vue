<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
       </el-breadcrumb> 

          <!-- 卡片视图区 -->
           <el-card class="box-card">
               <!-- 1.提示标签 -->
               <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="true" show-icon>
               </el-alert>
               <!-- 2.输入框 :选择商品分类：  -->
               <el-row class="cat_opt">
                   <el-col>
                       <span>选择商品分类：</span>
                       <!-- 级联现选择框 -->
                         <el-cascader v-model="selectedCateKeys"
                            style="width:30%"
                            popper-class="flotage"
                            expand-trigger="hover"
                            :options="catelist"
                            :props="cateProps"
                            @change="handleChange"></el-cascader>
                   </el-col>
               </el-row>

               <!-- 3. tab页签区 -->
               <template>
                  <el-tabs v-model="activeName" @tab-click="handleTabClick">
                     <!-- 添加动态参数的面板 -->
                     <el-tab-pane label="动态参数" name="many">
                         <!-- 添加参数的按钮 -->
                         <el-button type="primary" size="mini" :disabled='isBtnDisabled'  @click="addDialogVisible = true">添加参数</el-button>
                         <!-- 动态参数表格 -->
                        <el-table :data="manyTableData" border stripe>
                            <el-table-column type="expand">
                                  <template slot-scope="scope">
                                      <!-- 渲染tag标签 -->
                                    <el-tag v-for="(item , i) in scope.row.attr_vals" :key="i" closable style="margin-left:10px" 
                                    @close="handleClose(i,scope.row)"
                                    >{{item}}</el-tag>
                                     <!-- 输入文本框 -->
                                    <el-input
                                        class="input-new-tag"
                                         v-if="scope.row.inputVisible"
                                        v-model="scope.row.inputValue"
                                         ref="saveTagInput"
                                         size="small"
                                         @keyup.enter.native="handleInputConfirm(scope.row)"
                                         @blur="handleInputConfirm(scope.row)"
                                         style="width:150px"
                                            >
                                        </el-input>
                                        <el-button v-else class="button-new-tag"  style="width:150px" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                                </template>
                            </el-table-column>
                            <el-table-column type="index" label="#"></el-table-column>
                            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                            <el-table-column label="操作">
                                <template slot-scope="scope">
                                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                     </el-tab-pane>

                     <!-- 添加静态属性的面板 -->
                     <el-tab-pane label="静态属性" name="only">
                         <!-- 添加属性的按钮 -->
                          <el-button type="primary" size="mini" :disabled=' isBtnDisabled' @click="addDialogVisible = true">添加属性</el-button>
                           <!--  静态属性表格 -->
                        <el-table :data="onlyTableData" border stripe>
                            <el-table-column type="expand">
                                 <template slot-scope="scope">
                                      <!-- 渲染tag标签 -->
                                    <el-tag v-for="(item , i) in scope.row.attr_vals" :key="i" closable style="margin:10px" 
                                    @close="handleClose(i,scope.row)"
                                    >{{item}}</el-tag>
                                     <!-- 输入文本框 -->
                                    <el-input
                                        class="input-new-tag"
                                         v-if="scope.row.inputVisible"
                                        v-model="scope.row.inputValue"
                                         ref="saveTagInput"
                                         size="small"
                                         @keyup.enter.native="handleInputConfirm(scope.row)"
                                         @blur="handleInputConfirm(scope.row)"
                                         style="width:150px"
                                            >
                                        </el-input>
                                        <el-button v-else class="button-new-tag"  style="width:150px" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                                </template>
                            </el-table-column>
                            <el-table-column type="index" label="#"></el-table-column>
                            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                            <el-table-column label="操作">
                                <template slot-scope="scope">
                                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>

                     </el-tab-pane>

                  </el-tabs>
                </template>

           </el-card>

           <!-- 添加参数对话框 -->
            <el-dialog
  @close="addDialogClosed"
  :title="'添加'+titleText"
  :visible.sync="addDialogVisible"
  width="50%"
  >
  <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>

  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams()">确 定</el-button>
  </span>
            </el-dialog>

         <!--编辑参数对话框  -->
            <el-dialog
  @close="editDialogClosed"
  :title="'修改'+titleText"
  :visible.sync="editDialogVisible"
  width="50%"
  >
  <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
  </el-form>

  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams()">确 定</el-button>
  </span>
            </el-dialog>

    </div>
</template>

<script>
import LoginVue from '../Login.vue';
export default {
  // 生命周期函数
    created(){
        this.getCateList()
    },

    // 数据
    data(){
        return{
            catelist:[],
            // 级联选择框
            cateProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children'
            },
            // 级联选择框绑定到的数组
            selectedCateKeys:[],
            // 被激活的页签的名称
            activeName:'many',
            // 动态参数是数据
            manyTableData:[],
            // 静态参数的数据
            onlyTableData:[],
            // 控制添加对话框的显示与隐藏
            addDialogVisible:false,
            // 添加参数的表单对象
            addForm:{
                attr_name:''
            },
            // 添加表单的验证规则对象
            addFormRules:{
                attr_name:[{required:true,message:'请输入参数名称',trigger:'blur'}]
            },
            // 控制编辑对话框的显示与隐藏
            editDialogVisible:false,
            // 修改表单的数据对象
            editForm:{},
            // 修改表单的数据验证对象
             editFormRules:{
                attr_name:[{required:true,message:'请输入参数名称',trigger:'blur'}]
            },

            // 输入tag:显示隐藏文本框
            inputVisible:false,
            // 文本框输入的内容
            inputValue:''
            
        }
    },

    // 方法/函数
    methods:{
        // 获取所有的商品分类列表
       async getCateList(){
         const {data:res}=await this.$http.get('categories');
         if(res.meta.status !==200){
            return this.$message.error('获取商品分类失败')
         }
         this.catelist=res.data;
        //  console.log(this.catelist);
        },
        //级联选择框变化，会触发这个函数
        handleChange(){
          this.getParamsData()
        },

        //tab页签点击事件处理的函数
        handleTabClick(){
            this.getParamsData()
        } ,

        // 获取参数的列表数据
       async getParamsData(){
             if(this.selectedCateKeys.length !==3){
                 this.selectedCateKeys=[];
                 this.manyTableData=[];
                 this.onlyTableData=[];
                 return 
           }
        // 根据所选分类的id,和当前所处的面板，获取对应的参数
        const{data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}})
          if(res.meta.status !==200){
             return this.$message.error("获取参数列表失败！")
          }
            //  console.log(res.data);
            // 判断

            res.data.forEach( item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(' '):[];
                // 控制文本框显示与隐藏
                item.inputVisible =false
                //文本框输入的值
                item.inputValue=''
            });
            console.log(res.data);

            if(this.activeName ==='many'){
                this.manyTableData = res.data
            }else{
                this.onlyTableData = res.data
            }

        },

        // 监听对话框的关闭事件
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },

        // 点击确认添加参数
       addParams(){
         this.$refs.addFormRef.validate(async valid=>{
            if(!valid) return;
           const {data :res}=await this.$http.post(`categories/${this.cateId}/attributes`,{
                attr_name:this.addForm.attr_name,
                attr_sel:this.activeName
            })
            if(res.meta.status !==201){ return this.$message.error("添加参数失败！")};
            this.$message.success('添加参数成功！');
            this.addDialogVisible = false;
            this.getParamsData()
        })
        },

        // 编辑按钮
       async showEditDialog(attr_id){
          const {data:res}= await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{
                params:{attr_sel:this.activeName}
            })
             if(res.meta.status !==200){
               return this.$message.error("获取参数失败！")
             }
             this.editForm=res.data;
            this.editDialogVisible=true
            },

        // 监听编辑框的关闭事件（重置表单）
        editDialogClosed(){
             this.$refs.editFormRef.resetFields()
        },
        // 编辑-点击确认按钮
        editParams(){
            this.$refs.editFormRef.validate(async valid =>{
               if(!valid){return}
                const {data:res}= await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{ attr_name:this.editForm.attr_name, attr_sel: this.activeName})
                 if(res.meta.status !==200){
                  return this.$message.error('修改失败!')
              }
            this.$message.success('修改成功！')
            this.editDialogVisible = false;
            this.getParamsData()
            })

         
           
            },
    
            // 删除操作
          async  removeParams( attr_id ){
         const confirmResult=await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=> err)
        if(confirmResult !=='confirm'){
            return this.$message.info('已取消删除！')
        }
        // 删除的逻辑
         const {data:res}= await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
         if(res.meta.status !==200){
            return this.$message.error("删除参数失败！")
         }
         this.$message.success("删除参数成功");
         this.getParamsData()
        },

        // 文本框失去焦点，按下Enter都会触发
        async handleInputConfirm(row){
             if(row.inputValue.trim().length ===0){
                 row.inputValue =''
                 row.inputVisible=false
                 return
             }
             row.attr_vals.push(row.inputValue.trim());
             row.inputValue='';
             row.inputVisible=false;
             this.saveAttrVals(row);
            
                         
         },
          // 将attr_vals的操作保存到数据库
         async saveAttrVals(row){
                 const {data:res}= await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
                  attr_name:row.attr_name,
                  attr_sel:row.attr_sel,
                  attr_vals:row.attr_vals.join(' '),
              });
              if(res.meta.status !==200){
                 return this.$message.error('添加失败！')
              }
             this.$message.success('添加成功');
            },

        //  点击按钮，展示文本输入框
        showInput(row){
            row.inputVisible = true;
             this.$nextTick(_ => {
            this.$refs.saveTagInput.$refs.input.focus();
        });
        },

        // 删除对应的标签
        handleClose(i,row){
            row.attr_vals.splice(i ,1);
            this.saveAttrVals(row);
        }

    },
    //计算属性
    computed:{
        // 如果按钮需要被禁用，则返回true ,否则返回false
        isBtnDisabled(){
            if(this.selectedCateKeys.length !== 3){
                return true
            }
            return false
        },
        // 当前选中的三级分类的ID
        cateId(){
            if(this.selectedCateKeys.length ===3){
                return this.selectedCateKeys[2]
            }
            return null;
        },
        // 动态计算标题的文本
        titleText(){
            if(this.activeName ==='many'){
                return '动态参数'
            }else{
                return '静态属性'
            }
        },


    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin: 15px 0;
    font-size:16px;
}


</style>