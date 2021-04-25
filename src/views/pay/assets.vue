<template>
  <div class="app-container">
    <el-row :gutter="20">
      <el-col :span="6">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>可用余额（元）</span>
            <el-button style="float: right; padding: 3px 0;width:80px;height:30px" type="primary"
               @click="handleAssets">申请提现</el-button>
          </div>
          <div class="text item">20.00</div>
        </el-card></el-col
      >
      <el-col :span="6">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>累计收入（元）</span>
          </div>
          <div class="text item">60.00</div>
        </el-card></el-col
      >
      <el-col :span="6">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>待结算余额（元）</span>
          </div>
          <div class="text item">21.00</div>
        </el-card></el-col
      >
      <el-col :span="6">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>冻结金额（元）</span>
          </div>
          <div class="text item">0.00</div>
        </el-card></el-col
      >
    </el-row>

    <el-table
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
    >
      <el-table-column
        label="交易时间"
        align="center"
        prop="created_time"
        width="200"
      >
      </el-table-column>
      <el-table-column
        label="提款账号"
        align="center"
        prop="account"
        width="600"
      >
      </el-table-column>
      <el-table-column label="开户人" align="center" prop="name" width="100">
      </el-table-column>
      <el-table-column label="提现金额" align="center" width="300">
        <template slot-scope="{ row }"> ￥{{ row.price }} </template>
      </el-table-column>
      <el-table-column align="center" label="状态">
        <template slot-scope="{ row }">
          <el-tag :type="row.status ? 'success' : 'danger'">
            {{ row.status ? "提现完成" : "审核中" }}
          </el-tag>
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

    <!-- 申请提现弹出框 -->
    <el-dialog title="提现" :visible.sync="dialogFormVisible" >
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="100px" style="width: 400px; margin-left:50px;"> 
        <el-form-item label="提现金额"  prop="price">
          <el-input-number v-model="temp.price" :min="0" ></el-input-number>
        </el-form-item>
        <el-form-item label="提现账户"  prop="accout">
          <el-select
      v-model="temp.account"
      placeholder="请选择提现账户"

    >
      <el-option
                v-for="item in payments"
                :key="item.id"
                :label="item.bank"
                :value="item.id">
                <span style="float: left">{{ item.bank }}</span>
                <span style="float: right; color: #8492a6; font-size: 13px">{{ item.account }}</span>
                </el-option>
    </el-select>
        </el-form-item>
        
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="updateData()">
          提交
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import { fetchAssets,  Cashout,fetchPayments } from "@/api/pay";
export default {
  mounted() {
    this.getList();
    fetchPayments().then(response=>{
      // console.log(response);
      this.payments=response.data.items
    })
  },
  components: { Pagination },
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
      tableData: [],
      temp: {
        price:0,
        account:''
      },
      payments:[], //提现人账号
      rules: {
        price: [{ required: true, message: '提现金额必填', trigger: 'blur' },
                {type:'number',max:20,message:"提现金额不能大于20元",trigger:"blur"}
        ],
        account:[{ required: true, message: '提现账户不能为空', trigger: 'change' },
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
    //获取订单列表
    getList() {
      this.listLoading = true;
      fetchAssets(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //显示申请提现弹框
    handleAssets() {
      this.dialogFormVisible = true
    },
    //提交修改
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
           Cashout(tempData).then(() => {
            this.dialogFormVisible = false
            this.$notify({
              title: '成功',
              message: '提现申请成功,等待审核',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    //
    
  },
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