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
          v-model="form.dateRange"
          type="datetimerange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        >
        </el-date-picker>
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
  name: "confirm",
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
      title: "日程事件",
      constForm: {}, //对比form是否改变
      form: {
        scheduleId: "",
        title: "",
        dateRange: [],
        startDate: "",
        endDate: "",
        address: "",
        comment: ""
      }
    };
  },
  computed: {
  },
  methods: {
    deepCheck(obj1, obj2) {
      // 深度遍历，对比值是否相等
      for (let key in obj1) {
        if (obj1[key] instanceof Object && !(obj1[key] instanceof Date)) {
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
      this.form.startDate = moment(this.form.dateRange[0]).format("YYYY-MM-DD HH:mm:ss");
      this.form.endDate =  moment(this.form.dateRange[1]).format("YYYY-MM-DD HH:mm:ss");
      this.form.ownerDate = moment(form.dateRange[0]).format("YYYY-MM-DD");
      this.$emit("confirm", this.form);
    },
    close() {
      this.$emit("close");
    },
    handleDialogClose() {
      this.close();
    },
    getForm() {
      return this.form;
    },
    setForm(data) {
      this.form = {
        scheduleId: undefined,
        title: "",
        dateRange: [],
        type:'',
        startDate: "",
        endDate: "",
        address: "",
        comment: ""
      };
      for (let key in this.form) {
        if (data[key] && data[key] != "") {
          this.form[key] = data[key];
        }
      }
      if(data.type == "add"){
        this.title = "添加日程事件"
      }
      else{
         this.title = "修改日程事件"
      }
      this.constForm = Object.assign({}, this.form);
      // console.log(this.form)
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
