<template>
  <div class="app-container">
    <el-dropdown @command="changeAllAccess">
      <el-button el-button type="danger">
        黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
      </el-button>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item>禁止评论</el-dropdown-item>
        <el-dropdown-item command="b">禁止访问</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
    <el-input
      v-model="listQuery.user.usernamer"
      placeholder="请输入用户名"
      style="width: 200px; margin-left: 800px"
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
      <el-table-column label="用户内容">
        <template slot-scope="{ row }">
          <img
            :src="row.user.avatar"
            width="50"
            height="50"
            style="float: left; border-radius: 25px; margin-right: 10px"
          />
          <p>{{ row.user.username }}</p>
        </template>
      </el-table-column>
      <el-table-column
        prop="total_consume"
        align="center"
        label="消费总额"
        width="100"
      >
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
        width="350"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUserDetail(row)">
            详情
          </el-button>
          <el-button size="mini" type="success" style="margin-right:10px"> 联系用户 </el-button>
          <el-dropdown @command="changeAccess(row)">
            <el-button size="mini" el-button type="danger">
              黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>{{row.no_comment?"允许评论":"禁止评论"}}</el-dropdown-item>
              <el-dropdown-item  command="b">{{row.no_access?"允许访问":"禁止访问"}}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
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
    <el-dialog
      title="用户详情"
      :visible.sync="dialogFormVisible"
      width="70%"
    >
      <el-row style="margin-bottom:30px">
        <el-col :span="6"><div>ID：{{this.temp.id}}</div></el-col>
        <el-col :span="6"><div>用户名：{{this.temp.username}}</div></el-col>
        <el-col :span="6"><div>昵称：{{this.temp.nickname?this.temp.nickname:'未设置'}}</div></el-col>
        <el-col :span="6"><div>手机号:{{this.temp.phone}}</div></el-col>
      </el-row>
      <el-row>
        <el-col :span="6"><div>锁定：{{this.temp.status?"是":"否"}}</div></el-col>
        <el-col :span="6"><div>会员：{{this.temp.user_level}}</div></el-col>
        <el-col :span="6"><div>会员到期时间：{{this.temp.user_level_expire}}</div></el-col>
        <el-col :span="6"><div>注册时间:{{this.temp.created_time}}</div></el-col>
      </el-row>
      <el-tabs  style="height: 350px;"  @tab-click="handleTabClick">
    <el-tab-pane label="课程订阅">
      <el-table
      ref="dataForm"
      :data="tempCourse"
      style="width: 100%;height: 200px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="250"
      highlight-current-row
      row-key="id"
    >
      <el-table-column label="课程标题" prop="title">
      </el-table-column>
      <el-table-column label="购买价格" prop="price">
      </el-table-column>
      <el-table-column label="购买时间" prop="created_time">
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="专栏订阅">
      <el-table
      ref="dataForm"
      :data="tempColumn"
      style="width: 100%;height: 200px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="250"
      highlight-current-row
      row-key="id"
    >
      <el-table-column label="专栏标题" prop="title">
      </el-table-column>
      <el-table-column label="购买价格" prop="price">
      </el-table-column>
      <el-table-column label="购买时间" prop="created_time">
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="订单记录">
      <el-table
      ref="dataForm"
      :data="tempOrder"
      style="width: 100%;height: 200px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="250"
      highlight-current-row
      row-key="id"
    >
      <el-table-column label="ID" prop="id">
      </el-table-column>
      <el-table-column label="订单号" prop="no">9
      </el-table-column>
      <el-table-column label="购买价格" prop="price">
      </el-table-column>
      <el-table-column label="状态">
        <template slot-scope="{ row }">
          {{row.status==="pendding"?'待支付':row.status==='success'?'成功':'失败'}}
        </template>
      </el-table-column>
      <el-table-column label="商品" prop="title">
      </el-table-column>
      <el-table-column label="购买时间" prop="created_time">
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="观看历史">
      <el-table
      ref="dataForm"
      :data="tempHistory"
      style="width: 100%;height: 200px"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="250"
      highlight-current-row
      row-key="id"
    >
      <el-table-column label="课程标题" prop="title">
      </el-table-column>
      <el-table-column label="课程类型">
        <template slot-scope="{ row }">
          {{types(row.type)}}
        </template>
      </el-table-column>
      <el-table-column label="学习时长" prop="total_time">
      </el-table-column>
      </el-table>
    </el-tab-pane>
    <el-tab-pane label="用户评论">
      <el-table
      ref="dataForm"
      :data="tempComment"
      style="width: 100%"
      :default-sort="{ prop: 'date', order: 'descending' }"
      border
      :height="250"
      highlight-current-row
      row-key="id"
    >
      <el-table-column label="评论内容" prop="content">
      </el-table-column>
      <el-table-column label="评论时间" prop="created_time">
      </el-table-column>
      <el-table-column label="课程标题" prop="title">
      </el-table-column>
      <el-table-column label="类型">
        <template slot-scope="{ row }">
          {{types(row.type)}}
        </template>
      </el-table-column>
      
      </el-table>
    </el-tab-pane>
    </el-tabs>
      <pagination
      style="margin-top:0"
      v-show="total > 0"
      :total="total"
      :page.sync="tempCourseQuery.page"
      :limit.sync="tempCourseQuery.limit"
      @pagination="getDetailList"
    />
    </el-dialog>
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import { updateAudio, deleteAudio, createAudio } from "@/api/audio";
import { fetchList,changeCommentStatus,changeAccessStatus,fetchUserDetail,fetchUserCourse,fetchUserColumn,fetchUserOrder,fetchUserHistory,fetchUserComment } from "@/api/user";
import Tinymce from "@/components/Tinymce";
import Dropzone from "@/components/Dropzone";
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
      listLoading: false,
      total: 100,
      listQuery: {
        page: 1,
        limit: 20,
        user:{
          username:""
        }
      },
     tempCourseQuery: {
        page: 1,
        limit: 20,
      },
      tableData: [],
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "添加",
      },
      temp: {}, //用户详情,
      tempCourse:[],//用户的课程订阅
      tempColumn:[],//用户的专栏订阅
      tempOrder:[],//用户的订单记录
      tempHistory:[],//用户的观看历史
      tempComment:[],//用户的用户评论
    };
  },
  methods: {
    types(type){
      if(type==='video'){
        return "视频"
      }
      if(type==='audio'){
        return "音频"
      }
      if(type==='media'){
        return "图文"
      }
      if(type==='column'){
        return "专栏"
      }
    },
    //筛选搜索
    handleFilter() {
      this.listQuery.page = 1;
      console.log(this.listQuery);
      this.getList(this.listQuery);
    },
    //获取用户列表
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
    //获取选中获取用户
    handleSelectionChange(val) {
        this.multipleSelection = val; //获取多选用户
        // console.log(val);    
    },
    //更改多个用户黑名单设置
    changeAllAccess(command){
      if(!this.multipleSelection.length){
         this.$message.error('请先选中用户');
         return false
      }
      if(command==='b'){
        this.$confirm("是否要禁止选中用户访问权限, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        changeAccessStatus(1).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$notify({
              title: "提示",
              message: "禁止选中用户访问权限成功",
              type: "success",
              duration: 2000,
            });
            row.no_access=!row.no_access
          } else {
            this.$notify({
              title: "提示",
              message: "禁止选中用户访问权限失败,请重试",
              type: "error",
              duration: 2000,
            });
          }
        });
      });
      }else{
        this.$confirm("是否要禁止选中用户评论权限, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        changeCommentStatus(1).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$notify({
              title: "提示",
              message: "禁止选中用户评论权限成功",
              type: "success",
              duration: 2000,
            });
            row.no_comment=!row.no_comment
          } else {
            this.$notify({
              title: "提示",
              message: "禁止选中用户评论权限失败,请重试",
              type: "error",
              duration: 2000,
            });
          }
        });
      });
      }
      this.multipleSelection=[] //清除
    },
    //更改单个用户黑名单设置
    changeAccess(row,command){
      if(command==='b'){
        this.$confirm("此操作将修改用户访问权限, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        changeAccessStatus(!row.no_access).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$notify({
              title: "提示",
              message: "修改用户访问权限成功",
              type: "success",
              duration: 2000,
            });
            row.no_access=!row.no_access
          } else {
            this.$notify({
              title: "提示",
              message: "修改用户访问权限失败,请重试",
              type: "error",
              duration: 2000,
            });
          }
        });
      });
      }else{
        this.$confirm("此操作将修改用户评论权限, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        changeCommentStatus(!row.no_comment).then((response) => {
          // console.log(response)
          if ((response.code = 20000)) {
            this.$notify({
              title: "提示",
              message: "修改用户评论权限成功",
              type: "success",
              duration: 2000,
            });
            row.no_comment=!row.no_comment
          } else {
            this.$notify({
              title: "提示",
              message: "修改用户评论权限失败,请重试",
              type: "error",
              duration: 2000,
            });
          }
        });
      });
      }
      
    },
    //点击用户详情按钮
    handleUserDetail(row) {
      this.dialogFormVisible = true;
      fetchUserDetail({id:row.user.id}).then(response=>{
        this.temp = response.data.user; 
      })
      fetchUserCourse({id:row.user.id}).then(response=>{
        this.tempCourse=response.data.items
      })
      fetchUserColumn({id:row.user.id}).then(response=>{
        this.tempColumn=response.data.items
      })
      fetchUserOrder({id:row.user.id}).then(response=>{
        this.tempOrder=response.data.items
      })
      fetchUserHistory({id:row.user.id}).then(response=>{
        this.tempHistory=response.data.items
      })
      fetchUserComment({id:row.user.id}).then(response=>{
        this.tempComment=response.data.items
      })
    },
    //页码切换
    getDetailList(){
      fetchUserCourse(this.tempCourseQuery).then(response=>{
        this.tempCourse=response.data.items
      })
      fetchUserColumn(this.tempCourseQuery).then(response=>{
        this.tempColumn=response.data.items
      })
      fetchUserOrder(this.tempCourseQuery).then(response=>{
        this.tempOrder=response.data.items
      })
      fetchUserHistory(this.tempCourseQuery).then(response=>{
        this.tempHistory=response.data.items
      })
      fetchUserComment(this.tempCourseQuery).then(response=>{
        this.tempComment=response.data.items
      })
    },
    handleTabClick(tab, event) {
      this.tempCourseQuery.page=1
      this.tempCourseQuery.limit=20
      this.getDetailList()
    }

  },
};
</script>
<style scoped>
table {
  margin-left: 10px;
}
</style>