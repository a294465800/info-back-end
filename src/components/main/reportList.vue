<style>
.report-wrap {
  height: 100%;
  padding: 15px;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.breadcrumb {
  margin-bottom: 20px;
}

.report-operation {
  margin-bottom: 20px;
  text-align: right;
}

.report-picker > div {
  margin-right: 20px;
}

.report-wrap .table-list {
  flex: 1;
  overflow: auto;
}

.pages {
  text-align: right;
}

.report-wrap .el-input-group__prepend {
  width: 200px;
}
</style>


<template>
  <section v-if="loading" class="loading">
    <i class="el-icon-loading"></i>
  </section>
  <section v-else class="report-wrap">
    <el-breadcrumb separator="/" class="breadcrumb">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>举报与申诉</el-breadcrumb-item>
    </el-breadcrumb>

    <div class="report-operation">
      <div style="text-align: right;">
        <el-input placeholder="请输入搜索内容" v-model="selectInput" style="margin-bottom: 20px; width: 900px;" @keyup.enter.native="selectSearch">
          <el-select v-model="select" slot="prepend" placeholder="请选择">
            <el-option label="用户名" value="username"></el-option>
            <el-option label="用户编号" value="user_id"></el-option>
          </el-select>
          <el-button slot="append" icon="search" @click="selectSearch"></el-button>
        </el-input>
      </div>
      <div class="report-picker">
        <!-- <el-select v-model="searchFrom.city" multiple filterable placeholder="请选择城市">
          <el-option v-for="item in cities" :key="item.id" :label="item.name" :value="item.id">
          </el-option>
        </el-select>
        <el-select v-model="searchFrom.category" multiple filterable placeholder="请选择种类">
          <el-option v-for="item in categories" :key="item.id" :label="item.name" :value="item.id">
          </el-option>
        </el-select>
        <el-date-picker v-model="searchFrom.dateRange" type="daterange" align="right" placeholder="选择日期范围" :picker-options="dateOptions">
        </el-date-picker> -->
        <el-select v-model="searchFrom.state" filterable placeholder="请选择申诉情况" @change="pickerSearch">
          <el-option v-for="item in reportTypes" :key="item.id" :label="item.name" :value="item.id">
          </el-option>
        </el-select>
        <!-- <el-button type="primary" icon="search" @click="pickerSearch">搜索</el-button> -->
      </div>
    </div>

    <div class="table-list">
      <el-table ref="multipleTable" :data="reportList" border stripe tooltip-effect="dark" style="width: 100%">
        <el-table-column label="信息编号" prop="commodity_id">
        </el-table-column>
        <el-table-column label="信息名" prop="commodity_name" show-overflow-tooltip>
          <template slot-scope="scope">
            <el-button type="text" @click="goToInfo(scope.row)">{{scope.row.commodity?scope.row.commodity.title: ''}}</el-button>
          </template>
        </el-table-column>
        <el-table-column label="被举报次数">
          <template slot-scope="scope">{{scope.row.report_number + ' 次'}}</template>
        </el-table-column>
        <el-table-column label="浏览量">
          <template slot-scope="scope">{{scope.row.read_number + ' 次'}}</template>
        </el-table-column>
        <el-table-column label="举报时间" prop="created_at"></el-table-column>
        <!-- <el-table-column label="已申诉">
          <template slot-scope="scope">
            <span v-if="scope.row.replay === 0">暂无</span>
            <el-button v-else type="text" @click="replayReport(scope.$index, scope.row)">回复申诉</el-button>
          </template>
        </el-table-column> -->
        <el-table-column prop="username" label="举报者">
          <template slot-scope="scope">
            <el-button type="text" @click="goToUser(scope.row.user_id)">{{scope.row.username}}</el-button>
          </template>
        </el-table-column>
        <el-table-column label="处理情况">
          <template slot-scope="scope">
            <span class="normal" v-if="scope.row.state == 0">未处理</span>
            <span class="info" v-else-if="scope.row.state == 1">已处理</span>
            <span class="warning" v-else>已延期</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <!-- <el-button size="small" type="primary" @click="handleReplay(scope.$index, scope.row)">回复</el-button> -->
            <el-button v-if="scope.row.state != 1" size="small" type="primary" @click="handle(scope.$index, scope.row)">处理</el-button>
            <el-button v-if="scope.row.state == 0" size="small" type="warning" @click="handleDelay(scope.$index, scope.row)">延期</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="pages">
      <el-pagination  @current-change="handleCurrentChange" :current-page.sync="page" :page-size="eachPage" layout="total, prev, pager, next" :total="count">
      </el-pagination>
    </div>

  </section>
</template>


<script>
export default {
  data() {
    return {
      loading: true,
      //搜索
      selectInput: "",
      select: "username",

      searchFrom: {
        state: ""
      },

      //这里是手动加一，以防误判
      reportTypes: [
        {
          id: 1,
          name: "未处理"
        },
        {
          id: 2,
          name: "已处理"
        },
        {
          id: 3,
          name: "已延期"
        }
      ],

      //页码相关
      page: 1,
      eachPage: 10,
      count: 0,
      //数据
      reportList: []
    };
  },

  created() {
    this.$api.getReports("", res => {
      this.reportList = res.data.data;
      this.loading = false;
      this.count = res.data.count;
    });
  },

  methods: {
    //用户跳转
    goToUser(id) {
      this.$router.push({ name: "usersingle", params: { id: id } });
    },

    //查看单条信息
    goToInfo(row) {
      let temp = Object.assign({}, row.commodity, row);
      this.$router.push({
        name: "auditsingle",
        params: { info: temp, state: 3 }
      });
    },
    /*
    * 第一个搜索
    */
    selectSearch() {
      if (this.selectInput) {
        const search = {
          [this.select]: this.selectInput
        };
        this.$api.getReports(search, res => {
          this.reportList = res.data.data;
          this.count = res.data.count;
        });
      } else {
        this.$api.getReports("", res => {
          this.reportList = res.data.data;
          this.count = res.data.count;
        });
      }
    },

    /*
    * 第二个搜索
    */
    pickerSearch() {
      this.$api.getReports(this.searchFrom, res => {
        this.reportList = res.data.data;
        this.count = res.data.count;
      });
    },

    /*
    * 回复申诉
    */
    replayReport(index, row) {
      // console.log()
    },

    /*
    * 回复举报者
    */
    handleReplay(index, row) {},

    /*
    * 处理举报信息
    */
    handle(index, row) {
      this.$operation.tableMessageBox("此操作将处理该举报信息", () => {
        this.$api.changeReport(row.id, { state: 1 }, res => {
          this.reportList[index].state = 1;
          this.$message({
            type: "success",
            message: "已处理"
          });
        });
      });
    },

    /*
    * 延期处理
    */
    handleDelay(index, row) {
      this.$operation.tableMessageBox("此操作将延期该举报信息", () => {
        this.$api.changeReport(row.id, { state: 2 }, res => {
          this.reportList[index].state = 2;
          this.$message({
            type: "success",
            message: "已延期"
          });
        });
      });
    },

    /*
    * 页数改变
    */
    handleCurrentChange(page) {
      let getData = {
        page: page,
        state: this.searchFrom.state,
        [this.select]: this.selectInput
      };
      this.$api.getReports(getData, res => {
        this.reportList = res.data.data;
      });
    }
  }
};
</script>
