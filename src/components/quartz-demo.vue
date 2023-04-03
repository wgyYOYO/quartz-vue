<template>
  <div>
    <div style="padding-left: 16px;float: left">NONE无, NORMAL正常, PAUSED暂停, COMPLETE完全, ERROR错误, BLOCKED阻塞
      <el-button type="primary" icon="el-icon-plus" circle @click="openEdit()"></el-button>
    </div>

    <el-table
        v-show="isTable"
        :data="tableData"
        style="width: 100%;argin: 20px">
      <el-table-column
          fixed
          align="center" header-align="center"
          prop="job.jobName"
          label="jobName"
          width="240">
      </el-table-column>
      <el-table-column
          align="center" header-align="center"
          prop="job.jobGroupName"
          label="jobGroupName"
          width="240">
      </el-table-column>
      <el-table-column
          align="center" header-align="center"
          prop="job.jobCron"
          label="jobCron"
          width="120">
        <template slot-scope="scope">
          {{ 'CRON' == scope.row.job.type ? scope.row.job.jobCron : "" }}
        </template>
      </el-table-column>
      <el-table-column
          align="center" header-align="center"
          prop="job.jobAtTime"
          label="jobAtTime"
          width="120">
        <template slot-scope="scope">
          {{ 'SIMPLE' == scope.row.job.type ? scope.row.job.jobAtTime : "" }}
        </template>
      </el-table-column>
      <el-table-column
          align="center" header-align="center"
          prop="job.jobCount"
          label="jobCount"
          width="120">
        <template slot-scope="scope">
          {{ 'SIMPLE' == scope.row.job.type ? scope.row.job.jobCount : "" }}
        </template>
      </el-table-column>

      <el-table-column
          align="center" header-align="center"
          prop="state"
          label="state"
          width="120">
      </el-table-column>
      <el-table-column
          align="center" header-align="center"
          prop="job.jobData"
          label="jobData"
      >
        <template slot-scope="scope">
          {{ scope.row.job.jobData }}
        </template>
      </el-table-column>
      <el-table-column
          width="240px"
          align="center" header-align="center"
          label="操作">
        <template slot-scope="scope">
          <el-button type="success" icon="el-icon-check" circle
                     v-if="scope.row.state == 'NULL'"
                     @click="open(scope.row)">
          </el-button>
          <el-button type="success" icon="el-icon-switch-button" circle
                     v-if="scope.row.state != 'NULL' && scope.row.state != 'PAUSED'"
                     @click="pause(scope.row)">
          </el-button>
          <el-button type="info" icon="el-icon-switch-button" circle
                     v-if="scope.row.state != 'NULL' && scope.row.state == 'PAUSED'" @click="resume(scope.row)">
          </el-button>
          <el-button type="primary" icon="el-icon-edit" circle
                     @click="openEdit(scope.row)"></el-button>
          <el-button type="danger" icon="el-icon-delete" circle
                     @click="delJob(scope.row)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-form v-if="!isTable" :model="formData" label-suffix=":" label-width="160px" size="small">
      <el-form-item>
      </el-form-item>
      <el-form-item prop="type" label="type">
        <el-select v-model="formData.type" clearable
                   :disabled="formData.id"
                   :style="{width: '100%'}">
          <el-option v-for="(item, index) in dict" :key="index" :label="item.label"
                     :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="jobClazz" label="jobClazz">
        <el-input v-model="formData.jobClazz" :disabled="formData.id"></el-input>
      </el-form-item>
      <el-form-item prop="jobGroupName" label="jobGroupName">
        <el-input v-model="formData.jobGroupName" :disabled="formData.id"></el-input>
      </el-form-item>
      <el-form-item prop="jobName" label="jobName">
        <el-input v-model="formData.jobName" :disabled="formData.id"></el-input>
      </el-form-item>
      <el-form-item prop="jobAtTime" label="jobAtTime" v-if="'SIMPLE'== formData.type">
        <el-input v-model="formData.jobAtTime" :disabled="formData.id"></el-input>
      </el-form-item>
      <el-form-item prop="jobCount" label="jobCount" v-if="'SIMPLE'== formData.type">
        <el-input v-model="formData.jobCount" :disabled="formData.id"></el-input>
      </el-form-item>
      <el-form-item prop="jobCron" label="jobCron" v-if="'CRON'== formData.type">
        <el-row>
          <el-col :span="22">
            <el-input v-model="formData.jobCron"></el-input>
          </el-col>
          <el-col :span="2">
            <el-button type="primary" @click="showDialog">生成 cron</el-button>

          </el-col>
        </el-row>






      </el-form-item>
      <el-form-item prop="jobData" label="jobData">
        <el-input v-model="formData.jobData"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="save(formData)">保存</el-button>
        <el-button @click="isTable = true">取消</el-button>
      </el-form-item>
    </el-form>


    <el-dialog title="生成 cron" :visible.sync="showCron">
      <vcrontab @hide="showCron=false" @fill="crontabFill" :expression="expression" :hideComponent="['year']"></vcrontab>
    </el-dialog>
  </div>
</template>

<script>
import request from "../util/request";

export default {
  name: "quartz-demo",
  data() {
    return {
      isTable: true,
      tableData: [],
      formData: {},
      dict: [
        {label: "CRON", value: "CRON"},
        {label: "SIMPLE", value: "SIMPLE"},
      ],
      cronPopover: false,
      expression: "",
      showCron: false
    }
  },
  mounted() {
    this.getList();

  },
  methods: {
    crontabFill(value) {
      //确定后回传的值
      // this.input = value;
      debugger
        // let split = value.split(" ");
        // split.pop()
        // let join = split.join(" ");
        this.formData.jobCron = value

    },
    showDialog() {
      this.expression = this.input;//传入的 cron 表达式，可以反解析到 UI 上
      this.showCron = true;
    },
    // changeCron(val) {
    //   let split = val.split(" ");
    //   split.pop()
    //   let join = split.join(" ");
    //   this.formData.jobCron = join
    //
    // },
    getList() {
      let self = this;
      request({
        url: "/quartz",
        method: 'get',
      }).then(res => {
        self.tableData = res;
      })
    },
    delJob(row) {
      let self = this;
      request({
        url: '/quartz/' + row.job.id,
        method: 'DELETE',

      }).then(data=>{
        self.getList();
        self.isTable = true;
      })
    },
    pause(row) {
      let self = this;
      request({
        url: '/quartz/pause/' + row.job.id,
        method: 'post',
        contentType: "application/json;charset=UTF-8",
        data: JSON.stringify(row),

      }).then(data=>{
        self.getList();
        self.isTable = true;

      })
    },
    open(row) {
      let self = this;
      request({
        url: '/quartz/open/' + row.job.id,
        method: 'post',
        contentType: "application/json;charset=UTF-8",
        data: JSON.stringify(row),

      }).then(data=>{
        self.getList();
        self.isTable = true;
      })
    },
    resume(row) {
      let self = this;
      request({
        url: '/quartz/resume/' + row.job.id,
        method: 'post',
        contentType: "application/json;charset=UTF-8",
        data: JSON.stringify(row),
      }).then(data=>{
        self.getList();
        self.isTable = true;
      })
    },
    save(row) {
      let self = this;
      request({
        url: "/quartz/save",
        method: 'post',
        headers: {
          'Context-Type': 'application/json'
        },
        data: row,
      }).then(data=>{
        self.getList();
        self.isTable = true;
      })
    },
    openEdit(row = {
      job: {
        "type": "CRON",
        "jobClazz": "antu.com.land.service.quartz.domian.bean.RpcQuartzJobBean",
        "jobGroupName": "",
        "jobName": "",
        "jobAtTime": 10000,
        "jobCount": 1,
        "jobCron": "",
        "jobData": "{\"url\": \"\", \"method\": \"\", \"data\": \"\"}"
      }
    }) {
      console.log(row);
      let self = this;
      self.formData = Object.assign({}, row.job, {});
      self.isTable = false;
    }
  }
}
</script>

<style scoped>

</style>
