<template>
    <div class="login_container">
        <!-- 1.头部标题logo -->
         <div class="login-header">
             <div>
                 <h1 class="fl"><img src="../assets/img/login/logo.png" alt=""></h1>
                 <p class="fl">云商城后台管理系统</p>
             </div>
        </div>

        <!-- 2登录框 -->
        <div class="login_box">
          <!--2.1 头像 -->
          <!-- <div class="avatar_box">
              <img src="../assets/img/login/tx.jpeg" alt="">
          </div> -->
          <h2>登 &nbsp;&nbsp;&nbsp;&nbsp; 录</h2>
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
        <div class="foot_box">
                  <p>本网站由小张个人独立开发|@copy版权所有|转载注明原作者 </p>
                  <p>友情链接:
                      <a href="https://blog.csdn.net/qq_42889406">https://blog.csdn.net/qq_42889406</a>
                </p>
                  <p>版本:vue_shop|2020.12.09|x.01.0</p>
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
*{
    padding:0;
    margin: 0;
}
.login_container{
    position:absolute;
    width: 100%;
    height:42.5rem;
    background:url("../assets/img/login/login-bg.jpg") no-repeat fixed center;
}

// 头部栏
.login-header{
    width:100%;
    height:5rem;
    background-color:rgba(75, 73, 73,0.75);
}

.login-header>div{
    margin-left:3.125rem;    
}
.login-header>div img{
    width: 3.75rem;
    height: 3.75rem;
    margin: .625rem;
}

.login-header>div>p{
    color: #fff;
    font-size:2rem;
    line-height:5rem;
    font-weight: bold; 
}



//登录框
.login_box{
   width:28.125rem;
   height:18.75rem;
   background-color: #fff;
   border-radius: .625rem;
   position:absolute;
   left: 70%;
   top:50% ;
   transform: translate(-50%,-50%);
}
.login_box>h2{
    margin-top:1.875rem;
    text-align: center;
}
.avatar_box{
    width:6.25rem;
    height:6.25rem;
    padding: .625rem;
    border:.125rem solid #cbcccc;
    border-radius: 50%;
    position:relative;
    top:-20%;
    margin:0 auto;
    background-color: #fff;
}
.avatar_box>img{
   width:100%;
   height:100%;
   border-radius: 50%;
   
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

// 底部内容
.foot_box{
    position:absolute;
    bottom:0;
    width:100%;
    height:5rem;
    padding:.9375rem 0;
    color:#eee;
    background-color:rgba(75, 73, 73,0.9);
    text-align: center;
}

.foot_box>p{
    font-size: .875rem;
    line-height: 1.5625rem;
}
</style>
