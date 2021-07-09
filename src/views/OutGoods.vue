<template>
  <Nav>
    <div class="outgoods">
      <div class="outgoods-wrapper">
        <div class="outgoods-search">
          <el-input/>
          <el-button size="small" type="primary">查询</el-button>
        </div>
      </div>
      <el-table
        :data="partTableData"
        border
        stripe
        style="width: 100%">
        <el-table-column fixed prop="id" label="操作编号" min-width="150"/>
        <el-table-column fixed prop="goodsId" label="货品Id" min-width="150"/>
        <el-table-column fixed prop="goodsName" label="货名" min-width="150"/>
        <el-table-column prop="size" label="大小" min-width="150"/>
        <el-table-column prop="amount" label="数量" min-width="150"/>
        <el-table-column prop="date" label="时间" min-width="150"/>
        <el-table-column prop="factory" label="厂商" min-width="150"/>
        <el-table-column prop="adminName" label="操作员" min-width="150"/>
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
        v-if="editData"
        class="outgoods-dialog"
        title="编辑入库记录"
        :visible.sync="editShow"
        width="30%">
        <el-form ref="editForm" :rules="rule" :model="editData" label-width="100px" size="mini" class="addOutgoods-form">
          <el-table-column fixed prop="goodsId" label="货品Id" min-width="150"/>
          <el-table-column fixed prop="goodsName" label="货名" min-width="150"/>
          <el-table-column prop="size" label="大小" min-width="150"/>
          <el-table-column prop="amount" label="数量" min-width="150"/>
          <el-table-column prop="date" label="日期" min-width="150"/>
          <el-table-column prop="factory" label="厂商" min-width="150"/>
          <el-table-column prop="adminName" label="操作员" min-width="150"/>

          <el-form-item label="货物Id" prop="goodsId">
            <el-input v-model="editData.goodsId" clearable></el-input>
          </el-form-item>
          <el-form-item label="货名" prop="goodsName">
            <el-input v-model="editData.goodsName" clearable></el-input>
          </el-form-item>
          <el-form-item label="大小" prop="size">
            <el-input v-model="editData.size" clearable></el-input>
          </el-form-item>
          <el-form-item label="数量" prop="amount">
            <el-input v-model="editData.amount" clearable></el-input>
          </el-form-item>
          <el-form-item label="日期" prop="date">
            <el-input v-model="editData.date" clearable></el-input>
          </el-form-item>
          <el-form-item label="厂商" prop="factory">
            <el-input v-model="editData.factory" clearable></el-input>
          </el-form-item>
          <el-form-item label="操作员" prop="adminName">
            <el-input v-model="editData.adminName" clearable></el-input>
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
export default class InGoods extends Vue {
  editShow = false;
  tableData = [];
  hasAuth = false;
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
    this.currentPage = id;
  }
  created() {
    this.getOutgoodss();
    this.hasAuth = (JSON.parse(window.sessionStorage.getItem('admin')).auth == 2)
  }
  getOutgoodss() {
    axios.get('http://localhost:9999/c/outgoods/list')
    .then((response) => {
      const { data } = response.data;
      this.tableData = data
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  deleteRow(obj) {
    this.$confirm('确认删除').then(() => {
      const config = {
        method: 'delete',
        url: 'http://localhost:9999/c/outgoods/delete',
        headers: { 'Content-Type': 'application/json' },
        data: obj
      };
      axios(config).then(() => {
        const loading = this.$loading({
          text: '正在加载中......',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)',
        })
        this.$message.success('删除成功');
        this.getOutgoodss();
        loading.close();
      }).catch((error) => {
        this.$message.error(error);
      });
    })
  }

  edit(obj) {
    this.editData = JSON.parse(JSON.stringify(obj));
    this.editShow = true;
  }
  confirmEdit() {
    this.$refs.editForm.validate((valid) => {
      if (valid) {
        axios.post('http://localhost:9999/c/outgoods/edit', this.editData).then(() => {
          const loading = this.$loading({
            text: '正在加载中......',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
          this.$message.success('编辑成功');
          this.editShow = false;
          this.getOutgoodss();
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
.outgoods {
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

