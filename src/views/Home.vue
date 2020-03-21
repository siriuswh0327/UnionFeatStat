<template>
  <div class="wrapper">
    <h1 class="title">呆萌活跃度周统计表</h1>
    <main class="stat-wrapper">
      <div class="stat-table-wrapper">
        <el-table
          :data="tableData"
          style="width: 100%"
          :row-class-name="tableRowClassName"
          :default-sort = "{prop: 'weeklyFeats', order: 'descending'}"
        >
          <el-table-column
            prop="name"
            label="游戏名"
          >
          </el-table-column>
          <el-table-column
            label="是否新成员"
            width="100"
          >
            <template slot-scope="scope">
              <span>{{ scope.row.isNew ? '是' : '否' }}</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="taskFinishedWeek"
            label="周任务次数"
            width="120"
            sortable
          >
          </el-table-column>
          <el-table-column
            prop="weeklyFeats"
            label="周功勋"
            width="90"
            sortable
          >
          </el-table-column>
          <el-table-column
            prop="memberQualifiedLine"
            label="合格线"
            width="70"
          >
          </el-table-column>
        </el-table>

      </div>
      <div class="stat-chart-wrapper">
        <el-upload
          class="upload-container"
          drag
          accept="application/json"
          action="https://jsonplaceholder.typicode.com/posts/"
          :on-change="dealData"
          :show-file-list="false"
        >
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将json文件拖到此处，或<em>点击上传</em></div>
        </el-upload>
      </div>
    </main>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: 'Home',
  data() {
    return {
      dateTimeToday: 0,
      baseQualifiedLine: 450,//基准合格线
      activeWualifiedLine: 800,//基准优秀线
      baseFinishedTask: 150,//周任务次数合格线
      tableData: []
    }
  },
  created() {
    const today = new Date();
    const BASE_LINE_FRI = 300; // 周五功勋基准线
    const ACTIVE_LINE_FRI = 535; // 周五功勋活跃线
    const BASE_LINE_WEEK = 450; //周日功勋基准线
    const ACTIVE_LINE_WEEK = 800; //周日功勋活跃线
    const BASE_FINISHED_TASK_FRI = 90; //周五任务次数基准线
    const BASE_FINISHED_TASK_WEEK = 150; //周日任务次数基准线
    let unWeekend = today.getDay() < 7;
    this.dateTimeToday = today.getTime();
    this.baseQualifiedLine = unWeekend ? BASE_LINE_FRI : BASE_LINE_WEEK;
    this.activeWualifiedLine = unWeekend ? ACTIVE_LINE_FRI : ACTIVE_LINE_WEEK;
    this.baseFinishedTask = unWeekend ? BASE_FINISHED_TASK_FRI : BASE_FINISHED_TASK_WEEK;

    console.log(this.baseQualifiedLine);
    console.log(this.activeWualifiedLine);
    console.log(this.baseFinishedTask);
  },
  methods: {
    dealData(file) {
      const that = this;
      let reader = new FileReader();
      reader.readAsText(file.raw);
      reader.onload = function() {
        console.log(this.result);
        try{
          const result = JSON.parse(this.result);
          const members = result.data.members;
          console.log(members);
          let unionMemberData = members.map((item) => {
            //获取基本参数姓名、加入时间、周任务次数、周功勋
            let { name, join_time, task_finished_week, weekly_feats, total_feats } = item;
            //判断是否新成员
            let isNew = (that.dateTimeToday - join_time * 1000) < (7 * 24 * 3600 * 1000);//入寮天数是否小于7天
            //根据任务次数计算合格线
            let memberQualifiedLine = that.calQualifiedLine(task_finished_week);
            //给寮友打标记
            let tag = that.tagMembers(task_finished_week, weekly_feats, memberQualifiedLine, isNew);

            return {
              name: name,// 游戏名
              isNew: isNew,// 是否新成员
              taskFinishedWeek: task_finished_week,// 周任务完成数
              weeklyFeats: weekly_feats, // 周功勋
              totalFeats: total_feats,// 累计功勋
              memberQualifiedLine: memberQualifiedLine,// 合格线
              tag: tag // 标记
            }
          });

          console.log(unionMemberData);

          that.tableData = unionMemberData;
        }catch(err){
          console.error(err);
        }
      }
    },
    calQualifiedLine(taskFinishedWeek) {
      console.log("计算合格线");
      /*
      *功勋合格线计算方法说明：
      *featDiff差值 = 实际任务完成数 - 基准任务完成数
      *如果差值大于等于0， 合格线qualifiedLineCorrection = 0
      *反之，每少30次任务，功勋合格线标准提升40
      */
      let featDiff = taskFinishedWeek - this.baseFinishedTask;
      let qualifiedLineCorrection = featDiff < 0 ? Math.floor(featDiff / 30) * 40 : 0;
      return (this.baseQualifiedLine -  qualifiedLineCorrection);
    },
    tagMembers(taskFinishedWeek, weeklyFeats, qualifiedLine, isNew) {
      console.log("给寮友打标记");
      /*
      * 寮友标记说明：
      * danger: 高危待清理区（周任务次数＜基准任务数baseFinishedTask && 周功勋<基准合格线baseQualifiedLine）
      * warning: 黄色警告区 （周任务次数<基准任务数baseFinishedTask && 周功勋>=基准合格线baseQualifiedLine） 
      * active: 活跃寮友 （周任务次数=== 基准任务数baseFinishedTask+60 && 周功勋>=基准优秀线activeWualifiedLine）
      * normal: 合格寮友
      * new： 新进
      */
      let memberTag = "";
      if(weeklyFeats < qualifiedLine){
        memberTag = isNew ? "new" : "danger";
      }else if(taskFinishedWeek < this.baseFinishedTask && weeklyFeats >= qualifiedLine){
        memberTag = isNew ? "new" : "warning";
      }else if(taskFinishedWeek === (this.baseFinishedTask + 60) &&  weeklyFeats >= this.activeWualifiedLine){
        memberTag = "active";
      }else{
        memberTag = "normal";
      }

      return memberTag;
    },
    tableRowClassName({row}) {
      //根据标签附上对应的行样式
      let rowClass = row.tag === "normal" ? "" : row.tag + "-member";
      return rowClass;
    }
  }
}
</script>

<style lang="scss">
  @import '@/assets/css/global.scss';

  .wrapper {
    width: 100%;
    height: 100%;

    .title {
      width: 100%;
      text-align: center;
      font-size: 36px;
    }


    .stat-wrapper {
      width: 100%;
      height: calc(100vh - 50px);
      box-sizing: border-box;
      padding: 20px;
      display: flex;

      .stat-table-wrapper {
        flex: 1;
        overflow: auto;

        .el-table {
          .danger-member {
            background: #F44336;
          }
          .warning-member {
            background: #FFE57F;
          }
          .active-member {
            background: #1de9b6;
          }
          .new-member {
            background: #B0bEC5;
          }
        }
      }

      .stat-chart-wrapper {
        width: 360px;
        margin-left: 20px;

        .upload-container {
          width: 100%;
          margin: 0 auto;
        }
      }
    }
  }

</style>