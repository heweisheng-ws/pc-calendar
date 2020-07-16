<template>
  <div class="calendar">
    <FullCalendar
      ref="calendar"
      class="fullCalendar"
      defaultView="dayGridMonth"
      locale="zh-cn"
      :slot-event-overlap="false"
      :holiaday="holiday"
      firstDay="1"
      weekNumberCalculation="ISO"
      :eventTimeFormat="eventTime"
      :header="header"
      :customButtons="customButtons"
      @dateClick="handleDateClick"
      :plugins="calendarPlugins"
      :events="calendarEvents"
      @eventClick="handleEventClick"
      :button-text="{
        today: '今天'
      }"
    >
    </FullCalendar>
    <calendarDialog
      ref="dialog"
      :visible.sync="confirm.visible"
      @close="close"
      @confirm="checkConfirm(arguments)"
      @del="delEvent(arguments)"
    />
  </div>
</template>

<script>
// @ is an alias to /src
import FullCalendar from "@fullcalendar/vue";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import moment from "moment";

import calendarDialog from "./calendar-dialog";

export default {
  name: "calendar",
  components: {
    FullCalendar,
    calendarDialog
  },
  data() {
    return {
      holiday: {
        "2020-06-01": "假",
        "2020-06-02": "假",
        "2020-06-25": "班"
      },
      confirm: {
        visible: false
      },
      customButtons: {
        myNext: {
          text: "下个月",
          click: () => {
            this.myButton("next");
          }
        },
        myPrev: {
          text: "上个月",
          click: () => {
            this.myButton("prev");
          }
        },
        myNextYear: {
          text: "下一年",
          click: () => {
            this.myButton("nextYear");
          }
        },
        myPrevYear: {
          text: "上一年",
          click: () => {
            this.myButton("prevYear");
          }
        },
        myToday: {
          text: "今天",
          click: () => {
            this.myToday();
          }
        }
      },
      calendarPlugins: [dayGridPlugin, interactionPlugin],
      header: {
        left: "myPrevYear ,myPrev ,title , today , myNext, myNextYear",
        center: "none",
        right: "none"
      },
      eventTime: {
        hour: "numeric",
        minute: "2-digit",
        hour12: false
      },
      calendarEvents: [
        {
          scheduleId: "1",
          title: "部门会议1",
          start: "2020-07-01 00:00:00",
          end: "2020-07-03 00:00:00",
          ownerDate: "2020-06-01",
          showMore: false
        },
        {
          scheduleId: "2",
          title: "部门会议1",
          start: "2020-07-01 00:00:00",
          end: "2020-07-03 00:00:00",
          ownerDate: "2020-06-01",
          showMore: false
        },
        {
          scheduleId: "3",
          title: "部门会议2",
          start: "2020-07-1 00:00:00",
          end: "2020-07-03 00:00:00",
          ownerDate: "2020-06-01",
          showMore: false
        },
      ]
    };
  },
  methods: {
    myToday() {
      this.$refs.calendar.$options.calendar.today();
    },
    myButton(type) {
      if (type === "next") {
        this.$refs.calendar.$options.calendar.next();
      } else if (type === "prev") {
        this.$refs.calendar.$options.calendar.prev();
      } else if (type === "nextYear") {
        this.$refs.calendar.$options.calendar.nextYear();
      } else if (type === "prevYear") {
        this.$refs.calendar.$options.calendar.prevYear();
      }
    },
    checkConfirm(arg) {
      this.confirm.visible = false;
      console.log(arg);
      let data = arg[0];
      if (!data.startDate) {
        return;
      }
      let params = {
        address: "",
        comment: "",
        endDate: "",
        scheduleId: "",
        ownerDate: "",
        remind: "",
        replayStatus: "",
        startDate: "",
        title: "",
        type: ""
      };
      params = Object.assign(params, data);
      delete params.radio;
      params.start = moment(params.startDate).format("YYYY-MM-DD HH:mm:ss");
      params.end = moment(params.endDate).format("YYYY-MM-DD HH:mm:ss");
      let eventIndex = this.calendarEvents.findIndex(
          item => item.scheduleId == params.scheduleId
          );
          console.log(eventIndex)
      if (eventIndex != -1) {
          this.calendarEvents[eventIndex] = params
      } else {
        console.log(params)
        this.calendarEvents.push(params);
      }
    },
    close() {
      this.confirm.visible = false;
    },
    delEvent(arg) {
      console.log(arg);
      if (!arg[0].scheduleId) {
        this.close();
        return;
      }
      let eventIndex = this.calendarEvents.findIndex(
         item => item.scheduleId == arg[0].scheduleId
       );
     if (eventIndex > -1) {
         this.calendarEvents.splice(eventIndex, 1);
       }
    
      this.close();
    },
    handleDateClick(arg) {
      console.log(arg)
      if (window.moredafult) {
        window.moredafult = false;
        return;
      }
      let classList = arg.dayEl.classList;
      if (classList.contains("fc-last-month")) {
        let dom = document.getElementsByClassName("fc-myPrev-button");
        dom[0].click();
        return;
      } else if (classList.contains("fc-next-month")) {
        let dom = document.getElementsByClassName("fc-myNext-button");
        dom[0].click();
        return;
      }
      let data = {
        scheduleId: arg.view.uid,
        title: "",
        dateRange: []
      };
      this.$refs.dialog.setForm(data);
      this.confirm.visible = true;
    },
    handleEventClick(info) {
      let data = Object.assign({}, info.event.extendedProps);
      data.title = info.event.title;
      data.dateRange = [info.event.start, info.event.end];
      console.log(data)
      this.$refs.dialog.setForm(data);
      this.confirm.visible = true;
    },
  
  },
  mounted() {
    var that = this;
    window.addEventListener("clickMore", function(event) {
      console.log(event.detail.eventRange);
      that.calendarEvents.map(item => {
        if (
          moment(item.viewStartDate).format("YYYY-MM-DD") ==
          moment(
            event.detail.eventRange.def.extendedProps.viewStartDate
          ).format("YYYY-MM-DD")
        ) {
          item.showMore = true;
        }
      });
      console.log(that.calendarEvents);
      that.calendarEvents = [...that.calendarEvents];
    });
  }
};
</script>

<style scoped lang="scss">
@import "~@fullcalendar/core/main.css";
@import "~@fullcalendar/daygrid/main.css";

.calendar {
  height: 100vh;
  padding: 80px 200px;
  // padding-bottom: 30px;
  // width: 40vw;
  // height: 500px;
}
.fullCalendar {
  height: 600px;
  // box-sizing: border-box;
}
</style>
