<template>
  <div class="app-container">
    <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
      >创建拼团</el-button
    >
    <el-table
      :data="tableData"
      style="width: 100%; margin-top: 30px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
    >
      <el-table-column label="拼团内容" width="800">
        <template slot-scope="{ row }">
          <img
            :src="row.value.cover"
            width="100"
            height="50"
            style="float: left; margin-right: 10px"
          />
          <p style="margin-top: 0; margin-bottom: 0">{{ row.value.title }}</p>
          <p style="margin-top: 0; margin-bottom: 0">
            原始价格：￥<span style="color: red">{{ row.price }}</span>
          </p>
          <p style="margin-top: 0; margin-bottom: 0">
            拼团价格：￥<span style="color: red">{{ row.value.price }}</span>
          </p>
        </template>
      </el-table-column>
      <el-table-column label="拼团人数" prop="p_num" align="center" width="80">
      </el-table-column>

      <el-table-column
        label="拼团时限（小时）"
        prop="expire"
        align="center"
        width="100"
      >
      </el-table-column>
      <el-table-column label="拼团状态" align="center" width="80">
        <template slot-scope="{ row }">
          <p
            :style="{
              color: row.status
                ? row.groupstatu === '拼团中'
                  ? 'red'
                  : ''
                : '',
            }"
          >
            {{ row.status ? row.groupstatu : "已下架" }}
          </p>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row }">
          <el-button
            type="primary"
            style="margin-right: 10px"
            size="mini"
            @click="handleUpdate(row)"
          >
            编辑
          </el-button>
          <el-button
            :disabled="row.status != 1"
            size="mini"
            type="danger"
            @click="handleModifyStatus(row, 0)"
          >
            下架
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

    <!-- 弹出框 -->
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="类型" prop="type">
          <el-select v-model="temp.type">
            <el-option
              v-for="item in types"
              :key="item.key"
              :label="item.text"
              :value="item.key"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="账号" prop="value">
          <el-button type="primary" @click="connectValue">关联</el-button>
          <el-card v-if="temp.value">
            <img :src="temp.value.cover" :width="200" :height="100" alt="" />
            <p>{{ temp.value.title }}</p>
            <p style="color: red">{{ temp.value.price }}</p>
          </el-card>
        </el-form-item>
        <el-form-item label="拼团价" prop="price">
          <el-input-number :min="0" v-model="temp.price"></el-input-number>
        </el-form-item>
        <el-form-item label="拼团人数" prop="p_num">
          <el-input-number :min="2" v-model="temp.p_num"></el-input-number>
        </el-form-item>
        <el-form-item label="是否开启拼团 ">
          <el-radio-group v-model="temp.status">
            <el-radio :label="0"> 否 </el-radio>
            <el-radio :label="1"> 是 </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="拼团时限">
          <el-radio-group v-model="temp.expire">
            <el-radio :label="24"> 24小时 </el-radio>
            <el-radio :label="48"> 48小时 </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="拼团活动开始时间-结束时间">
          <div class="block">
            <span class="demonstration">起始日期时刻为 12:00:00</span>
            <el-date-picker
              v-model="timerange"
              type="datetimerange"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :default-time="['12:00:00']"
            >
            </el-date-picker>
          </div>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          提交
        </el-button>
      </div>
    </el-dialog>

    <!-- 关联课程弹出框 -->
    <choose-course ref="chooseCourse"></choose-course>
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import { fetchGroup, updateGroup,createGroup } from "@/api/marketing";
import chooseCourse from "@/components/chooseCourse/index.vue";
export default {
  mounted() {
    this.getList();
  },
  components: { Pagination, chooseCourse },
  computed: {
    timerange: {
      get() {
        return [this.temp.start_time, this.temp.end_time];
      },
      set(val) {
        this.temp.start_time = val[0];
        this.temp.end_time = val[1];
      },
    },
  },
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
      types: [
        { key: "course", text: "课程" },
        { key: "column", text: "专栏" },
      ],
      tableData: [],
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "添加",
      },
      temp: {
        id: undefined,
        type: "course",
        value: null,
        price: 0,
        p_num: 2,
        status: 1,
        expire: 48,
        start_time: "",
        end_time:""
      },
      rules: {
        type: [{ required: true, message: "类型不能为空", trigger: "change" }],
        value: [{ required: true, message: "关联课程/专栏不能为空" }],
        price: [{ required: true, message: "拼团价不能为空", trigger: "blur" }],
        p_num: [
          { required: true, message: "拼团人数不能为空", trigger: "blur" },
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
    // 关联课程/专栏
    connectValue() {
      this.$refs.chooseCourse.open((val) => {
        let item = val[0];
        this.temp.value = {
          id: item.id,
          title: item.title,
          cover: item.cover,
          price: item.price,
        };
        this.$refs.dataForm.clearValidate();
      }, 1);
    },
    //获取拼团列表
    getList() {
      this.listLoading = true;
      fetchGroup(this.listQuery).then((response) => {
        // console.log(response.data);
        let nowTime = new Date().getTime();
        this.tableData = response.data.items.map((item) => {
          let o = { ...item };
          if (o.status === 1) {
            let date1 = new Date(item.start_time);
            let startTime = date1.getTime();
            let date2 = new Date(item.end_time);
            let endTime = date2.getTime();
            if (startTime > nowTime) {
              o.groupstatu = "未开始";
            } else {
              if (endTime < nowTime) {
                o.groupstatu = "已结束";
              } else {
                o.groupstatu = "拼团中";
              }
            }
          }
          return o;
        });
        this.total = response.data.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    //修改商品状态
    handleModifyStatus(row, status) {
      this.$confirm("此操作将下架该商品, 是否继续?", "提示", {
        confirmButtonText: "下架",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        updateGroup(row.status).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$message({
              message: "下架成功",
              type: "success",
            });
            row.status = status;
          } else {
            this.$message({
              message: "下架失败",
              type: "error",
            });
          }
        });
      });
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        type: "course",
        // value:{cover:"",price:"",title:""},
        value: null,
        price: 0,
        p_num: 2,
        status: 1,
        expire: 48,
        start_time: "",
        end_time:""
      };
    },
    //显示添加商品页面
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    //提交添加商品
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
          createGroup(this.temp).then(() => {
            this.tableData.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "创建成功",
              message: "创建拼团成功",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    //点击编辑按钮
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    //提交修改
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          updateGroup(tempData).then(() => {
            const index = this.tableData.findIndex(
              (v) => v.id === this.temp.id
            );
            this.tableData.splice(index, 1, this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "修改成功",
              message: "修改拼团信息成功",
              type: "success",
              duration: 2000,
            });
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