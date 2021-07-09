<template>
  <Nav>
    <div class="user">
      <div class="user-wrapper">
        <div class="user-search">
          <el-input/>
          <el-button size="small" type="primary">查询</el-button>
        </div>
        <el-button size="small" type="primary" @click="showAddUser = true" :disabled="!hasAuth">添加用户</el-button>
      </div>
      <el-table
        :data="partTableData"
        border
        stripe
        style="width: 100%">
        <el-table-column fixed prop="id" label="用户ID" min-width="150"/>
        <el-table-column fixed prop="name" label="姓名" min-width="150"/>
        <el-table-column prop="gender" label="性别" min-width="150"/>
        <el-table-column prop="age" label="年龄" min-width="150"/>
        <el-table-column prop="tel" label="手机号" min-width="150"/>
        <el-table-column fixed="right" label="操作" min-width="200">
          <template slot-scope="scope" v-if="hasAuth">
            <el-button type="text" size="small" @click="edit(scope.row)">编辑</el-button>
            <el-button @click="deleteRow(scope.row)" type="text" size="small">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination class="fenye"  @current-change="handleCurrentChange" :current-page="currentPage"
        :page-size="8"
        background
        layout="total, prev, pager, next"
        :total="this.tableData.length">
      </el-pagination>
      <el-dialog
        class="user-dialog"
        title="添加用户"
        :visible.sync="showAddUser"
        width="30%">
        <el-form ref="addForm" :rules="rule" :model="addForm" label-width="100px" size="mini" class="addUser-form">
          <el-form-item label="姓名" prop="name">
            <el-input v-model="addForm.name" clearable></el-input>
          </el-form-item>
          <el-form-item label="性别" prop="gender">
            <el-select v-model="addForm.gender" placeholder="请选择">
              <el-option label="女" value="女"/>
              <el-option label="男" value="男"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model="addForm.age" clearable></el-input>
          </el-form-item>
          <el-form-item label="电话号码" prop="tel">
            <el-input v-model="addForm.tel"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="showAddUser = false" size="small">取 消</el-button>
          <el-button type="primary" @click="addUser" size="small">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        v-if="editData"
        class="user-dialog"
        title="编辑用户"
        :visible.sync="editShow"
        width="30%">
        <el-form ref="editForm" :rules="rule" :model="editData" label-width="100px" size="mini" class="addUser-form">
          <el-form-item label="姓名" prop="name">
            <el-input v-model="editData.name" clearable></el-input>
          </el-form-item>
          <el-form-item label="性别" prop="gender">
            <el-select v-model="editData.gender" placeholder="请选择">
              <el-option label="女" value="女"/>
              <el-option label="男" value="男"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model="editData.age" clearable></el-input>
          </el-form-item>
          <el-form-item label="电话号码" prop="tel">
            <el-input v-model="editData.tel"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editShow = false" size="small">取 消</el-button>
          <el-button type="primary" @click="confirmEdit" size="small">确 定</el-button>
        </span>
      </el-dialog>
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
  showAddUser = false;
  editShow = false;
  hasAuth = false;
  addInit = {
    name: '',
    tel: '',
    gender:'',
    age:'',
  }
  tableData = [];
  addForm = this.addInit;
  editData = null;
  currentPage = 1;
  rule = {
    name: {
      required: true,
      message: '请输入名称',
      trigger: 'blur'
    },
    gender: {
      required: true,
      message: '请输入性别',
      trigger: 'blur'
    },
    tel: [
      {required: true,message:'请输入电话号码', trigger: 'blur' },
      {required: true, pattern: /^1\d{10}$/, message: '请输入合法号码（11位，1开头)',trigger: 'blur' }
    ],
    age: [
      {required: true,message:'请输入年龄', trigger: 'blur' },
      {required: true,pattern: /^(?:[1-9][0-9]?|1[01][0-9]|120)$/, message: '合法年龄1-120', rigger: 'blur'}
    ]
  }
  get partTableData() {
    return this.tableData.slice((this.currentPage-1)*8, Math.min(this.currentPage*8, this.tableData.length + 1));
  }
  handleCurrentChange(id) {
    console.log(id);
    this.currentPage = id;
  }
  created() {
    this.getUsers();
    this.hasAuth = (JSON.parse(window.sessionStorage.getItem('admin')).auth == 2)
  }
  getUsers() {
    axios.get('http://localhost:9999/c/user/list')
    .then((response) => {
      const { data } = response.data;
      this.tableData = data
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  deleteRow(id) {
    this.$confirm('确认删除').then(() => {
      const config = {
        method: 'delete',
        url: 'http://localhost:9999/c/user/delete',
        headers: { 'Content-Type': 'application/json' },
        data: id
      };
      axios(config).then(() => {
        const loading = this.$loading({
          text: '正在加载中......',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)',
        })
        this.$message.success('删除成功');
        this.getUsers();
        loading.close();
      }).catch((error) => {
        this.$message.error(error);
      });
    })
  }
  addUser() {
    this.$refs.addForm.validate((valid) => {
      if (valid) {
        let ids = 1;
        const arr = [];
        this.tableData.forEach(item => {arr.push(parseInt(item.id))});
        while(arr.includes(ids))ids++;
        const data = Object.assign(this.addForm,{id: ids.toString()});
        axios.post('http://localhost:9999/c/user/insert', data).then(() => {
          const loading = this.$loading({
            text: '正在加载中......',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
          this.$message.success('添加成功');
          this.addForm = this.addInit;
          this.showAddUser = false;
          this.getUsers();
          loading.close();
        }).catch((err) => {
          this.$message.error(err);
        })
      }
    })

  }
  edit(obj) {
    this.editData = JSON.parse(JSON.stringify(obj));
    this.editShow = true;
  }
  confirmEdit() {
    this.$refs.editForm.validate((valid) => {
      if (valid) {
        axios.post('http://localhost:9999/c/user/edit', this.editData).then(() => {
          const loading = this.$loading({
            text: '正在加载中......',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
          this.$message.success('编辑成功');
          this.editShow = false;
          this.getUsers();
          loading.close();
        }).catch((err) => {
          this.$message.error(err);
        })
      }
    })
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

