<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
       </el-breadcrumb> 

          <!-- 卡片视图区 -->
           <el-card class="box-card">
               <!-- 提示栏 -->
                <el-alert title="添加商品信息" type="info" show-icon center> </el-alert>
                <!-- 步骤栏 -->
                <el-steps :active="activeIndex-0" align-center finish-status="success">
                     <el-step title="基本信息" ></el-step>
                     <el-step title="商品参数" ></el-step>
                     <el-step title="商品属性"></el-step>
                     <el-step title="商品图片" ></el-step>
                     <el-step title="商品内容" ></el-step>
                     <el-step title="完成" ></el-step>
                </el-steps>

                <!-- 输入区 -->
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
                    <el-tabs tab-position="left"  v-model="activeIndex"
                     :before-leave="beforeTabLeave" @tab-click="tabClicked">
                    <!-- 1 基本信息-->
                    <el-tab-pane label="基本信息" name="0">
                         <el-form-item label="商品名称" prop="goods_name">
                                 <el-input v-model="addForm.goods_name"></el-input>
                         </el-form-item>
                         <el-form-item label="商品价格" prop="goods_price">
                                 <el-input v-model="addForm.goods_price" type="number"></el-input>
                         </el-form-item>
                         <el-form-item label="商品重量" prop="goods_weight">
                                 <el-input  v-model="addForm.goods_weight"  type="number"></el-input>
                         </el-form-item>
                         <el-form-item label="商品数量" prop="goods_number">
                                 <el-input type="number" v-model="addForm.goods_number" ></el-input>
                         </el-form-item>
                         <!-- 商品分类 -->
                        <el-form-item label="商品分类" prop="goods_cat">
                             <el-cascader v-model="addForm.goods_cat" :options="catelist"  
                             :props="cateProps" @change="handleChange" expand-trigger="hover" >
                             </el-cascader>
                         </el-form-item>
                        
                    </el-tab-pane>

                    <!-- 2 商品参数-->
                    <el-tab-pane label="商品参数" name="1">
                        <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                            <!-- 复选框 -->
                             <el-checkbox-group v-model="item.attr_vals">
                                   <el-checkbox :label="cb" v-for="(cb ,i) in item.attr_vals" :key="i" border></el-checkbox>
                            </el-checkbox-group>

                        </el-form-item>
                        
                    </el-tab-pane>

                    <!-- 3 商品属性-->
                    <el-tab-pane label="商品属性" name="2">
                        <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                                    <el-input v-model="item.attr_vals"></el-input>
                        </el-form-item>
                    </el-tab-pane>

                    <!-- 4 商品图片-->
                    <el-tab-pane label="商品图片" name="3">
                        <el-upload
                             action="http://127.0.0.1:8888/api/private/v1/upload"
                            :on-preview="handlePreview" :on-remove="handleRemove" 
                             list-type="picture" :headers="headerObj" :onsuccess="handleSuccess">
                                <el-button size="small" type="primary">点击上传</el-button>
                              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                            </el-upload>

                        
                    </el-tab-pane>

                    <!-- 5 商品内容-->
                    <el-tab-pane label="商品内容" name="4">
                        <!-- 富文本编辑器 -->
                         <quill-editor
                         ref="myQuillEditor"
                         v-model="addForm.goods_introduce"
                         :options="editorOption"
                          @blur="onEditorBlur($event)"
                          @focus="onEditorFocus($event)"
                           @ready="onEditorReady($event)" />
                          <el-button type="primary" class="btnAdd" @click="add()">添加商品</el-button>

                    </el-tab-pane>
                 </el-tabs>
                </el-form>
           </el-card>

           <!-- 图片预览 -->
             <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
                <img :src="previewPath" alt="" class="previewImg">
            </el-dialog>

    </div>
</template>

<script>
import _ from 'lodash'
export default {
  // 生命周期函数
    created(){
        // 获取商品分类
        this.getCateList();
    },
    // 数据
    data(){
        return{
            // 激活状态索引
          activeIndex:'0',
        //   参数
          addForm:{
              goods_name:'',
              goods_price:0,
              goods_weight:0,
              goods_number:0,
              goods_cat:[],
              pics:[],
              goods_introduce:'',
              attrs:[]
          },
        //   验证规则集
          addFormRules:{
             goods_name:[{required:true ,message:'请输入商品名称' ,trigger:'blur'}],
             goods_price:[{required:true ,message:'请输入商品价格' ,trigger:'blur'}],
             goods_weigth:[{required:true ,message:'请输入商品重量' ,trigger:'blur'}],
             goods_number:[{required:true ,message:'请输入商品数量' ,trigger:'blur'}],
             goods_cat:[{required:true ,message:'请输入商品分类' ,trigger:'blur'}]
          },
        //   商品分类列表
            catelist:[],
        // 商品分类列表 
            cateProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children'
            },
        //动态参数列表数据
        manyTableData:[], 

        // 静态属性
        onlyTableData:[],

        // 图片上传组件的请求头对象
        headerObj:{
            Authorization:window.sessionStorage.getItem('token')
        },
        // 预览图片
        previewPath:'',
        previewVisible:false,

        }
    },
    // 方法/函数
    methods:{
        // 获取商品分类
       async  getCateList(){
         const {data:res}=await this.$http.get('categories')
         if(res.meta.status !==200){
             return this.$message.error('获取商品分类列表失败！')
         }

        //  this.$message.success('获取商品分类列表成功');
         this.catelist=res.data ;
        //  console.log(res.data);
        },

        // 级联选择器触发函数
        handleChange(){
            if(this.addForm.goods_cat.length !==3){
                this.addForm.goods_cat =[]; }

        },

        // 禁用其他选项卡
        beforeTabLeave(activeName,oldActiveName){
            if(oldActiveName ==='0' && this.addForm.goods_cat.length !==3){
                this.$message.error("请选择商品分类！");
                return false
            }
            // console.log("进入："+activeName);
            // console.log("离开："+oldActiveName);
        },


        // 
       async tabClicked(){
           if(this.activeIndex ==='1'){
             const {data:res} =await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:'many'}})
                if(res.meta.status !==200){
                    return this.$message.error("获取动态参数列表失败！")
                }
                // console.log(res.data);
                res.data.forEach(item=>{
                    item.attr_vals =item.attr_vals.length ===0 ?[]:item.attr_vals.split(' ')
                })
                     this.manyTableData = res.data;

           }else if(this.activeIndex ==='2'){
            const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:'only'}})
                 if(res.meta.status !==200){
                    return this.$message.error("获取静态属性列表失败！")
                }
                this.onlyTableData =res.data;
           }
        },


        // 图片预揽效果
        handlePreview(file){
            this.previewPath=file.response.data.url;
            this.previewVisible= true;
        },

        // 移除图片的效果
        handleRemove(file){
            // 1.获取将要删除的图片的临时路径
            const filePath = file.response.data.tem_path;
            // 2.从pics数组中，找到这图片对应发索引值
            const i= this.addForm.pics.findIndex(x =>x.pic ===filePath);
            // 3.调用数组的splics方法，把图片信息对象，从pice数组中移除
            this.addForm.pics.splice(1,i);
        },

        // 监听图片上传成功的事件
        handleSuccess(response){
            // 1.拼接图片信息
            const picInfo ={
                pic:response.data.tmp_path
            }
            // 2.添加图片到数组中
            this.addForm.pics.push(picInfo)
        },
        // 添加商品
        add(){
            // 数据验证
            this.$refs.addFormRef.validate(async valid=>{
                if(!valid){
                    return this.$message.error('请填写必要的表单项！')
                }
                //执行添加的业务逻辑
               const form= _.cloneDeep(this.addForm)
               form.goods_cat = form.goods_cat.join(',')
            //    处理动态参数
            this.manyTableData.forEach(item =>{
                const newInfo ={attr_id:item.attr_vals,attr_value:item.attr_vals.join(' ')}
                this.addForm.attrs.push(newInfo)
            })
            // 处理静态属性
            this.onlyTableData.forEach(item =>{
                const newInfo ={ attr_id:item.attr_id , attr_value:item.attr_vals}
                 this.addForm.attrs.push(newInfo)
            })

            form.attrs =this.addForm.attrs
            //    console.log(form);
            // 发起请求
             const {data:res} =await this.$http.post('goods',form)
             if(res.meta.status !==201){
                 return this.$message.error("添加失败！")
             }
               this.$message.success("添加成功");
               this.$router.push('/goods')
            })
        },
        
    },
    // 计算属性
    computed:{
        cateId(){
            if(this.addForm.goods_cat.length===3){
             return this.addForm.goods_cat[2]
            }
            return null
        },
    }
    
}
</script>


<style lang="less" scoped>
.el-checkbox{
    margin:0px 10px 0px !important;
}
.previewImg{
    width:100%;
}
.btnAdd{
    margin-top: 20px;
}

</style>