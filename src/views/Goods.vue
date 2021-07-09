<template>
  <Nav>
    <div class="goods">
      <div class="goods-wrapper">
        <div class="goods-search">
          <el-input/>
          <el-button size="small" type="primary">查询</el-button>
        </div>
        <el-button size="small" type="primary" @click="showAddGoods = true" :disabled="!hasAuth">添加货物</el-button>
      </div>
      <el-table
        :data="partTableData"
        border
        stripe
        style="width: 100%">
        <el-table-column fixed prop="id" label="货品编号" min-width="150"/>
        <el-table-column fixed prop="name" label="货名" min-width="150"/>
        <el-table-column prop="size" label="尺码" min-width="150"/>
        <el-table-column prop="date" label="添加日期" min-width="150"/>
        <el-table-column prop="amount" label="数量" min-width="150"/>
        <el-table-column prop="factory" label="厂商" min-width="150"/>
        <el-table-column fixed="right" label="操作" min-width="200">
          <template slot-scope="scope" v-if="hasAuth">
            <el-button type="text" size="small" @click="edit(scope.row)">编辑</el-button>
            <el-button @click="deleteRow(scope.row)" type="text" size="small">删除</el-button>
            <el-button type="text" size="small" @click="flag = 1; inShow = true; inKData = JSON.parse(JSON.stringify(scope.row))">入库</el-button>
            <el-button type="text" size="small" @click="flag = 2; inShow = true; inKData = JSON.parse(JSON.stringify(scope.row))">出库</el-button>
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
        class="goods-dialog"
        title="添加货物"
        :visible.sync="showAddGoods"
        width="30%">
        <el-form ref="addForm" :rules="rule" :model="addForm" label-width="100px" size="mini" class="addGoods-form">
          <el-form-item label="货名" prop="name">
            <el-input v-model="addForm.name" clearable/>
          </el-form-item>
          <el-form-item label="尺码" prop="size">
            <el-input v-model="addForm.size" clearable/>
          </el-form-item>
          <el-form-item label="日期" prop="date">
            <el-input v-model="addForm.date" clearable/>
          </el-form-item>
          <el-form-item label="数量" prop="amount">
            <el-input v-model="addForm.amount" clearable/>
          </el-form-item>
          <el-form-item label="厂商" prop="factory">
            <el-input v-model="addForm.factory"/>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="showAddGoods = false" size="small">取 消</el-button>
          <el-button type="primary" @click="addGoods" size="small">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        v-if="editData"
        class="goods-dialog"
        title="编辑货物"
        :visible.sync="editShow"
        width="30%">
        <el-form ref="editForm" :rules="rule" :model="editData" label-width="100px" size="mini" class="addGoods-form">
          <el-form-item label="货名" prop="name">
            <el-input v-model="editData.name" clearable/>
          </el-form-item>
          <el-form-item label="尺码" prop="size">
            <el-input v-model="editData.size" clearable/>
          </el-form-item>
          <el-form-item label="日期" prop="date">
            <el-input v-model="editData.date" clearable/>
          </el-form-item>
          <el-form-item label="数量" prop="amount">
            <el-input v-model="editData.amount" clearable/>
          </el-form-item>
          <el-form-item label="厂商" prop="factory">
            <el-input v-model="editData.factory"/>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editShow = false" size="small">取 消</el-button>
          <el-button type="primary" @click="confirmEdit" size="small">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        v-if="inData"
        class="goods-dialog"
        title="入库信息"
        :visible.sync="inShow"
        width="30%">
        <el-form ref="inForm" :model="inData" label-width="100px" size="mini" class="addGoods-form">
          <el-form-item label="日期" prop="date">
            <el-input v-model="inData.date" clearable/>
          </el-form-item>
          <el-form-item label="数量" prop="amount">
            <el-input v-model="inData.amount" clearable/>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="inShow = false" size="small">取 消</el-button>
          <el-button type="primary" @click="inK" size="small">确 定</el-button>
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
export default class Goods extends Vue {
  hasAuth = false;
  showAddGoods = false;
  editShow = false;
  inShow = false;
  flag = null;
  addInit = {
    name: '',
    tel: '',
    gender:'',
    age:'',
  }
  tableData = [];
  addForm = this.addInit;
  editData = null;
  inKData = null;
  currentPage = 1;
  inInit = {
    amount: '',
    date: ''
  }
  inData = JSON.parse(JSON.stringify(this.inInit))
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
    ],
    amount: {
      required: true,
      message: '请输入数目',
      trigger: 'blur'
    },
    factory: {
      required: true,
      message: '请输入厂商',
      trigger: 'blur'
    },
    size: {
      required: true,
      message: '请输入尺码',
      trigger: 'blur'
    },
    date: {
      required: true,
      message: '请输入日期',
      trigger: 'blur'
    },
  }
  get partTableData() {
    return this.tableData.slice((this.currentPage-1)*8, Math.min(this.currentPage*8, this.tableData.length + 1));
  }
  handleCurrentChange(id) {
    console.log(id);
    this.currentPage = id;
  }
  created() {
    this.getGoods();
    this.hasAuth = (JSON.parse(window.sessionStorage.getItem('admin')).auth == 2)
  }
  getGoods() {
    axios.get('http://localhost:9999/c/goods/list')
    .then((response) => {
      const { data } = response.data;
      this.tableData = data
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  inK(){
    const obj = JSON.parse(JSON.stringify(this.inKData));
    const flag = this.flag;
    this.inShow = true;
    let tmp = {
      id: '',
      goodsId: obj.id,
      goodsName: obj.name,
      adminName: JSON.parse(window.sessionStorage.getItem('admin')).username,
      size: obj.size,
      amount: this.inData.amount,
      date: this.inData.date,
      factory: obj.factory
    }
    const url = (flag === 1 ? 'http://localhost:9999/c/ingoods/add' : 'http://localhost:9999/c/outgoods/add');
    const msg = (flag === 1 ? '入库成功' : '出库成功');
    flag !== 1 ? obj.amount = obj.amount - this.inData.amount : obj.amount = parseInt(obj.amount) + parseInt(this.inData.amount);
    if(window.localStorage.getItem('numIn') == null) {
      window.localStorage.setItem('numIn', '0');
    }
    if(window.localStorage.getItem('numOut') == null) {
      window.localStorage.setItem('numOut', '0');
    }
    let idss;
    if(flag === 1) {
      idss = parseInt(JSON.parse(window.localStorage.getItem('numIn')));
      idss += 1;
      window.localStorage.setItem('numIn', idss);
    } else {
      idss = parseInt(JSON.parse(window.localStorage.getItem('numOut')));
      idss += 1;
      window.localStorage.setItem('numOut', idss);
    }
    tmp.id = idss;
    const config = {
      method: 'post',
      url: url,
      headers: { 'Content-Type': 'application/json' },
      data: tmp
    };
    axios(config).then(() => {
      const loading = this.$loading({
        text: '正在加载中......',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)',
      })
      this.$message.success(msg);
      this.inData = JSON.parse(JSON.stringify(this.inInit))
      axios.post('http://localhost:9999/c/goods/edit', obj).then(() => {
        this.inShow = false;
        this.getGoods();
      }).catch((err) => {
        this.$message.error(err);
      })
      loading.close();
    }).catch((error) => {
      this.$message.error(error);
    });
  }
  deleteRow(id) {
    this.$confirm('确认删除').then(() => {
      const config = {
        method: 'delete',
        url: 'http://localhost:9999/c/goods/delete',
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
        this.getGoods();
        loading.close();
      }).catch((error) => {
        this.$message.error(error);
      });
    })
  }
  addGoods() {
    this.$refs.addForm.validate((valid) => {
      if (valid) {
        let ids = 1;
        const arr = [];
        this.tableData.forEach(item => {arr.push(parseInt(item.id))});
        while(arr.includes(ids))ids++;
        const data = Object.assign(this.addForm,{id: ids.toString()});
        axios.post('http://localhost:9999/c/goods/insert', data).then(() => {
          const loading = this.$loading({
            text: '正在加载中......',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
          this.$message.success('添加成功');
          this.addForm = this.addInit;
          this.showAddGoods = false;
          this.getGoods();
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
        axios.post('http://localhost:9999/c/goods/edit', this.editData).then(() => {
          const loading = this.$loading({
            text: '正在加载中......',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
          this.$message.success('编辑成功');
          this.editShow = false;
          this.getGoods();
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
.goods {
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



