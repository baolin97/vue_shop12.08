<template>
    <div>
      <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
       </el-breadcrumb>
       <!-- 卡片视图区 -->
       <el-card class="box-card">
           <!-- 搜索区 -->
           <el-row :gutter="40">
                <el-col :span="10">
                 <!-- 搜索框 -->
                  <el-input placeholder="请输入内容" class="input-with-select" v-model="queryInfo.query" clearable @clear="getUserList()">
                       <el-button slot="append" icon="el-icon-search" @click="getUserList()"></el-button>
                 </el-input>
                </el-col>
                <el-col :span="4">
                        <el-button type="primary" @click="addDialogVisible=true" >添加用户</el-button>
                </el-col>
            </el-row>

            <!--用户表格 -->
             <el-table :data="userlist" border stripe>
                    <el-table-column label="#" type="index"  > </el-table-column>
                    <el-table-column label="姓名" prop="username" > </el-table-column>
                    <el-table-column label="邮箱" prop="email" > </el-table-column>
                    <el-table-column label="电话" prop="mobile" > </el-table-column>
                    <el-table-column label="角色" prop="role_name" > </el-table-column>
                    <el-table-column label="状态" > 
                        <template slot-scope="scope">
                                <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)" active-color="#409EFF" inactive-color="#C0CCDA">
                                </el-switch>
                        </template>
                    </el-table-column>

                    <el-table-column label="操作"   width="200">
                         <template slot-scope="scope">
                             <!-- 修改 -->
                               <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                               <!-- 删除 -->
                               <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                               <!-- 权限 -->
                                 <el-tooltip  effect="dark" content="设置权限" placement="top" :enterable=false>
                                    <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                                 </el-tooltip>
                            
                         </template>
                     </el-table-column>
            </el-table>
             <!-- 分页 -->
        <el-pagination  @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[2, 5, 10, 20]"
                 :page-size="queryInfo.pagesize"
                 layout="total, sizes, prev, pager, next, jumper"
                 :total="total">
        </el-pagination>
       </el-card>

       <!-- 添加用户提示框 -->
        <el-dialog
        title="添加管理员"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed">
            <!-- 内容主体区 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
               <!-- 用户名 -->
            <el-form-item label="用户名：" prop="username">
              <el-input v-model="addForm.username"></el-input>
            </el-form-item>
                  <!-- 密码 -->
            <el-form-item label="密码：" prop="password" >
              <el-input v-model="addForm.password" type="password"></el-input>
            </el-form-item>
                <!-- 用户名 -->
            <el-form-item label="邮箱：" prop="email">
              <el-input v-model="addForm.email"  ></el-input>
            </el-form-item>
                <!-- 用户名 -->
            <el-form-item label="手机：" prop="mobile">
              <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>

            </el-form>
            <span slot="footer" class="dialog-footer">
         <el-button @click="addDialogVisible = false">取 消</el-button>
         <el-button type="primary" @click="adduser">确 定</el-button>
        </span>
        </el-dialog>

        <!-- 编辑用户提示框 -->
        <el-dialog
          title="修改用户信息"
          :visible.sync="editDialogVisible" width="50%"
           @close="editDialogClosed()">
               <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                   <!-- 用户名 -->
                    <el-form-item label="用户名：">
                      <el-input v-model="editForm.username" disabled></el-input>
                    </el-form-item>
                    <!-- 邮箱 -->
                    <el-form-item label="邮箱：" prop="email">
                      <el-input v-model="editForm.email" ></el-input>
                    </el-form-item>
                    <!-- 联系方式 -->
                    <el-form-item label="手机：" prop="mobile">
                      <el-input v-model="editForm.mobile" ></el-input>
                    </el-form-item>
                </el-form>
             <span slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
             <el-button type="primary" @click="editUserInfo">确 定</el-button>
             </span>
        </el-dialog>

        <!-- 分配角色对话框 -->
        <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible"  width="50%"
          @close="setRoleDialogClosed"
        >
          
          <div>
             <p>当前的用户：{{userInfo.username}}</p>
             <p>当前的角色：{{userInfo.role_name}}</p>
            <p>分配新角色：
            <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option v-for="item in rolesList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id">
              </el-option>
            </el-select>
            </p>
          </div>
          

          <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo()">确 定</el-button>
           </span>
        </el-dialog>

    </div>
</template>

<script>
export default {
    // 数据
    data(){
        // 验证邮箱的规则
        var checkEmail=(rule ,value,cb)=>{
            const regEmail=/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            if(regEmail.test(value)){
                return cb()
            }
            cb(new Error('请输入合法的邮箱'))
        };

        //验证手机号的规则
        var chekcMobile=(rule ,value,cb)=>{
            const regMobile =/^(0|86|17951)?(13[0-9]|15[0-9]|17[678]|18[0-9]|14[57])[0-9]{8}$/
          if(regMobile.test(value)){
              return cb()
          }
          cb(new Error('请输入合法的手机号'))
        };

        return{
             userlist:[],
              total:0,


            // 获取用户列表参数
            queryInfo:{
                // 搜索内容
             query:'',
            //  当前页数
             pagenum:1,
            //  当前每页显示数据
             pagesize:2
            },

            // 新增用户表单数据
            addForm:{
                username:'',
                password:'',
                email:'',
                mobile:'',
            },

            // 表单验证规则对象
            addFormRules:{
                username:[
                    {required:true ,message:'请输入用户名',trigger:'blur'},
                    {min:3,max:10, message:'用户名长度在3-10个字符之间',trigger:'blur'}
                ],
                password:[
                     {required:true ,message:'请输入密码',trigger:'blur'},
                    {min:6,max:15, message:'用密码长度在3-10个字符之间',trigger:'blur'}
                ],
                email:[
                     {required:true ,message:'请输入邮箱',trigger:'blur'},
                     {validator:checkEmail ,trigger:'blur'}
              ],
                mobile:[
                    {required:true ,message:'请输入手机号',trigger:'blur'},
                     {validator:chekcMobile ,trigger:'blur'}
                ],

            },

            // 控制用户添加框的显示
            addDialogVisible:false,

            // 控制修改用户编辑框的显示
             editDialogVisible:false,

            //  查询到的用户信息对象
            editForm:{},

            // 修改表单的验证规则对象
            editFormRules:{
            email:[{required:true ,message:'请输入用户邮箱',trigger:'blur'},
            {validator: checkEmail ,trigger:'blur'}],
            mobile:[{required:true ,message:'请输入手机号码',trigger:'blur'},
            {validator:chekcMobile,trigger:'blur'}]
            
            
            },
            // 控制分配角色对话框的显示隐藏

            setRoleDialogVisible:false,
            // 需要分配角色的用户信息
            userInfo:{},
            //获取的角色列表
            rolesList:[],
            // 选中的角色id值
            selectedRoleId:''
        }
    },
    // 生命周期函数-创建
    created(){
        // 获取用户列表
        this.getUserList()
    },

    // 方法
     methods:{

        //  获取用户数据
    async  getUserList(){
        const {data:res} = await this.$http.get('users',{params:this.queryInfo})
           if(res.meta.status !==200 ){
               return this.$message.error('获取用户列表失败!');
           } 
           this.userlist=res.data.users,
           this.total=res.data.total  
        //  console.log(this.userlist);
      },

    // 监听pagesize改变的事件
      handleSizeChange(newSize){
        // console.log(newSize);
        this.queryInfo.pagesize=newSize;
        this.getUserList();
      },

    // 监听页面值改变的事件  
    handleCurrentChange(newPage){
        //  console.log(newPage);
        this.queryInfo.pagenum=newPage;
         this.getUserList();
     },

    //  状态改变
   async userStateChange(userinfo){
        //  console.log(userinfo);
     const {data:res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
    if(res.meta.status !==200){ 
        userinfo.mg_state=! userinfo.mg_state;
        return this.$message.error("更新用户失败!")};
       this.$message.success('更新用户状态成功')
     },

    //  监听添加用户对话框的关闭事件
    addDialogClosed(){
        this.$refs.addFormRef.resetFields();
    },

    // 添加新用户
    adduser(){
         this.$refs.addFormRef.validate(async valid=>{
            //  console.log(valid);
            if( !valid){ return };
               const {data:res}=await this.$http.post('users', this.addForm)
         if(res.meta.status !== 201){
             this.$message.error('添加用户失败！')
         }
          this.$message.success('添加用户成功');
          this.addDialogVisible =false;
          this.getUserList();
         })
    },
    // 修改用户信息
  async showEditDialog(id){
      const {data:res}  =await this.$http.get('users/'+id);
      if(res.meta.status !== 200){
          return this.$message.error('查询用户信息失败！');
      }
        this.editForm = res.data
        // console.log(this.editForm),
        this.editDialogVisible = true;
        // console.log(id);
  
     },

    //  监听修改用户对话框的关闭事件
    editDialogClosed(){
        this.$refs.editFormRef.resetFields()
    },

    // 修改用户信息并提交
        editUserInfo(){
            this.$refs.editFormRef.validate(async valid=>{
                if(!valid) return ;
                const {data:res}=await this.$http.put('users/'+this.editForm.id,{
                    email:this.editForm.email,
                    mobile:this.editForm.mobile})

                    if(res.meta.status!==200){return this.$message.error('更新用户信息失败！')}
                    // console.log(valid);
                    this.getUserList();
                    this.$message.success("用户信息修改成功")
                    this.editDialogVisible= false;
            })
        },

    // 根据id删除用户信息
       async removeUserById(id){
        const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
           return err
        })
        // 判断是否删除了
        if(confirmResult !='confirm' ){
          return this.$message.info("已经取消删除")
        }
        const {data:res}= await this.$http.delete('users/'+id);
        if(res.meta.status !==200){ return this.$message.error('删除失败')}
        this.$message.success("删除成功");
        this.getUserList();

    },

    // 展示分配角色的对话框
  async  setRole(userInfo){
      this.userInfo=userInfo;
      //获取角色列表
      const {data:res}=await  this.$http.get(`roles`);
          if(res.meta.status !==200){
            return this.$message.error('获取角色列表失败！')
          };
          this.rolesList=res.data;
        this.setRoleDialogVisible=true;
    },
    // 点击确定按钮，提交角色
   async saveRoleInfo(){
      if(!this.selectedRoleId){
        return $message.error("未选择角色信息！")
      }

      const {data:res}= await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId});
        if(res.meta.status !==200){
         return this.$message.error("设置用户角色失败！")
        }
        this.$message.success("设置用户角色成功");
        this.getUserList();
        this.setRoleDialogVisible = false;
     },
    // 监听分配角色 取消按钮
    setRoleDialogClosed(){
      this.selectedRoleId="",
      this.userInfo={}
    }
    },   
}
</script>

<style lang="less" scoped>
</style>