<template>
  <div class="app-container">
    <el-button type="primary" icon="el-icon-edit" @click="handleCreate">新增视频</el-button>

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
      border
    >
      <el-table-column prop="id" label="ID" align="center" sortable width="100">
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
          <el-button type="primary" size="mini" @click="handleUpdate(row)"> 编辑 </el-button>
          <el-button
            v-if="row.status != '0'"
            size="mini"
            @click="handleModifyStatus(row, 0)"
          >
            下架
          </el-button>
          <el-button
            v-if="row.status != '1'"
            size="mini"
            type="success"
            @click="handleModifyStatus(row, 1)"
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

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" fullscreen>
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="封面" prop="cover">
          <dropzone id="myVueDropzone" url="https://httpbin.org/post"  />
        </el-form-item>
        <el-form-item label="试看内容" prop="try">
          <tinymce v-model="temp.try" :height="300" :width="600" />
        </el-form-item>
        <el-form-item label="课程内容" prop="content">
          <tinymce v-model="temp.content" :height="300" :width="600" />
        </el-form-item>
        <el-form-item label="视频内容" >
          <el-upload
            class="upload-demo"
            action="https://jsonplaceholder.typicode.com/posts/"
          >
          <el-button size="small" type="primary">上传视频</el-button>
          <div slot="tip" class="el-upload__tip">支持mp4，avi，wmv，mov，flv，rmvb，3gp，m4v，mkv格式；文件最大不超过5G。当前店铺版本最大支持720高清转码</div>
          </el-upload>
        </el-form-item>
        <el-form-item label="课程价格" prop="price">
          <el-input-number v-model="temp.price"></el-input-number>
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="temp.status">
        <el-radio :label="0">
          下架
        </el-radio>
        <el-radio :label="1">
          上架
        </el-radio>
          </el-radio-group>
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
import { fetchList, updateVideo, deleteVideo,createVideo } from "@/api/video";
import Tinymce from '@/components/Tinymce'
import Dropzone from '@/components/Dropzone'
export default {
  mounted() {
    this.getList();
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
      fetchList(this.listQuery).then((response) => {
        console.log(response.data);
        this.tableData = response.data.items;
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //修改状态
    handleModifyStatus(row, status) {
      updateVideo(row.status).then((response) => {
        // console.log(response)
        if ((response.code = 20000)) {
          this.$message({
            message: "修改视频状态成功",
            type: "success",
          });
          row.status = status;
        } else {
          this.$message({
            message: "修改视频状态失败",
            type: "error",
          });
        }
      });
    },
    //删除商品
    handleDelete(row, index) {
      this.$confirm("此操作将永久删除该视频, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        deleteVideo(row.id).then((response) => {
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
          createVideo(this.temp).then(() => {
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
          updateVideo(tempData).then(() => {
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
  },
};
</script>
<style scoped>
table {
  margin-left: 10px;
}
</style>