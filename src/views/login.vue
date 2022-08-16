<template>
  <div class="page">
    <el-row type="flex" justify="center" align="middle" class="container">
      <el-col :lg="14" :xl="10" class="hidden-md-and-down">
        <el-row class="panel">
          <el-col :span="12">
            <div class="left">
              <img src="../assets/login/logo.png" class="logo"/>
              <img src="../assets/login/big-1.png" class="big"/>
            </div>
          </el-col>
          <el-col :span="12">
            <div class="right">
              <div class="title-container">
                <h2>Office在线办公系统</h2>
                <span>( Ver 1.0 )</span>
              </div>
              <div v-if="!qrCodeVisible">
                <div class="row">
                  <el-input
                      v-model="username"
                      placeholder="用户名"
                      prefix-icon="el-icon-user"
                      clearable="clearable"
                  />
                </div>
                <div class="row">
                  <el-input
                      type="password"
                      v-model="password"
                      placeholder="密码"
                      prefix-icon="el-icon-lock"
                      clearable="clearable"
                  />
                </div>
                <div class="row">
                  <el-button type="primary" class="btn" @click="login">登陆系统</el-button>
                </div>
                <div class="row"><a class="link" @click="changeMode">二维码登陆</a></div>
              </div>
              <div v-if="qrCodeVisible">
                <div class="qrCode-container">
                  <img :src="qrCode" height="153" class="qrCode"/>
                  <img src="../assets/login/phone.png" height="148"/>
                </div>
                <div class="row"><a class="link" @click="changeMode">用户名密码登陆</a></div>
              </div>
            </div>
          </el-col>
        </el-row>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import 'element-plus/lib/theme-chalk/display.css';
import router from '../router/index.js';
import {isPassword, isUsername} from '../utils/validate.js';

export default {
  data() {
    return {
      username: null,
      password: null,
      qrCodeVisible: false,
      qrCode: '',
      uuid: null,
      qrCodeTimer: null,
      loginTimer: null
    };
  },

  methods: {

    login() {
      if (!isUsername(this.username)) {//前端校验失败
        this.$message({
          message: '用户名格式不正确',
          type: 'error',
          duration: 1200//1.2s
        });
      } else if (!isPassword(this.password)) {//前端校验失败
        this.$message({
          message: '密码格式不正确',
          type: 'error',
          duration: 1200//1.2s
        });
      } else {
        //封装数据
        let data = {userName: this.username, password: this.password};
        //发送请求
        this.$http('user/login', 'POST', data, true, resp => {
          if (resp.result) {
            localStorage.setItem('permissions', resp.permissions);
            localStorage.setItem('token', resp.token);
            this.$message({
              message: '登陆成功',
              type: 'success',
              duration: 1200
            });
            this.$router.push({name: 'Home'});
          } else {
            this.$message({
              message: '登陆失败',
              type: 'error',
              duration: 1200
            });
          }
        })
      }
    },

    changeMode() {
      let that = this;
      that.qrCodeVisible = !that.qrCodeVisible;
      //加载二维码图片
      if (that.qrCodeVisible) {
        that.loadQRCode();
        //创建刷新二维码定时器
        that.qrCodeTimer = setInterval(function () {
          that.loadQRCode();
        }, 5 * 60 * 1000);
        that.loginTimer = setInterval(function () {
          that.$http('user/wechatLogin', 'POST', {uuid: that.uuid}, true, function (resp) {
            if (resp.result) {
              clearInterval(that.qrCodeTimer);
              clearInterval(that.loginTimer);
              let permissions = resp.permissions;
              localStorage.setItem('permissions', permissions);
              router.push({name: 'Home'});
            }
          });
        }, 5000);
      } else {
        //销毁刷新二维码定时器
        clearInterval(that.qrCodeTimer);
        clearInterval(that.loginTimer);
      }
    },
    //加载二维码图片的封装方法
    loadQRCode() {
      this.$http('user/createQrCode', 'GET', null, true, resp => {
        this.qrCode = resp.pic;
        this.uuid = resp.uuid;
      });
    }
  }
};
</script>

<style lang="less" scoped="scoped">
@import url('login.less');
</style>
