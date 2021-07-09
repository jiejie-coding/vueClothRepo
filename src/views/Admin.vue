<template>
  <Nav>
    <div class="user">
      <div class="user-wrapper">
        <div class="user-search">
          <el-input/>
          <el-button size="small" type="primary">查询</el-button>
        </div>
      </div>
      <el-table
        :data="partTableData"
        border
        stripe
        style="width: 100%">
        <el-table-column fixed prop="username" label="姓名" min-width="150"/>
        <el-table-column prop="auth" label="权限" min-width="150">
          <template slot-scope="scope">
            {{scope.row.auth === 1 ? '只读' : '可写'}}
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" min-width="200">
          <template slot-scope="scope">
            <el-button type="text" size="small" @click="give(scope.row.username)">授予权限</el-button>
            <el-button type="text" size="small" @click="back(scope.row.username)">回收权限</el-button>
            <el-button @click="deleteRow(scope.row.username)" type="text" size="small">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination class="fenye"  @current-change="handleCurrentChange" :current-page="currentPage"
        :page-size="8"
        background
        layout="total, prev, pager, next"
        :total="this.tableData.length">
      </el-pagination>

    </div>
  </Nav>
</template>

<script>
import { Component, Vue } from 'vue-property-decorator';
import Nav from '@/components/Nav.vue';
import axios from 'axios';
@Component({
  components: { Nav },
})
export default class User extends Vue {
  tableData = [];
  currentPage = 1;
  get partTableData() {
    return this.tableData.slice((this.currentPage-1)*8, Math.min(this.currentPage*8, this.tableData.length + 1));
  }
  handleCurrentChange(id) {
    this.currentPage = id;
  }
  created() {
    this.getAdmins();
  }
  getAdmins() {
    axios.get('http://localhost:9999/c/admin/list')
    .then((response) => {
      const { data } = response.data;
      this.tableData = data
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  deleteRow(username) {
    this.$confirm('确认删除').then(() => {
      const config = {
        method: 'delete',
        url: 'http://localhost:9999/c/admin/delete',
        headers: { 'Content-Type': 'application/json' },
        data: username
      };
      axios(config).then(() => {
        return this.getAdmins();
      }).then(() => {
        this.$message.success('删除成功');
      }).catch((error) => {
        this.$message.error(error);
      });
    })
  }
  give(username) {
    const config = {
      method: 'post',
      url: 'http://localhost:9999/c/admin/auth',
      headers: { 'Content-Type': 'application/json' },
      data: {
        username: username,
        auth: 2
      }
    };
    axios(config).then(() => {
      return this.getAdmins();
    }).then(() => {
      let objs = JSON.parse(window.sessionStorage.getItem('admin'));
      objs.auth = 2;
      window.sessionStorage.setItem('admin',JSON.stringify(objs))
      this.$message.success('授予权限成功');
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  back(username) {
    const config = {
      method: 'post',
      url: 'http://localhost:9999/c/admin/auth',
      headers: { 'Content-Type': 'application/json' },
      data: {
        username: username,
        auth: 1
      }
    };
    axios(config).then(() => {
      return this.getAdmins();
    }).then(() => {
      let objs = JSON.parse(window.sessionStorage.getItem('admin'));
      objs.auth = 1;
      window.sessionStorage.setItem('admin',JSON.stringify(objs))
      this.$message.success('回收权限成功');
    }).catch((error) => {
      this.$message.error(error);
    });
  }

}

</script>

<style lang="scss">
.user {
  padding: 15px;
  &-wrapper {
    display: flex;
    margin-bottom: 10px;
    text-align: right;
    justify-content: space-between;
  }
  &-search {
    display: flex;
  }
  &-dialog {
    text-align: left;
  }
}
.fenye {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: -99;
}
</style>

