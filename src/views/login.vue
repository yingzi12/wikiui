<template>
  <div style="margin:0 auto; width:1258px;">
    <el-menu
        class="el-menu-demo"
        mode="horizontal"
        :ellipsis="false"
        :router="true"
    >
      <el-menu-item ><el-image style="width: 100%;height:55px" :src="logo" fit="fill"></el-image></el-menu-item>
      <el-menu-item index="/index" ><span style="font-size: 30px;font-weight:bold;">首页</span></el-menu-item>
      <el-menu-item index="/world/index" ><span style="font-size: 30px;font-weight:bold;">世界</span></el-menu-item>
      <el-menu-item index="/story/index" ><span style="font-size: 30px;font-weight:bold;">故事</span></el-menu-item>
      <el-menu-item index="/user/index" ><span style="font-size: 30px;font-weight:bold;">家园</span></el-menu-item>
      <el-menu-item index="/wiki/help" ><span style="font-size: 30px;font-weight:bold;">帮助</span></el-menu-item>
      <div class="flex-grow" />
<!--      <el-button v-if="!isLogin" text style="margin-top: 10px"><router-link :to="{path:'/login'}">登录</router-link></el-button>-->
<!--      <el-button v-if="!isLogin"  text style="margin-top: 10px"><router-link :to="{path:'/register'}">注册</router-link></el-button>-->
<!--      <el-button v-if="isLogin" text style="margin-top: 10px">{{ username }}</el-button>-->
<!--      <el-button v-if="isLogin" text style="margin-top: 10px">退出</el-button>-->
    </el-menu>
  </div>
  <el-divider  style="margin:0px"/>
  <div class="login">
    <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form">
      <h3 class="title">心世界</h3>
      <el-form-item prop="username">
        <el-input
          v-model="loginForm.username"
          type="text"
          size="large"
          auto-complete="off"
          placeholder="账号"
        >
          <template #prefix><svg-icon icon-class="user" class="el-input__icon input-icon" /></template>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          v-model="loginForm.password"
          type="password"
          size="large"
          auto-complete="off"
          placeholder="密码"
          @keyup.enter="handleLogin"
        >
          <template #prefix><svg-icon icon-class="password" class="el-input__icon input-icon" /></template>
        </el-input>
      </el-form-item>
      <el-form-item prop="code" v-if="captchaEnabled">
        <el-input
          v-model="loginForm.code"
          size="large"
          auto-complete="off"
          placeholder="验证码"
          style="width: 63%"
          @keyup.enter="handleLogin"
        >
          <template #prefix><svg-icon icon-class="validCode" class="el-input__icon input-icon" /></template>
        </el-input>
        <div class="login-code">
          <img :src="codeUrl" @click="getCode" class="login-code-img"/>
        </div>
      </el-form-item>
      <el-checkbox v-model="loginForm.rememberMe" style="margin:0px 0px 25px 0px;">记住密码</el-checkbox>
      <el-form-item style="width:100%;">
        <el-button
          :loading="loading"
          size="large"
          type="primary"
          style="width:100%;"
          @click.prevent="handleLogin"
        >
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
        <div  style="width:100%">
          <div style=" float: right;width: 50%">
          <router-link style="float: right;" class="link-type" :to="'/register'">立即注册</router-link>
          </div>
          <div style=" float: left;width: 50%">
             <router-link  class="link-type" :to="'/restpasswort'">找回密码</router-link>
          </div>
          </div>
      </el-form-item>
    </el-form>

  </div>
  <!--  底部  -->
  <div>
    <el-divider style="margin: 0px"/>
    <el-row :gutter="20">
      <el-col :span="2"> </el-col>
      <el-col :span="20" style="margin: 0px">
        <div class="tiaozhuan" style="margin: 0px;text-align: center;">
          <el-space  spacer="|">
            <div><router-link to="">关于心世界</router-link></div>
            <div><router-link to="">联系我们</router-link></div>
            <div> <router-link to="">帮助中心</router-link></div>
            <div><router-link to="">提交建议</router-link></div>
            <div><router-link to="">举报中心</router-link></div>
            <div> <router-link to="">漏洞提交</router-link></div>
          </el-space>
        </div>
        <div style="margin: 0px;text-align: center;"> <p  style="margin: 0px;">Copyright © 2002-2022 www.xingshijie.com All Rights Reserved 版权所有 心世界信息科技有限公司</p></div>
        <div style="margin: 0px;text-align: center;"><p style="margin: 0px;">本站所收录的作品、社区话题、用户评论、用户上传内容或图片等均属用户个人行为。如前述内容侵害您的权益，欢迎举报投诉，一经核实，立即删除，本站不承担任何责任</p></div>
      </el-col>
      <el-col :span="2"></el-col>
    </el-row>
  </div>
</template>

<script setup>
import { getCodeImg } from "@/api/login";
import logo from '@/assets/logo/logo.png'

import Cookies from "js-cookie";
import { encrypt, decrypt } from "@/utils/jsencrypt";
import useUserStore from '@/store/modules/user'

const userStore = useUserStore()
const router = useRouter();

const loginForm = ref({
  username: "",
  password: "",
  rememberMe: false,
  code: "",
  uuid: ""
});

const loginRules = {
  username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
  password: [{ required: true, trigger: "blur", message: "请输入您的密码" }],
  code: [{ required: true, trigger: "change", message: "请输入验证码" }]
};

const codeUrl = ref("");
const loading = ref(false);
// 验证码开关
const captchaEnabled = ref(true);
// 注册开关
const register = ref(true);
const redirect = ref(undefined);

function handleLogin() {
  // proxy.$refs.loginRef.validate(valid => {
  //   if (valid) {
      loading.value = true;
      // 勾选了需要记住密码设置在 cookie 中设置记住用户名和密码
      if (loginForm.value.rememberMe) {
        Cookies.set("username", loginForm.value.username, { expires: 30 });
        Cookies.set("password", encrypt(loginForm.value.password), { expires: 30 });
        Cookies.set("rememberMe", loginForm.value.rememberMe, { expires: 30 });
      } else {
        // 否则移除
        Cookies.remove("username");
        Cookies.remove("password");
        Cookies.remove("rememberMe");
      }
      // 调用action的登录方法
      userStore.login(loginForm.value).then(() => {
        router.push({ path: redirect.value || "/" });
      }).catch(() => {
        loading.value = false;
        // 重新获取验证码
        if (captchaEnabled.value) {
          getCode();
        }
      });
  //   }
  // });
}

function getCode() {
  getCodeImg().then(res => {
    captchaEnabled.value = res.captchaEnabled === undefined ? true : res.captchaEnabled;
    if (captchaEnabled.value) {
      codeUrl.value = "data:image/gif;base64," + res.img;
      loginForm.value.uuid = res.uuid;
    }
  });
}

function getCookie() {
  const username = Cookies.get("username");
  const password = Cookies.get("password");
  const rememberMe = Cookies.get("rememberMe");
  loginForm.value = {
    username: username === undefined ? loginForm.value.username : username,
    password: password === undefined ? loginForm.value.password : decrypt(password),
    rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
  };
}

getCode();
getCookie();
</script>

<style lang='scss' scoped>
.login {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  //background-image: url("../assets/images/login-background.jpg");
  background-size: cover;
}
.title {
  margin: 0px auto 30px auto;
  text-align: center;
  color: #707070;
}

.login-form {
  border-radius: 6px;
  background: #ffffff;
  width: 400px;
  padding: 25px 25px 5px 25px;
  .el-input {
    height: 40px;
    input {
      height: 40px;
    }
  }
  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 0px;
  }
}
.login-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}
.login-code {
  width: 33%;
  height: 40px;
  float: right;
  img {
    cursor: pointer;
    vertical-align: middle;
  }
}
.el-login-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #fff;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
}
.login-code-img {
  height: 40px;
  padding-left: 12px;
}
p{
  color: #a6a6a6;
  font: 12px/20px PingFangSC-Regular,-apple-system,Simsun;
  height: 20px;
  overflow: hidden;
}

.tiaozhuan {
  color: #1a1a1a;
  font-family: PingFangSC-Regular,-apple-system,Simsun;
  font-size: 12px;
  line-height: 27px;
  margin-right: 16px;
}
.flex-grow {
  flex-grow: 1;
}
</style>
