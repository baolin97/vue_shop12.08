<template>
    <div class="login_container">
        <!-- 1.头部标题logo -->
        <div>
            <h1>LOGO</h1>
        </div>
        <!-- 2登录框 -->
        <div class="login_box">
          <!--2.1 头像 -->
          <div class="avatar_box">
              <img src="../assets/logo.png" alt="">
          </div>
          <!-- 2.2 登录表单 -->
           <div> 
               <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="70px" class="form_box">
                     <el-form-item  label="账号：" prop="username">
                       <el-input v-model="loginForm.username" placeholder="请输入账号" prefix-icon="iconfont icon-user-fill"></el-input>
                     </el-form-item>
                     <el-form-item label="密码：" prop="password">
                        <el-input v-model="loginForm.password" type="password" placeholder="请输入密码" prefix-icon="iconfont icon-lock"></el-input>
                     </el-form-item>
                     <el-form-item class="btns">
                        <el-button type="primary" @click="login">登录</el-button>
                        <el-button type="info" @click="resetLoginForm">重置</el-button>
                    </el-form-item>
               </el-form>
           </div>
        </div>
        <!-- 3.底部信息区 -->
        <div>
            <p> 本网站由小张同学提供技术支持</p>
        </div>

    </div>
</template>

<script>
export default {
    data(){
        return{
            // 登录表单的数据绑定对象
        loginForm:{
            username:'',
            password:''
        },
        // 验证对象集
         loginFormRules:{
            //  验证用户名
               username:[
                     { required:true, message: '请输入用户名称', trigger: 'blur' },
                     { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
               ],
            // 验证密码
               password:[
                    { required:true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
               ]
           }
        }
           
    },
    methods:{
        resetLoginForm(){
            this.$refs.loginFormRef.resetFields();
        },
        login(){
            this.$refs.loginFormRef.validate(async valid=>{
                if(!valid) return;
             const{data:res} =await this.$http.post('login',this.loginForm);
            //  console.log(result);    
            if(res.meta.status !==200){ 
                this.$message.error(res.meta.msg);
                this.resetLoginForm()}else{
                this.$message.success(res.meta.msg);
                // console.log(res);
                window.sessionStorage.setItem("token" ,res.data.token);
                this.$router.push('/home');
            };            
                })
                // this.resetLoginForm()
        }
        
    }
   
}
</script>

<style lang="less" scoped>
.login_container{
    height: 100%;
    background-color: #429da1;
}
.login_box{
   width:28.125rem;
   height:18.75rem;
   background-color: #fff;
   border-radius: .625rem;
   position:absolute;
   left: 50%;
   top:50% ;
   transform: translate(-50%,-50%);
}
.avatar_box{
    width:8.125rem;
    height:8.125rem;
    padding: .625rem;
    border:.125rem solid #eee;
    border-radius: 50%;
    position:relative;
    top:-30%;
    margin:0 auto;
    background-color: #fff;
  
}
.avatar_box>img{
   width:100%;
   height: 100%;
   border-radius: 50%;
   background-color: #eee;
}
.form_box{
    position: absolute;
    top:6.25rem;
    padding:0 1.875rem 0 .625rem;
    box-sizing: border-box;
    width: 100%;
}
.btns{
    display: flex;
    justify-content: flex-end;

}

</style>
