<template>
  <div class="wrapper">
    <h1 class="title">呆萌活跃度周统计表</h1>
    <main class="stat-wrapper">
      <div class="stat-table-wrapper">
        <el-table
          :data="tableData"
          style="width: 100%"
          :row-class-name="tableRowClassName"
          :default-sort = "{prop: 'weeklyFeats', order: 'descending'}">
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

        <div class="btn-export-wrapper">
          <el-button type="primary" @click="showLotterryCondition">导出抽奖名单</el-button>

          <el-dialog
            title="抽奖标准设置"
            :visible.sync="lotterryConditionDialogVisible"
            class="lottery-setting-dialog"
            width="320px">
            <el-row :gutter="20"> 
              <el-col :span="10" :offset="2">
                <span class="lottery-condition-label">周功勋：</span>
              </el-col>
              <el-col :span="10">
                <el-input 
                  v-model="lotteryWeeklyFeat" 
                  size="small">
                </el-input>
              </el-col>
            </el-row>
            <el-row :gutter="20"> 
              <el-col :span="10" :offset="2">
                <span class="lottery-condition-label">周任务次数：</span>
              </el-col>
              <el-col :span="10">
                <el-input 
                  v-model="lotteryWeeklyFinishedTask" 
                  size="small">
                </el-input>
              </el-col>
            </el-row>
            <el-row :gutter="20"> 
              <el-col :span="10" :offset="2">
                <span class="lottery-condition-label">累积功勋：</span>
              </el-col>
              <el-col :span="10">
                <el-input 
                  v-model="lotteryTotalFeat" 
                  size="small">
                </el-input>
              </el-col>
            </el-row>
            <el-row type="flex" justify="center" style="margin-top:15px;">
              <el-button 
                type="primary"
                @click="exportMembers">
                导出名单
              </el-button>
            </el-row>
          </el-dialog>

        </div>
      </div>
    </main>
  </div>
</template>

<script>
// @ is an alias to /src
import FileSaver from 'file-saver'

export default {
  name: 'Home',
  data() {
    return {
      dateTimeToday: 0, //今日时间戳
      baseQualifiedLine: 450,//基准合格线
      activeQualifiedLine: 800,//基准优秀线
      baseFinishedTask: 150,//周任务次数合格线
      tableData: [],
      lotterryConditionDialogVisible: false, //抽奖界面条件筛选
      lotteryWeeklyFeat: 800, //抽奖周功勋标准
      lotteryWeeklyFinishedTask: 210, //抽奖周任务次数标准
      lotteryTotalFeat: 0 //抽奖累计功勋标准
    }
  },
  created() {
    const today = new Date();
    const WeekDay = today.getDay();//星期几
    console.log(WeekDay);
    const BASE_LINE_SINGLE = 60 * WeekDay; // 非周日功勋基准线
    const ACTIVE_LINE_SINGLE = 100 * WeekDay; // 非周日功勋活跃线
    const BASE_LINE_WEEK = 450; //周日功勋基准线
    const ACTIVE_LINE_WEEK = 800; //周日功勋活跃线
    const BASE_FINISHED_TASK_SINGLE = 30 * WeekDay; //周五任务次数基准线
    const BASE_FINISHED_TASK_WEEK = 210; //周日任务次数基准线
    let unWeekend = WeekDay != 0;//判断本日是不是周日
    this.dateTimeToday = today.getTime();
    this.baseQualifiedLine = unWeekend ? BASE_LINE_SINGLE : BASE_LINE_WEEK;
    this.activeQualifiedLine = unWeekend ? ACTIVE_LINE_SINGLE : ACTIVE_LINE_WEEK;
    this.baseFinishedTask = unWeekend ? BASE_FINISHED_TASK_SINGLE : BASE_FINISHED_TASK_WEEK;

    console.log(this.baseQualifiedLine);
    console.log(this.activeQualifiedLine);
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
      * active: 活跃寮友 （周任务次数=== 基准任务数baseFinishedTask+60 && 周功勋>=基准优秀线activeQualifiedLine）
      * normal: 合格寮友
      * new： 新进
      */
      let memberTag = "";
      if(weeklyFeats < qualifiedLine){
        memberTag = isNew ? "new" : "danger";
      }else if(taskFinishedWeek < this.baseFinishedTask && weeklyFeats >= qualifiedLine){
        memberTag = isNew ? "new" : "warning";
      }else if(taskFinishedWeek >= this.baseFinishedTask &&  weeklyFeats >= this.activeQualifiedLine){
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
    },
    //导出json文件
    exportJSON(data) {
      try {
        const dataStr = JSON.stringify(data);
        const blob = new Blob([dataStr],{type:''});
        const todayStr = this.createDateStr();
        const fileName = "LotteryMembersOfDM" + todayStr + ".json";
        FileSaver.saveAs(blob,fileName);
      } catch (err) {
        console.error(err);
      }
    },
    //显示抽奖门槛设置
    showLotterryCondition() {
      if(this.tableData.length > 0){
        this.lotterryConditionDialogVisible = true;
      }else{
        this.$alert('统计表为空，请先上传json数据', '提示', {
          confirmButtonText: '确定',
        });
      }
    },
    //导出名单
    exportMembers() {
      console.log("导出名单")
      const LotteryWeeklyFeat = this.lotteryWeeklyFeat;
      const LotteryWeeklyFinishedTask = this.lotteryWeeklyFinishedTask;
      const LotteryTotalFeat = this.lotteryTotalFeat;
      
      if(this.tableData.length > 0) {
        let exportTableData = this.tableData.filter(
            member => (member.weeklyFeats >= LotteryWeeklyFeat 
                    && member.taskFinishedWeek >=LotteryWeeklyFinishedTask
                    && member.totalFeats >= LotteryTotalFeat)
        ).map(member => {
          return {
            name: member.name,
            editable: false
          }
        })
        console.log(exportTableData);
        this.$nextTick(() => {
          this.exportJSON(exportTableData);
        })

      }
    },
    createDateStr() {
      const date = new Date();
      let year = date.getFullYear().toString();
      let month = (date.getMonth() + 1).toString().padStart(2,'0');
      let day = date.getDay().toString().padStart(2,'0');
      let hour = date.getHours().toString().padStart(2,'0');
      let minute = date.getMinutes().toString().padStart(2,'0');

      return (year + month + day + hour + minute);
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
          color: #777;

          .danger-member {
            background: #FF8a80;
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
        .btn-export-wrapper {
          margin-top: 20px;
          display: flex;
          justify-content: flex-end;


          
        }
      }
    }
  }

  .el-row {
    margin-bottom: 10px;

    .lottery-condition-label { 
      display: block;
      width: 100%;
      height: 32px;
      line-height: 32px;
      text-align: right;
    }
  }

  .lottery-setting-dialog {
    .el-dialog__body {
      padding: 20px 20px 5px;
    }
  }
</style>