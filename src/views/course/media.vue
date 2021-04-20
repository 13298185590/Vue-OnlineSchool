<template>
  <div class="app-container">
    <el-button type="primary" icon="el-icon-edit">新增图文</el-button>

    <el-select
      v-model="listQuery.status"
      placeholder="商品状态"
      clearable
      class="filter-item"
      style="width: 130px; margin-left: 700px"
      @change="handleFilter"
    >
      <el-option
        v-for="item in calendarTypeOptions"
        :key="item.key"
        :value="item.display_name"
      />
    </el-select>
    <el-input
      v-model="listQuery.title"
      placeholder="请输入关键词"
      style="width: 200px; margin-left: 10px"
      class="filter-item"
    />
    <el-button
      v-waves
      class="filter-item"
      type="primary"
      icon="el-icon-search"
      @click="handleFilter"
    >
      Search
    </el-button>
    <el-table
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border="true"
    >
      <el-table-column prop="id" label="ID" align="center" sortable width="100">
      </el-table-column>
      <el-table-column label="图文内容">
        <template slot-scope="{ row }">
          <img :src="row.cover" width="200" height="100" style="float: left" />
          <p>{{ row.title }}</p>
          <p style="color: red">￥{{ row.price }}</p>
        </template>
      </el-table-column>
      <el-table-column
        prop="sub_count"
        align="center"
        label="订阅量"
        width="100"
      >
      </el-table-column>
      <el-table-column label="状态" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status | statusFilter">
            {{ row.status ? "已下架" : "已上架" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="created_time"
        align="center"
        label="创建时间"
        width="200"
      >
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="280"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini"> 编辑 </el-button>
          <el-button
            v-if="row.status != '0'"
            size="mini"
            @click="handleModifyStatus(row, '0')"
          >
            下架
          </el-button>
          <el-button
            v-if="row.status != '1'"
            size="mini"
            type="success"
            @click="handleModifyStatus(row, '1')"
          >
            上架
          </el-button>
          <el-button
            v-if="row.status != 'deleted'"
            size="mini"
            type="danger"
            @click="handleDelete(row, $index)"
          >
            删除
          </el-button>
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
import { fetchList, updateMedia, deleteMedia } from "@/api/media";
export default {
  mounted() {
    this.getList();
  },
  components: { Pagination },
  data() {
    return {
      listLoading: false,
      total: 100,
      listQuery: {
        page: 1,
        limit: 20,
        title: undefined,
        status: undefined,
      },
      calendarTypeOptions: [
        { key: "CN", display_name: "已下架" },
        { key: "US", display_name: "已上架" },
      ],
      tableData: [],
    };
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        1: "success",
        0: "danger",
      };
      return statusMap[status];
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type];
    },
  },
  methods: {
    formatter(row, column) {
      return row.address;
    },
    //筛选搜索
    handleFilter() {
      this.listQuery.page = 1;
      console.log(this.listQuery);
      this.getList(this.listQuery);
    },
    //获取商品列表
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //修改商品状态
    handleModifyStatus(row, status) {
      updateMedia(row.status).then((response) => {
        // console.log(response)
        if ((response.code = 20000)) {
          this.$message({
            message: "修改商品状态成功",
            type: "success",
          });
          row.status = status;
        } else {
          this.$message({
            message: "修改商品状态失败",
            type: "error",
          });
        }
      });
    },
    //删除商品
    handleDelete(row, index) {
      this.$confirm("此操作将永久删除该商品, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        deleteMedia(row.id).then((response) => {
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