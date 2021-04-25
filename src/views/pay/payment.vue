<template>
  <div class="app-container">
    <el-button type="primary" icon="el-icon-edit" @click="handleCreate">新增收款账号</el-button>
    <el-table
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
    >
      <el-table-column
        label="账号"
        prop="account"
        width="600"
      >
      </el-table-column>
      <el-table-column label="开户人" align="center" prop="name" width="100">
      </el-table-column>
      <el-table-column label="开户行" prop="bank" align="center" width="300">
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)"> 编辑 </el-button>
          <el-popconfirm
             title="确定要删除该账号吗？"
             @onConfirm="handleDelete(row, $index)"
          >
            <el-button size="mini" type="danger" slot="reference">删除</el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />

    <!-- 弹出框 -->
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" >
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="100px" style="width: 400px; margin-left:50px;">
        <el-form-item label="账号" prop="account">
          <el-input v-model="temp.account" />
        </el-form-item>
        <el-form-item label="省市区">
            <Distpicker :province="temp.province" :city="temp.city" :area="
            temp.area" @province="handleDistPicker($event,'province')" @city="handleDistPicker($event,'city')" @area="handleDistPicker($event,'area')"></Distpicker>
        </el-form-item>
        <el-form-item label="详细地址" prop="path">
          <el-input v-model="temp.path" />
        </el-form-item>
        <el-form-item label="所属银行" prop="bank">
          <el-select v-model="temp.bank" placeholder="请选择">
            <el-option
                v-for="item in banks"
                :key="item.value"
                :label="item.text"
                :value="item.text">
                </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="姓名" prop="path">
          <el-input v-model="temp.name" />
        </el-form-item>
        
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          提交
        </el-button>
      </div>
    </el-dialog>
    
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import Distpicker from 'v-distpicker'
import {getBank} from '@/utils/bank'
import {   createPayment,fetchPayments,deletePayment,updatePayment } from "@/api/pay";
export default {
  mounted() {
    this.getList();
    this.banks=getBank()
  },
  components: { Pagination,Distpicker },
  computed: {},
  data() {
    return {
      dialogFormVisible: false,
      listLoading: false,
      total: 100,
      listQuery: {
        page: 1,
        limit: 20,
        title: undefined,
        status: undefined,
      },
      banks:[],
      tableData: [],
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '添加'
      },
      temp: {
        id: undefined,
        account: '',
        province:"省",
        city:"市",
        area:"区",
        path:"",
        name:"",
        bank:null
      },
      rules: {
        account:[{required: true, message: '账号不能为空', trigger: 'blur' },
        ],
        path:[{required: true, message: '地址不能为空', trigger: 'blur' },
        ],
        bank:[{required: true, message: '所属银行不能为空', trigger: 'change' },
        ],
        name:[{required: true, message: '姓名不能为空', trigger: 'blur' },
        ],
      },
    };
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        success: "success",
        fail: "danger",
        pendding: "danger",
      };
      return statusMap[status];
    },
  },
  methods: {
    handleDistPicker(e,k){
      this.temp[k] = e.value
    },
    //获取订单列表
    getList() {
      this.listLoading = true;
      fetchPayments(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //删除账号
    handleDelete(row, index) {
        deletePayment(row.id).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$notify({
              title: "提示",
              message: "删除成功",
              type: "success",
              duration: 2000,
            });
            this.tableData.splice(index, 1);
          } else {
            this.$notify({
              title: "提示",
              message: "删除失败,请重试",
              type: "error",
              duration: 2000,
            });
          }
        });
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        account: '',
        province:"省",
        city:"市",
        area:"区",
        path:"",
        name:"",
        bank:null
      }
    },
    //显示添加商品页面
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    //提交添加商品
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if(this.temp.province==='省'||this.temp.city==='市'||this.temp.area==='区'){
            this.$message.error('请先选择省市区')
            return false
          }
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          createPayment(this.temp).then(() => {
            this.tableData.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '添加成功',
              message: '添加收款账户成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    //点击编辑按钮
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    //提交修改
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if(this.temp.province==='省'||this.temp.city==='市'||this.temp.area==='区'){
            this.$message.error('请先选择省市区')
            return false
          }
          const tempData = Object.assign({}, this.temp)
          updatePayment(tempData).then(() => {
            const index = this.tableData.findIndex(v => v.id === this.temp.id)
            this.tableData.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '修改成功',
              message: '修改收款账号成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    }
    
  }
};
</script>
<style scoped>
table {
  margin-left: 10px;
}
.text {
  font-size: 26px;
  font-weight: bold;
}

.item {
  margin-bottom: 18px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}
</style>