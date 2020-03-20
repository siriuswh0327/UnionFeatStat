<template>
  <div class="wrapper">
    <h1 class="title">呆萌活跃度周统计表</h1>
    <section class="upload-wrapper">
      <el-upload
        class="upload-container"
        drag
        accept="application/json"
        action="https://jsonplaceholder.typicode.com/posts/"
        :on-change="dealData"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将json文件拖到此处，或<em>点击上传</em></div>
      </el-upload>
    </section>
    <main class="stat-wrapper">
      <div class="stat-table-wrapper"></div>
      <div class="stat-chart-wrapper"></div>
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
    const BASE_LINE_FRI = 300;
    const ACTIVE_LINE_FRI = 535;
    const BASE_LINE_WEEK = 450;
    const ACTIVE_LINE_WEEK = 800;
    const BASE_FINISHED_TASK_FRI = 90;
    const BASE_FINISHED_TASK_WEEK = 150;
    let unWeekend = today.getDay() <= 5;
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
          console.log(result.data.members);
          let unionMemberData = result.data.members.map((item) => {
            //获取基本参数姓名、加入时间、周任务次数、周功勋
            let {name, join_time, task_finished_week, weekly_feats} = item;
            //判断是否新成员
            let isNew = (that.dateTimeToday - join_time * 1000) < (7 * 24 * 3600 * 1000);
            //根据任务次数计算合格线
            let memberQualifiedLine = that.calQualifiedLine(task_finished_week);
            //给寮友打标记
            let tag = that.tagMembers(task_finished_week, weekly_feats, memberQualifiedLine);

            return {
              name: name,
              isNew: isNew,
              taskFinishedWeek: task_finished_week,
              weeklyFeats: weekly_feats,
              memberQualifiedLine: memberQualifiedLine,
              tag: tag
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
      console.log(taskFinishedWeek);
      let featDiff = taskFinishedWeek - this.baseFinishedTask;
      let qualifiedLineCorrection = featDiff < 0 ? Math.floor(featDiff / 30) * 40 : 0;
      console.log(qualifiedLineCorrection);
      return (this.baseQualifiedLine -  qualifiedLineCorrection);
    },
    tagMembers(taskFinishedWeek, weeklyFeats, qualifiedLine) {
      console.log("给寮友打标记");
      console.log(taskFinishedWeek);
      console.log(weeklyFeats);
      console.log(qualifiedLine);
      /*
      * 寮友标记说明：
      * danger: 高危待清理区（周任务次数＜基准任务数baseFinishedTask && 周功勋<基准合格线baseQualifiedLine）
      * warning: 黄色警告区 （周任务次数<基准任务数baseFinishedTask && 周功勋>=基准合格线baseQualifiedLine） 
      * active: 活跃寮友 （周任务次数=== 基准任务数baseFinishedTask+60 && 周功勋>=基准优秀线activeWualifiedLine）
      * normal: 合格寮友（包括新进的）
      */
      let memberTag = "";
      if(taskFinishedWeek < this.baseFinishedTask && weeklyFeats < qualifiedLine){
        memberTag = "danger";
      }else if(taskFinishedWeek < this.baseFinishedTask && weeklyFeats >= qualifiedLine){
        memberTag = "warning"
      }else if(taskFinishedWeek === (this.baseFinishedTask + 60) &&  weeklyFeats >= this.activeWualifiedLine){
        memberTag = "active"
      }else{
        memberTag = "normal"
      }
      console.log(memberTag)

      return memberTag;
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

    .upload-wrapper {
      width: 100%;
      box-sizing: border-box;
      padding: 20px;

      .upload-container {
        width: 400px;
        margin: 0 auto;
      }
    }

    .stat-wrapper {
      width: 100%;
      box-sizing: border-box;
      padding: 20px;
      display: flex;

      .stat-table-wrapper {
        flex: 1;
      }

      .stat-chart-wrapper {
        width: 200px;
      }
    }
  }

</style>