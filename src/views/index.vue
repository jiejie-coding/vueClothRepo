<template>
  <div class="index">
    <img :src="logo" >
    <div class="tabs">
      <el-tabs type="border-card">
        <el-tab-pane label="登录">
          <el-form class="form" :model="loginDate">
            <el-form-item >
              <el-input v-model="loginDate.username" placeholder="请输入用户名" clearable>
                <el-button slot="prepend" icon="el-icon-user-solid"></el-button>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-input v-model="loginDate.password" type="password" placeholder="请输入密码" clearable>
                <el-button slot="prepend" icon="el-icon-key"></el-button>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" size="small" @click="login">登录</el-button>
            </el-form-item>
          </el-form>
        </el-tab-pane>
        <el-tab-pane label="注册">
            <el-form class="form" :model="regDate">
              <el-form-item >
                <el-input v-model="regDate.username" placeholder="请输入用户名" clearable>
                  <el-button slot="prepend" icon="el-icon-user-solid"></el-button>
                </el-input>
              </el-form-item>
              <el-form-item>
                <el-input v-model="regDate.password" type="password" placeholder="请输入密码" clearable>
                  <el-button slot="prepend" icon="el-icon-key"></el-button>
                </el-input>
              </el-form-item>
              <el-form-item>
                <el-input v-model="regDate.rePassword" type="password" placeholder="请输入密码" clearable>
                  <el-button slot="prepend" icon="el-icon-key"></el-button>
                </el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" size="small" @click="reg">注册</el-button>
              </el-form-item>
            </el-form>
          </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
import logo from '@/assets/style/logo.png';
import { Component, Vue } from 'vue-property-decorator';
import axios from 'axios';
@Component({
  components: { },
})
export default class Index extends Vue {
  logo = logo;
  loginInit = {
    username: '',
    password: ''
  }
  loginDate = this.loginInit;
  login() {
    const data = JSON.stringify(this.loginDate);
    const config = {
      method: 'post',
      url: 'http://localhost:9999/c/login',
      headers: { 'Content-Type': 'application/json' },
      data: data
    };
    axios(config).then((response) => {
      const { data } = response.data;
      if(data.flag === "1") {
        this.$message.error('该用户不存在');
      } else if(data.flag === '2') {
        this.$message.error('密码错误');
      } else if(data.flag === '3') {
        this.$message.success('登录成功');
        window.sessionStorage.setItem('admin',JSON.stringify({'username': data.username, 'auth': data.auth}));
        this.$router.push('/goods');
      }
    }).catch((error) => {
      this.$message.error(error);
    });
    this.loginDate = this.loginInit;
  }

  regInit = {
    username: '',
    password: '',
    rePassword: '',
  }
  regDate = JSON.parse(JSON.stringify(this.regInit));
  reg() {
    delete this.regDate.rePassword;
    const data = JSON.stringify(this.regDate);
    const config = {
      method: 'post',
      url: 'http://localhost:9999/c/register',
      headers: { 'Content-Type': 'application/json' },
      data: data
    };
    axios(config).then((response) => {
      const { data } = response.data;
      if(data.flag === "1") {
        this.$message.error('服务器错误');
      } else if(data.flag === '2') {
        this.$message.error('用户已经存在');
      } else if(data.flag === '3') {
        this.$message.success('注册成功');
        this.regDate = JSON.parse(JSON.stringify(this.regInit));
      }
    }).catch((error) => {
      this.$message.error(error)
    });
  }
}
</script>

<style lang="scss">
  .index {
    height: 100vh;
    background-color: #545c64;
    padding: 50px;
    display: flex;
    justify-content: space-between;
    > img {
      margin-left: 50px;
      display: block;
    }
    .tabs {
      float: right;
      width: 40vw;
      height: 400px;
      margin-top: 150px;
    }
    .form {
      margin-top: 30px;
      margin-left: 60px;
      margin-right: 60px;
    }
  }
</style>

