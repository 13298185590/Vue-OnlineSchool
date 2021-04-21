<template>
  <div class="app-container">
    <el-card class="box-card" :gutter="1">
      <el-col :span="5">
        <img :src="detaDetail.cover" width="200" height="100"  />
      </el-col>
      <el-col :span="18">
         <h3 style="margin:0">{{ detaDetail.title }}</h3>
        <p style="color:#77787b;float: right;margin-top:-20px">{{detaDetail.isend?"已完结":"连载中"}}</p>
          <p style="color:#77787b">{{ detaDetail.content  }}</p>
          <p style="color: red">￥{{ detaDetail.price }}</p>
          <el-button
            v-if="detaDetail.status != '0'"
            size="mini"
            type="warning"
            @click="handleModifyStatus(detaDetail, 0)"
          >
            下架
          </el-button>
          <el-button
            v-if="detaDetail.status != '1'"
            size="mini"
            type="warning"
            @click="handleModifyStatus(detaDetail, 1)"
          >
            上架
          </el-button>
          <el-button
            v-if="detaDetail.isend != '0'"
            size="mini"
            @click="handleModifyIsend(detaDetail, 0)"
          >
            设为连载中
          </el-button>
          <el-button
            v-if="detaDetail.isend != '1'"
            size="mini"
            @click="handleModifyIsend(detaDetail, 1)"
          >
            设为已完结
          </el-button>
      </el-col>
         
    </el-card>
    <el-button type="primary" icon="el-icon-edit" @click="handleCreate">新增目录</el-button>
    <el-button type="primary"  @click="$router.back()">返回专栏</el-button>

    <el-select
      v-model="listQuery.status"
      placeholder="商品状态"
      clearable
      class="filter-item"
      style="width: 130px; margin-left: 550px"
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
      ref="dragTable"
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      highlight-current-row
      row-key="id"
    >
      <el-table-column  type="index"
      :index="indexMethod"  align="center"  width="80">
      </el-table-column>
      <el-table-column label="视频内容">
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
            {{ row.status ? "已上架" : "已下架" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="180"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)"> 编辑 </el-button>
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
      <el-table-column align="center" label="Drag" width="80">
        <template slot-scope="{}">
          <svg-icon class="drag-handler" icon-class="drag" />
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

    <el-dialog title="选择课程" :visible.sync="dialogFormVisible"  >
      <el-input
      v-model="listQuery.title"
      placeholder="请输入关键词"
      style="width: 200px; margin-left: 400px;margin-bottom:10px"
      class="filter-item"
      />
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
      搜索
    </el-button>
    <el-tabs tab-position="left" style="height: 200px;">
    <el-tab-pane label="图文">
      <el-table
      ref="dragTable"
      :data="tableData"
      style="width: 100%;"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="500"
      highlight-current-row
      row-key="id"
    >
      <el-table-column  type="selection" width="80" align="center">
      </el-table-column>
      <el-table-column label="图文内容">
        <template slot-scope="{ row }">
          <img :src="row.cover" :width="100" :height="50" style="float: left" />
          <p style="margin:0">{{ row.title }}</p>
          <p style="color: red;margin-top:0">￥{{ row.price }}</p>
        </template>
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="音频">
      <el-table
      ref="dragTable"
      :data="tableData"
      style="width: 100%;"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="500"
      highlight-current-row
      row-key="id"
    >
      <el-table-column  type="selection" width="80" align="center">
      </el-table-column>
      <el-table-column label="音频内容">
        <template slot-scope="{ row }">
          <img :src="row.cover" :width="100" :height="50" style="float: left" />
          <p style="margin:0">{{ row.title }}</p>
          <p style="color: red;margin-top:0">￥{{ row.price }}</p>
        </template>
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="视频">
      <el-table
      ref="dragTable"
      :data="tableData"
      style="width: 100%;"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="500"
      highlight-current-row
      row-key="id"
    >
      <el-table-column  type="selection" width="80" align="center">
      </el-table-column>
      <el-table-column label="视频内容">
        <template slot-scope="{ row }">
          <img :src="row.cover" :width="100" :height="50" style="float: left" />
          <p style="margin:0">{{ row.title }}</p>
          <p style="color: red;margin-top:0">￥{{ row.price }}</p>
        </template>
      </el-table-column>
      </el-table>
    </el-tab-pane>
    </el-tabs>
      
       <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
      />
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
import { fetchList, updateColumn, deleteColumn,createColumn,fetchDetailCourse,fetchDetail } from "@/api/column";
import Tinymce from '@/components/Tinymce'
import Dropzone from '@/components/Dropzone'
import Sortable from 'sortablejs'
export default {
  mounted() {
    this.getList();
    this.getDetail()
  },
  components: { Pagination,Tinymce,Dropzone },
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
      detaDetail:{},//详情数据
      calendarTypeOptions: [
        { key: "CN", display_name: "已下架" },
        { key: "US", display_name: "已上架" },
      ],
      tableData: [],
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '添加'
      },
      temp: {
        id: undefined,
        title: '',
        status: 1,
        cover:"",
        try:"",
        content:"",
        price:0
      },
      rules: {
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
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
      fetchDetailCourse(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
        this.oldList = this.tableData.map(v => v.id)
        this.newList = this.oldList.slice()
        this.$nextTick(() => {
          this.setSort()
      })
      });
    },
    //获取课程详情
    getDetail() {
      this.listLoading = true;
      let id=this.$route.query.id;
      fetchDetail({id}).then((response) => {
        // console.log(response);
        this.detaDetail = response.data
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //删除商品
    handleDelete(row, index) {
      this.$confirm("此操作将永久删除该视频, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        deleteColumn(row.id).then((response) => {
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
    resetTemp() {
      this.temp = {
        id: undefined,
        title: '',
        status: 1,
        cover:'',
        try:"",
        content:"",
        price:0
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
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          createColumn(this.temp).then(() => {
            this.tableData.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '添加成功',
              message: '添加视频成功',
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
          const tempData = Object.assign({}, this.temp)
          updateColumn(tempData).then(() => {
            const index = this.tableData.findIndex(v => v.id === this.temp.id)
            this.tableData.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '修改成功',
              message: '修改视频信息成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    //表格拖拽
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: evt => {
          const targetRow = this.tableData.splice(evt.oldIndex, 1)[0]
          this.tableData.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        }
      })
    },
    //表格索引
    indexMethod(index) {
        return ++index;
    },
    //修改状态
    handleModifyStatus(row, status) {
      updateColumn(row.status).then((response) => {
        // console.log(response)
        if ((response.code = 20000)) {
          this.$message({
            message: "修改专栏状态成功",
            type: "success",
          });
          row.status = status;
        } else {
          this.$message({
            message: "修改专栏状态失败",
            type: "error",
          });
        }
      });
    },
    //修改状态
    handleModifyIsend(row, status) {
      updateColumn(row.isend).then((response) => {
        // console.log(response)
        if ((response.code = 20000)) {
          this.$message({
            message: "修改更新状态成功",
            type: "success",
          });
          row.isend = status;
        } else {
          this.$message({
            message: "修改更栏状态失败",
            type: "error",
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
.box-card{
  margin-bottom: 10px;
}
</style>