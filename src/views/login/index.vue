<template>
  <div class="login-container">
    <el-card class="login-form-layout">
      <el-form
        ref="loginForm"
        :model="loginForm"
        :rules="loginRules"
        label-position="left"
      >
        <div style="text-align:center">
          <svg-icon icon-class="login-mall" style="width: 56px;height: 56px;color: #409EFF" />
        </div>
        <h2 style="text-align:center">mall-admin-web</h2>
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            name="username"
            type="text"
            auto-complete="on"
            placeholder="请输入用户名"
          >
            <span slot="prefix">
              <svg-icon icon-class="user" class="color-main" />
            </span>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            name="password"
            :type="pwdType"
            placeholder="请输入密码"
            auto-complete="on"
            @keyup.enter.native="handleLogin"
          >
            <span slot="prefix">
              <svg-icon icon-class="password" class="color-main" />
            </span>
            <span slot="suffix" @click="showPwd">
              <svg-icon icon-class="eye" class="coloe-main" />
            </span>
          </el-input>
        </el-form-item>
        <el-form-item style="margin-bottom:60px;text-align:center">
          <el-button
            style="width:45%"
            type="primary"
            :loading="loading"
            @click.native.prevent="handleLogin"
          >登录</el-button>
          <el-button style="width:45%" type="primary" @click.native.prevent="handleTry">获取体验账号</el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <img :src="login_center_bg" class="login-center-layout">
    <el-dialog
      title="公众号二维码"
      :visible.sync="dialogVisible"
      :show-close="false"
      :center="true"
      width="30%">
      <div style="text-align:center">
        <span class="font-title-large"><span class="color-main font-extra-large">关注公众号</span>回复<span class="color-main font-extra-large">体验</span>获取体验账号</span>
        <br>
        <img src="http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg" width="160" height="160" style="margin-top: 10px">
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button title="primary" @click="dialogConfirm">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { isvalidUsername } from '@/utils/validate'
import login_center_bg from '@/assets/images/login_center_bg.png'
import {setSupport,getSupport,setCookie,getCookie} from '@/utils/support'

export default {
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!isvalidUsername(value)) {
        callback(new Error('请输入正确的用户名'))
      } else {
        callback()
      }
    }
    const validatePass = (rule, value, callback) => {
      if (value.length < 3) {
        callback(new Error('密码不能小于3位'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: '',
        password: ''
      },
      loginRules: {
        username: [
          { required: true, trigger: 'blur', validator: validateUsername }
        ],
        password: [{ required: true, trigger: 'blur', validator: validatePass }]
      },
      loading: false,
      pwdType: 'password',
      login_center_bg,
      dialogVisible: false,
    }
  },
  created() {
    this.loginForm.username = getCookie('username')
    this.loginForm.password = getCookie('password')
    if (this.loginForm.username === undefined || this.loginForm.username === null || this.loginForm.username === '') {
      this.loginForm.username = 'admin'
    }
    if (this.loginForm.password === undefined || this.loginForm.password == null) {
      this.loginForm.password = ''
    }
  },
  methods: {
    showPwd() {
      if (this.pwdType === 'password') {
        this.pwdType = ''
      } else {
        this.pwdType = 'password'
      }
    },
    handleLogin() {
        this.$refs.loginForm.validate(valid => {
          if (valid) {
            // let isSupport = getSupport();
            // if(isSupport===undefined||isSupport==null){
            //   this.dialogVisible =true;
            //   return;
            // }
            this.loading = true;
            this.$store.dispatch('Login', this.loginForm).then(() => {
              this.loading = false;
              setCookie('username',this.loginForm.username,15);
              setCookie('password',this.loginForm.password,15);
              this.$router.push({ path: '/' })
            }).catch(() => {
              this.loading = false
            })
          } else {
            console.log('参数验证不合法！')
            return false
          }
        })
      },
    handleTry() {
      this.dialogVisible = true
    },
    dialogConfirm() {
      this.dialogVisible = false
      setSupport(true)
    }
  }
}
</script>

<style scoped>
.login-container{
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login-form-layout{
  width: 300px;
  height: 360px;
  position: fixed;
}
.login-center-layout{
  height: 200px;
  background: #409EFF
}
</style>
