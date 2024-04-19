<template>
  <div class="Login">
    <!-- 登录框 -->
    <div class="login-form-main animate__animated animate__flipInX" v-if="showDom">
      <!-- 登录框背景 -->
      <div class="login-form-left">
        <div class="loginTitle">
          <img :src="Logo" alt="" class="loginLogo">
          <h3 class="title">GPS调度系统</h3>
        </div>
      </div>
      <!-- 登录框表单 -->
      <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form">
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" type="text" size="large" auto-complete="off" placeholder="账号">
            <template #prefix>
              <svg-icon icon-class="user" class="el-input__icon input-icon"/>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" size="large" auto-complete="off" placeholder="密码"
                    @keyup.enter="handleLogin">
            <template #prefix>
              <svg-icon icon-class="password" class="el-input__icon input-icon"/>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="code" v-if="captchaEnabled">
          <el-input v-model="loginForm.code" size="large" auto-complete="off" placeholder="验证码" style="width: 63%"
                    @keyup.enter="handleLogin">
            <template #prefix>
              <svg-icon icon-class="validCode" class="el-input__icon input-icon"/>
            </template>
          </el-input>
          <div class="login-code">
            <img :src="codeUrl" @click="getCode" class="login-code-img"/>
          </div>
        </el-form-item>
        <el-checkbox v-model="loginForm.rememberMe" style="margin:0px 0px 25px 0px;">记住密码</el-checkbox>
        <el-form-item style="width:100%;">
          <el-button :loading="loading" size="large" type="primary" style="width:100%;" @click.prevent="handleLogin">
            <span v-if="!loading">登 录</span>
            <span v-else>登 录 中...</span>
          </el-button>
          <div style="float: right;" v-if="register">
            <router-link class="link-type" :to="'/register'">立即注册</router-link>
          </div>
        </el-form-item>
      </el-form>
    </div>
    <!-- 底部标志 -->
    <div class="el-register-footer" v-if="showDom">
      <span>Copyright © 2021-2024 江苏恒旺数字科技有限公司 All Rights Reserved.</span>
    </div>
  </div>
</template>

<script setup name="Login">
import {getCodeImg} from "@/api/login";
import Cookies from "js-cookie";
import {encrypt, decrypt} from "@/utils/jsencrypt";
import useUserStore from '@/store/modules/user'
import {useRoute} from "vue-router"
import Logo from '../assets/logo/HWlogo.png'

const {proxy} = getCurrentInstance();

const userStore = useUserStore()
const router = useRouter();
const showDom = ref(false)

const route = useRoute()

const loginForm = ref({
  username: "",
  password: "",
  rememberMe: false,
  code: "",
  uuid: ""
});

if (route.query.username && route.query.password) {
  loginForm.value.username = route.query.username
  loginForm.value.password = route.query.password
  // 调用action的登录方法
  userStore.login(loginForm.value).then(() => {
    router.push({path: redirect.value || "/"});
  }).catch(() => {
    proxy.$modal.msgError('自动登录失败')
    showDom.value = true
  });
} else {
  showDom.value = true
}

const loginRules = {
  username: [{required: true, trigger: "blur", message: "请输入您的账号"}],
  password: [{required: true, trigger: "blur", message: "请输入您的密码"}],
  code: [{required: true, trigger: "change", message: "请输入验证码"}]
};

const codeUrl = ref("");
const loading = ref(false);
// 验证码开关
const captchaEnabled = ref(true);
// 注册开关
const register = ref(false);
const redirect = ref(undefined);

function handleLogin() {
  proxy.$refs.loginRef.validate(valid => {
    if (valid) {
      loading.value = true;
      // 勾选了需要记住密码设置在 cookie 中设置记住用户名和密码
      if (loginForm.value.rememberMe) {
        Cookies.set("username", loginForm.value.username, {expires: 30});
        Cookies.set("password", encrypt(loginForm.value.password), {expires: 30});
        Cookies.set("rememberMe", loginForm.value.rememberMe, {expires: 30});
      } else {
        // 否则移除
        Cookies.remove("username");
        Cookies.remove("password");
        Cookies.remove("rememberMe");
      }
      // 调用action的登录方法
      userStore.login(loginForm.value).then(() => {
        router.push({path: redirect.value || "/"});
      }).catch(() => {
        loading.value = false;
        // 重新获取验证码
        if (captchaEnabled.value) {
          getCode();
        }
      });
    }
  });
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
  localStorage.removeItem('companyId')
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
@media screen and (max-width: 768px) {
  .login-form-left {
    display: none !important;
  }
  .login-form-main {
    width: auto !important;
  }
  .login-form {
    border-radius: 10px !important;
  }
}

.Login {
  background: url('../assets/images/login-background.jpg') no-repeat;
  background-size: cover;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;

  .login-form-main {
    width: 830px;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 0.10417vw 0.625vw 0 rgba(0, 0, 0, 0.7);
    display: flex;
    height: 310px;
    transition: all 0.2s;

    .login-form {
      background: #fff;
      width: 320px;
      padding: 36px 25px 5px 25px;
      border-radius: 0 10px 10px 0;

      :deep(.el-form-item--default) {
        margin-bottom: 30px;
      }

      :deep(.el-input__wrapper) {
        box-shadow: none;
        border-bottom: 1px solid rgba(223, 224, 226, 0.7);
      }

      :deep(.el-form-item__error) {
        margin: 4px 0 0 15px;
      }

      .el-button {
        border-radius: 20px;
        box-shadow: 0 5px 15px 0 rgb(54 133 242 / 20%);
      }

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
        color: rgba(0, 102, 242, 0.7);
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

      .login-code-img {
        height: 40px;
        padding-left: 8px;
      }
    }

    .login-form-left {
      flex: 1;
      border-radius: 10px 0 0 10px;
      background: url('../assets/images/login-left-bg.jpg') no-repeat left;
      background-size: cover;
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1;
      position: relative;

      &::after {
        content: '';
        position: absolute;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 102, 242, 0.1);
        border-radius: 10px 0 0 10px;
        z-index: 2;
      }

      .loginTitle {
        z-index: 3;

        .title {
          font-size: 30px;
          //color: rgb(8, 52, 138);
          font-style: italic;
          color: #fff;
          font-weight: 700;
          text-align: center;
          margin: 10px 0;
        }

        .loginLogo {
          width: 100%;
          transform: scale(0.7);
          filter: brightness(100)
        }
      }

    }
  }

  .el-register-footer {
    height: 40px;
    line-height: 40px;
    position: fixed;
    bottom: 0;
    width: 100%;
    text-align: center;
    color: #444444;
    font-family: Arial;
    font-size: 12px;
    letter-spacing: 1px;
  }
}</style>
