<template>
  <el-form :model="ruleForm2" :rules="rules2" ref="ruleForm2" label-position="left" label-width="0px" class="demo-ruleForm login-container">
    <h3 class="title">系统登录</h3>
    <el-form-item prop="account">
      <el-input type="text" v-model="ruleForm2.account" auto-complete="off" placeholder="账号"></el-input>
    </el-form-item>
    <el-form-item prop="checkPass">
      <el-input type="password" v-model="ruleForm2.checkPass" auto-complete="off" placeholder="密码"></el-input>
    </el-form-item>
    <el-form-item prop="checkPass">
      <el-input type="hidden" v-model="ruleForm2.loginToken" auto-complete="off"></el-input>
    </el-form-item>
    <el-checkbox v-model="checked" checked class="remember">记住密码</el-checkbox>
    <el-form-item style="width:100%;">
      <el-button type="primary" style="width:100%;" @click.native.prevent="handleSubmit2" :loading="logining">登录</el-button>
      <!--<el-button @click.native.prevent="handleReset2">重置</el-button>-->
    </el-form-item>
  </el-form>
</template>

<script>
  import { requestLogin, requestLoginByToken } from '../api/api';
  import { setCookie, getCookie, delCookie } from '../utils/cookie'
  //import NProgress from 'nprogress'
  export default {
    data() {
      return {
        logining: false,
        ruleForm2: {
          account: '',
          checkPass: ''
        },
        rules2: {
          account: [
            { required: true, message: '请输入账号', trigger: 'blur' },
            //{ validator: validaePass }
          ],
          checkPass: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            //{ validator: validaePass2 }
          ]
        },
        checked: true
      };
    },
    created: function() {
        var user = getCookie('op_account_info')
        if (null != user) {
          user = JSON.parse(user)
          this.ruleForm2.account = user.userName
          this.ruleForm2.checkPass = user.loginToken
          this.ruleForm2.loginToken = user.loginToken
        } else {
          this.ruleForm2.loginToken = ""
        }
    },
    methods: {
      handleReset2() {
        this.$refs.ruleForm2.resetFields();
      },
      handleSubmit2(ev) {
        var _this = this;
        this.$refs.ruleForm2.validate((valid) => {
          if (valid) {
            //_this.$router.replace('/table');
            this.logining = true;
            //NProgress.start();
            
            var loginParams = { userName: this.ruleForm2.account, password: this.ruleForm2.checkPass };
            var loginRequest = requestLogin
            if (this.ruleForm2.loginToken != "" && this.ruleForm2.loginToken == this.ruleForm2.checkPass) {
                // 按照token登录
                loginRequest = requestLoginByToken
                loginParams = { userName: this.ruleForm2.account, token: this.ruleForm2.loginToken };
            }
            loginRequest(loginParams).then(data => {
              this.logining = false;
              //NProgress.done();
              if (data.state != 1) {
                this.$message({
                  message: data.data.msg,
                  type: 'error'
                });
              } else {
                var user = {
                  userName: data.data.userName,
                  nickName: data.data.nickName,
                  networkType: data.data.networkType,
                  loginToken: data.data.loginToken
                }
                sessionStorage.setItem('user', JSON.stringify(user));
                this.$router.push({ path: '/' });

                if (this.checked) {
                  // 记住用户名
                  let accountInfo = JSON.stringify({userName:user.userName, loginToken: user.loginToken})
                  setCookie("op_account_info", accountInfo)
                } else {
                  // 删除cookie记录
                  delCookie("op_account_info")
                }
              }
            });
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      }
    }
  }

</script>

<style lang="scss" scoped>
  .login-container {
    /*box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);*/
    -webkit-border-radius: 5px;
    border-radius: 5px;
    -moz-border-radius: 5px;
    background-clip: padding-box;
    margin: 180px auto;
    width: 350px;
    padding: 35px 35px 15px 35px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
    .title {
      margin: 0px auto 40px auto;
      text-align: center;
      color: #505458;
    }
    .remember {
      margin: 0px 0px 35px 0px;
    }
  }
</style>