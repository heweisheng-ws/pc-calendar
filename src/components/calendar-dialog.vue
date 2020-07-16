/* eslint-disable vue/return-in-computed-property */
<template>
  <el-dialog
    :visible="visible"
    width="800px"
    :title="title"
    :show-close="true"
    :before-close="handleDialogClose"
  >
    <el-form :model="form" ref="form">
      <div class="flex">
        <span class="form-input-title">
          标题:
        </span>
        <el-input
          size="small"
          v-model="form.title"
          autocomplete="off"
          prop="title"
        ></el-input>
      </div>
      <div class="flex">
        <span class="form-input-title">
          生效时间：
        </span>
        <el-date-picker
          size="small"
          v-show="form.allDayStatus == 'YES'"
          v-model="form.dateRange"
          format="yyyy 年 MM 月 dd 日"
          value-format="yyyy-MM-dd"
          type="daterange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
        <el-date-picker
          size="small"
          v-show="form.allDayStatus == 'NO'"
          v-model="form.dateRange"
          type="datetimerange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        >
        </el-date-picker>
        <el-radio-group class="radio-group" v-model="form.allDayStatus">
          <el-radio @click.native.prevent="clickItem('YES')" label="YES">
            全天</el-radio
          >
        </el-radio-group>
      </div>
      <div class="flex">
        <span class="form-input-title">
          日程类型：
        </span>
        <el-radio-group size="small" v-model="form.type">
          <el-radio label="OPEN"> 公开</el-radio>
          <el-radio label="PRIVATE"> 保密</el-radio>
        </el-radio-group>
      </div>
      <div class="flex">
        <span class="form-input-title">
          重复：
        </span>
        <el-select
          size="small"
          v-model="form.replayStatus"
          placeholder="请选择"
          :disabled="showReplay()"
        >
          <el-option label="不重复" value="NONE"> </el-option>
          <el-option
            v-for="item in options1"
            :key="item.value"
            :label="labelOptions1(item.label)"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </div>
      <div class="flex">
        <span class="form-input-title">
          提醒设置：
        </span>
        <el-select size="small" v-model="form.remind" placeholder="请选择">
          <el-option
            v-for="item in options2"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </div>
      <div class="flex">
        <span class="form-input-title">
          地点：
        </span>
        <el-input
          type="textarea"
          v-model="form.address"
          placeholder="请输入地点"
        />
      </div>
      <div class="flex">
        <span class="form-input-title">
          备注：
        </span>
        <el-input
          type="textarea"
          v-model="form.comment"
          placeholder="请输入备注"
        />
      </div>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button size="small" type="primary" @click="confirm">确 定</el-button>
      <el-button size="small" @click="close">取 消</el-button>
      <el-button v-show="delStatus" size="small" type="danger" @click="del()"
        >删除</el-button
      >
    </div>
  </el-dialog>
</template>

<script>
import moment from "moment";
export default {
  name: "comfirm",
  props: {
    visible: {
      default: true
    },
    delStatus: {
      default: true
    }
  },
  data() {
    return {
      title: "添加日期事件",
      options1: [
        // {
        //   value: "NONE",
        //   label: "不重复"
        // },
        {
          value: "DAY",
          label: "每天"
        },
        {
          value: "WEEK",
          label: "每周"
        },
        {
          value: "MONTH",
          label: "每月"
        },
        {
          value: "YEAR",
          label: "每年"
        }
      ],
      options2: [
        {
          value: "NONE",
          label: "不提醒"
        },
        {
          value: "ON_HOUR",
          label: "准点提醒"
        },
        {
          value: "FIVE_MIN",
          label: "提前5分钟"
        },
        {
          value: "TEN_MIN",
          label: "提前10分钟"
        },
        {
          value: "THIRTY_MIN",
          label: "提前30分钟"
        },
        {
          value: "AN_HOUR",
          label: "提前1小时"
        },
        {
          value: "TWO_HOUR",
          label: "提前2小时"
        },
        {
          value: "THREE_HOUR",
          label: "提前3小时"
        },
        {
          value: "A_DAY",
          label: "提前1天"
        },
        {
          value: "TWO_DAY",
          label: "提前2天"
        },
        {
          value: "THREE_DAY",
          label: "提前3天"
        }
      ],
      constForm: {}, //对比form是否改变
      form: {
        scheduleId: "",
        title: "",
        dateRange: [],
        startDate: "",
        endDate: "",
        remind: "NONE",
        allDayStatus: "NO",
        type: "OPEN",
        replayStatus: "NONE",
        address: "",
        comment: ""
      }
    };
  },
  computed: {
    // eslint-disable-next-line vue/return-in-computed-property
    time() {
      if (this.form.allDayStatus == "YES") {
        return "";
      } else {
        if (this.form.dateRange) {
          return moment(this.form.dateRange[0]).format("HH:mm");
        }
      }
    }
  },
  watch: {
    "form.allDayStatus"(val) {
      console.log(val);
      if (val == "YES") {
        this.form.replayStatus = "NONE";
      }
    }
  },
  methods: {
    showReplay() {
      console.log(this.form.dateRange);
      if (this.form.dateRange && this.form.dateRange.length > 0) {
        console.log(22);
        let start = moment(this.form.dateRange[0]).format("YYYY-MM-DD");
        let end = moment(this.form.dateRange[1]).format("YYYY-MM-DD");
        console.log(start, end);
        if (start != end) {
          this.form.replayStatus = "NONE";
          return true;
        } else {
          return false;
        }
      }
    },
    labelOptions1(label) {
      // console.log(moment(this.form.dateRange[0]).format("MMMM Do"));
      if (!this.form.dateRange) {
        return label;
      }
      let dateString = "";
      if (label == "每周") {
        dateString = this.parseWeek(
          moment(this.form.dateRange[0]).format("dddd")
        );
      } else if (label == "每月") {
        dateString = this.parseDate(
          moment(this.form.dateRange[0]).format("Do")
        );
      } else if (label == "每年") {
        dateString =
          this.parseYear(moment(this.form.dateRange[0]).format("MMMM")) +
          this.parseDate(moment(this.form.dateRange[0]).format("Do"));
      }
      return label + " " + dateString + " " + this.time;
    },
    parseWeek(weekDay) {
      let date;
      switch (weekDay) {
        case "Monday":
          date = "一";
          break;
        case "Tuesday":
          date = "二";
          break;
        case "Wednesday":
          date = "三";
          break;
        case "Thursday":
          date = "四";
          break;
        case "Friday":
          date = "五";
          break;
        case "Saturday":
          date = "六";
          break;
        case "Sunday":
          date = "日";
          break;

        default:
          break;
      }
      return date;
    },
    parseDate(th) {
      return th.substring(0, th.length - 2) + "日";
    },
    parseYear(year) {
      let date;
      switch (year) {
        case "January":
          date = "1";
          break;
        case "February":
          date = "2";
          break;
        case "March":
          date = "3";
          break;
        case "ThursApril":
          date = "4";
          break;
        case "May":
          date = "5";
          break;
        case "June":
          date = "6";
          break;
        case "July":
          date = "7";
          break;
        case "August":
          date = "8";
          break;
        case "September":
          date = "9";
          break;
        case "October":
          date = "10";
          break;
        case "November":
          date = "11";
          break;
        case "December":
          date = "12";
          break;

        default:
          break;
      }
      return date + "月";
    },
    deepCheck(obj1, obj2) {
      // 深度遍历，对比值是否相等
      for (let key in obj1) {
        // console.log(key);
        if (obj1[key] instanceof Object && !(obj1[key] instanceof Date)) {
          // for (let key2 in obj1[key]) {
          //   console.log(key2);
          // }
          if (!this.deepCheck(obj1[key], obj2[key])) {
            return false;
          }
        } else {
          if (obj1[key] instanceof Date && obj2[key] instanceof Date) {
            return obj1[key].getTime() == obj1[key].getTime();
          } else if (obj1[key] != obj2[key]) {
            console.log(obj1[key] != obj2[key]);
            return false;
          }
        }
      }
      return true;
    },
    confirm() {
      console.log(this.form);
      let form = this.form;
      if (form.title == "") {
        this.$message({
          showClose: true,
          message: "标题不能为空",
          type: "warning"
        });
        return;
      } else if (!form.dateRange || form.dateRange.length < 1) {
        this.$message({
          showClose: true,
          message: "请选择时间",
          type: "warning"
        });
        return;
      }
      // if (this.deepCheck(this.form, this.constForm)) {
      //   console.log("数据没修改");
      //   this.$emit("close");
      //   return;
      // }
      this.form.startDate = this.form.dateRange[0];
      this.form.endDate = this.form.dateRange[1];
      console.log(this.form.endDate)
      if(this.form.allDayStatus == 'YES'){
        this.form.endDate = moment(this.form.endDate).format("YYYY-MM-DD") + " "+ "23:59:59"
      }
      this.form.ownerDate = moment(form.dateRange[0]).format("YYYY-MM-DD");
      this.$emit("confirm", this.form);
    },
    close() {
      this.$emit("close");
    },
    handleDialogClose() {
      this.close();
    },
    clickItem(e) {
      if (e == "YES") {
        this.form.dateRange = [
          moment(new Date()).format("YYYY-MM-DD" + " 00:00:00"),
          moment(new Date()).format("YYYY-MM-DD" + " 23:59:59")
        ];
        console.log(this.form.dateRange);
      }

      e === this.form.allDayStatus
        ? (this.form.allDayStatus = "NO")
        : (this.form.allDayStatus = "YES");
    },
    getForm() {
      return this.form;
    },
    setForm(data) {
      this.form = {
        scheduleId: undefined,
        title: "",
        dateRange: [],
        startDate: "",
        endDate: "",
        remind: "NONE",
        allDayStatus: "NO",
        type: "OPEN",
        replayStatus: "NONE",
        address: "",
        comment: ""
      };
      if (data.scheduleId) {
        this.title = "修改日期事件";
      } else {
        this.title = "添加日期事件";
      }
      for (let key in this.form) {
        if (data[key] && data[key] != "") {
          this.form[key] = data[key];
        }
      }
      this.constForm = Object.assign({}, this.form);
    },
    del() {
      this.$emit("del", this.form);
    }
  }
};
</script>

<style scoped lang="scss">
.radio-group {
  margin-left: 20px;
}
.radio-group ::v-deep .el-radio__inner {
  box-shadow: initial !important;
}

.flex {
  display: flex;
  flex-direction: row;
  padding: 10px 20px 10px 0px;
  line-height: 30px;
}
.form-input-title {
  display: inline-block;
  width: 100px;
}
.el-input {
  flex: 1;
}
.el-textarea {
  flex: 1;
}
.el-radio-group {
  float: left;
  display: flex !important;
  height: 32px;
  align-items: center;
}
.dialog-footer {
  text-align: left;
  padding-left: 120px;
}
</style>
