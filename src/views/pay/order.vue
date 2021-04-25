<template>
  <div class="app-container">
    <el-button type="primary" :loading="downloadLoading" icon="el-icon-edit" @click="handleDownload">导出选中</el-button>

    <el-input
      v-model="listQuery.title"
      placeholder="请输入关键词"
      style="width: 200px; margin-left: 850px"
      class="filter-item"
    />
    <el-button
      class="filter-item"
      type="primary"
      icon="el-icon-search"
      @click="handleFilter"
    >
      搜索
    </el-button>
    <el-table
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        type="selection"
        width="50"
        align="center"
      ></el-table-column>
      <el-table-column label="订单号" prop="id" width="100">
      </el-table-column>
      <el-table-column
        prop="goods[0].title"
        label="商品名称"
        width="300"
      >
      </el-table-column>
      <el-table-column
        align="center"
        label="订单类型"
        width="100"
      >
        <template slot-scope="{ row }">
          {{row.type==='group'?'拼团':'普通'}}
        </template>
      </el-table-column>
      <el-table-column label="订单状态" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status | statusFilter">
            <!-- {{ row.status ? "已上架" : "已下架" }} -->
            {{row.status |statustext}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="原件/实付（元）" align="center"  width="200">
        <template slot-scope="{ row }">
          {{row.total_price}}/<span style="color:red">{{row.price}}</span>
        </template>
      </el-table-column>
      <el-table-column label="支付方式" align="center"  width="100">
        <template slot-scope="{ row }">
          {{row.pay_method==="wechat"?"微信支付":"支付宝支付"}}
        </template>
      </el-table-column>
      <el-table-column label="创建/支付时间" align="center"  width="200">
        <template slot-scope="{ row }">
          <p>{{row.created_time}}</p>
          <p>{{row.pay_time}}</p>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        
        <template slot-scope="{ row, $index }">
          <el-popconfirm
             title="确定要删除该订单吗？"
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
    
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import {fetchOrder,deleteOrder } from "@/api/pay";
import Tinymce from "@/components/Tinymce";
import Dropzone from "@/components/Dropzone";
let statusOptions = {
        pendding:"待支付",
        success:"成功",
        fail:"失败"
    }
export default {
  mounted() {
    this.getList();
  },
  components: { Pagination, Tinymce, Dropzone },
  computed:{
  },
  data() {
    return {
      multipleSelection: [], //选中的用户
      downloadLoading: false,
      listLoading: false,
      total: 100,
      listQuery: {
        page: 1,
        limit: 20,
        title: undefined,
        status: undefined,
      },
     tempCourseQuery: {
        page: 1,
        limit: 20,
      },
      tableData: [],
    };
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        success: "success",
        fail: "danger",
        pendding:"danger"
      };
      return statusMap[status];
    },
    statustext(status) {
      const statusMap = {
        success: "成功",
        fail: "失败",
        pendding:"待支付"
      };
      return statusMap[status];
    },
  },
  methods: {
    //筛选搜索
    handleFilter() {
      this.listQuery.page = 1;
      console.log(this.listQuery);
      this.getList(this.listQuery);
    },
    //获取订单列表
    getList() {
      this.listLoading = true;
      fetchOrder(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //获取选中获取用户
    handleSelectionChange(val) {
        this.multipleSelection = val; //获取多选用户    
    },
    //导出选中用户
    handleDownload(){
      if(this.multipleSelection.length){
        import('@/vendor/Export2Excel').then(excel => {
          const tHeader = ['订单号', '商品名称', '订单类型', '订单状态', '原价（元）','实付（元）','支付方式','创建时间','支付时间']
          const filterVal = ['id', 'title', 'type', 'status', 'total_price','price','pay_method','created_time','pay_time']
          const list = this.multipleSelection.map(item=>{
            let o={...item}
            o.title=item.goods[0].title
            o.type=item.type==='group'?'拼团':'普通'
            o.status=statusOptions[item.status]
            o.pay_method =item.pay_method==="wechat"?"微信支付":"支付宝支付"
            return o
          })
          console.log(list)
          const data = this.formatJson(filterVal,list)
          excel.export_json_to_excel({
            header: tHeader,
            data,
            // filename: this.filename
          })
          this.downloadLoading = false
        })
        // this.multipleSelection=[] //清除
      }else{
        this.$message({
          message: '请选择导出的订单',
          type: 'warning'
        })
      }
     
      
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    },
    //删除商品
    handleDelete(row, index) {
        console.log(1111);
        deleteOrder(row.id).then((response) => {
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


  },
};
</script>
<style scoped>
table {
  margin-left: 10px;
}
</style>