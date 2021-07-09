<template>
<div class="appfun">
  <div class="nav">
    <h1 class="logo">
      <img :src="logo" >
      <h6>服装库存管理</h6>
    </h1>
    <div class="wrapper">
      <el-menu
        class="el-menu-demo"
        mode="horizontal"
        background-color="#545c64"
        text-color="#fff"
        active-text-color="#e7410b"
        :router="true"
        :default-active = "path"
      >
        <el-menu-item index="/user">用户管理</el-menu-item>
        <el-menu-item index="/goods">货号管理</el-menu-item>
        <el-menu-item index="/ingoods">入库管理</el-menu-item>
        <el-menu-item index="/outgoods">出库管理</el-menu-item>
        <el-menu-item index="/admin" v-if="hasAuth">权限管理</el-menu-item>
      </el-menu>
    </div>
    <div class="user">
      <i class="el-icon-s-custom"/>
      {{ username }}
      <el-tooltip class="item" effect="light" content="编辑" placement="bottom-end">
        <i class="el-icon-edit-outline xxx" @click="flagShow = true"/>
      </el-tooltip>
    </div>
    <el-dialog
      class="dialog"
      title="修改基本信息"
      :visible.sync="flagShow"
      width="30%">
          <el-form class="form" :rules="rule" :model="xxx" ref="xxform">
            <el-form-item label='用户名：' prop="name">
              <el-input v-model="xxx.username" placeholder="请输入用户名" clearable/>
            </el-form-item>
            <el-form-item label='密码：' prop="password">
              <el-input v-model="xxx.password" type="password" placeholder="请输入密码" clearable/>
            </el-form-item>
            <el-form-item label='新密码：' prop="newp">
              <el-input v-model="xxx.newp" type="password" placeholder="请输入新密码" clearable/>
            </el-form-item>
            <el-form-item prop="rep">
              <el-input v-model="xxx.rep" type="password" placeholder="请重复输入新密码" clearable class="ooo"/>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" size="small" @click="ok">确认</el-button>
            </el-form-item>
          </el-form>

    </el-dialog>
  </div>
  <div class="mainSlot">
    <slot/>
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
export default class Nav extends Vue {
  logo = logo;
  flagShow = false;
  xxxInit = {
    oldName: '',
    username: '',
    password: '',
    newp: '',
    rep: ''
  }
  hasAuth = false;

  xxx = JSON.parse(JSON.stringify(this.xxxInit));
  rule = {
    username: {
      required: true,
      message: '请输入用户名',
      trigger: 'blur'
    },
    password: [
      { required: true, message: '请输入密码', trigger: 'blur' },
    ],
    newp: [
      { required: true, message: '请输入新密码', trigger: 'blur' },
      { validator: this.validateNewPwd, trigger: 'blur' }
    ],
    rep: [
      { required: true, message: '请输入确认密码', trigger: 'blur' },
      { validator: this.validateComfirmPwd, trigger: 'blur' }
    ]
  }

  validateNewPwd(rule, value, callback) {
    if (this.xxx.password === value) {
      callback(new Error('新密码与旧密码不可一致！'))
    } else {
      callback()
    }
  }

  validateComfirmPwd(rule, value, callback) {
    if (this.xxx.newp !== this.xxx.rep) {
      callback(new Error('确认密码与新密码不一致！'))
    } else {
      callback()
    }
  }

  get username() {
    return JSON.parse(window.sessionStorage.getItem('admin')).username;
  }
  created() {
    this.onRouteChanged();
    this.xxx.username = JSON.parse(window.sessionStorage.getItem('admin')).username;
    this.xxx.oldName = this.xxx.username;
    this.hasAuth = (JSON.parse(window.sessionStorage.getItem('admin')).auth == 2)
  }
  onRouteChanged() {
    this.path  = this.$route.path
  }

  ok() {
    this.$refs.xxform.validate((valid) => {
      if(valid) {
        const data = {
          username: this.xxx.oldName,
          password: this.xxx.password
        }
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
            const postDate = {
              username: this.xxx.username,
              password: this.xxx.newp
            }
            const config = {
              method: 'post',
              url: 'http://localhost:9999/c/admin/edit',
              headers: { 'Content-Type': 'application/json' },
              data: postDate
            };
            axios(config).then((response) => {
              this.$message.success('修改成功，请重新登录');
              this.xxx = JSON.parse(JSON.stringify(this.xxxInit));
              this.$router.push('/index');
            }).catch(err => {
              this.$message.error(err);
            })
          }
        }).catch((error) => {
          this.$message.error(error);
        });
      }
    })
  }
}
</script>

<style lang="scss" scoped>
  .nav {
    display: flex;
    background-color: #545c64;
    .el-menu.el-menu--horizontal {
      border-bottom: none;
    }
    > .logo {
      vertical-align: middle;
      width: 280px;
      color: #fff;
      font-weight: 400;
      display: flex;
      justify-content: center;
      align-items: center;
      > img {
        height: 40px;
        margin-right: 20px;
      }
    }
    > .wrapper {
      flex: 1;
    }
    > .user {
      display: flex;
      align-items: center;
      padding: 10px;
      color: white;
      font-size: 16px;
      font-weight: 700;
       i {
        font-size: 28px;
        margin-right: 6px;
         &:hover {
           cursor: pointer;
         }
      }
      .xxx {
        margin-left: 6px;
        font-size: 20px;
      }
    }
  }
  .ooo {
    margin-top: 8px;
  }
</style>