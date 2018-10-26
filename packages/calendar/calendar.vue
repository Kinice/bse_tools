<template>
  <div class="calendar">
    <div class="cal-head clearfix">
      <div class="left-con" @click="setMonth(-1)">
        <i class="angle"></i>
      </div>
      <div class="right-con" @click="setMonth(1)">
        <i class="angle"></i>
      </div>
      <div class="info">
        <h3 class="year">{{ year }} 年 {{ month }} 月</h3>
      </div>
    </div>
    <table>
      <thead>
        <tr>
          <th>日</th>
          <th>一</th>
          <th>二</th>
          <th>三</th>
          <th>四</th>
          <th>五</th>
          <th>六</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(line, index) in monthData" :key="index">
          <td
            v-for="(date, i) in line"
            :key="i"
            :class="date.checked ? 'choosen' : ''"
          >
            <span
              @click="chooseDate(date)"
              :class="[date.disable ? 'uncheckable' : '', date.isToday ? 'today' : '', colorList[date.id]]"
            >{{ date.date }}</span>
            <div class="text">{{ date.text }}</div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
const today = new Date();
const thirty = new Date();
thirty.setDate(new Date().getDate() + 30);

export default {
  name: 'calendar',

  data() {
    return {
      monthData: [],
      today: today,
      year: today.getFullYear(),
      month: today.getMonth() + 1,
      date: today.getDate(),
      thisYear: today.getFullYear(),
      thisMonth: today.getMonth() + 1,
      choosenDate: [],
      monthStartDate: '',
      monthEndDate: ''
    };
  },

  props: {
    disableBefore: {
      type: String,
      required: false,
      // default: `${today.getFullYear()}/${today.getMonth()
      //   + 1}/${today.getDate()} 00:00:00+0800`
      default: ''
    },

    disableAfter: {
      type: String,
      required: false,
      // default: `${thirty.getFullYear()}/${thirty.getMonth()
      //   + 1}/${thirty.getDate()} 00:00:00+0800`
      default: ''
    },

    colorList: {
      type: Object,
      required: false,
      default: () => ({})
    },

    info: {
      type: Object,
      required: false,
      default: () => ({})
    }
  },

  computed: {
    disableBeforeStamp() {
      return +new Date(this.disableBefore);
    },

    disableAfterStamp() {
      return +new Date(this.disableAfter);
    }
  },

  methods: {
    setMonth(diff) {
      if (this.month + diff === 13) {
        this.month = 1;
        this.year = this.year + 1;
      } else if (this.month + diff === 0) {
        this.month = 12;
        this.year = this.year - 1;
      } else {
        this.month += diff;
      }
      this.generateMonthData(this.year, this.month);
    },

    generateMonthData() {
      this.monthData = [];
      this.choosenDate = [];
      // afferent date (first day of month)
      let firstWeek = new Date(this.year, this.month - 1, 1).getDay();
      // this month last day
      let lastDate = new Date(this.year, this.month, 0),
        lastDateCount = lastDate.getDate();
      // past month last day
      let pastLastDate = new Date(this.year, this.month - 1, 0),
        pastLastDateCount = pastLastDate.getDate();

      for (let i = 0; i < 6; i++) {
        this.monthData.push([]);
        let lineFirst;
        if (i === 0) {
          lineFirst = firstWeek === 0 ? 1 : pastLastDateCount - firstWeek + 1;
        } else {
          lineFirst = 7 * i - firstWeek + 1;
        }
        for (let j = 0; j < 7; j++) {
          let info = {
            year: 0,
            month: 0,
            date: 0,
            timestamp: 0
          };
          // set date time data
          if (i === 0) {
            // in first line (handle past one month situation)
            if (lineFirst + j > pastLastDateCount) {
              // this month
              info.date = lineFirst + j - pastLastDateCount;
              info.month = this.month;
              info.year = this.year;
            } else {
              // past month
              info.date = lineFirst + j;
              info.month = this.month - 1 === 0 ? 12 : this.month - 1;
              info.year = this.month - 1 === 0 ? this.year - 1 : this.year;
              this.monthStartDate = `${this.year}${
                info.month < 10 ? '0' + info.month : info.month
              }${lineFirst < 10 ? '0' + lineFirst : lineFirst}`;
            }
          } else {
            // in other line (handle next month situation)
            if (lineFirst + j > lastDateCount) {
              // next month
              info.date = lineFirst + j - lastDateCount;
              info.month = this.month + 1 === 13 ? 1 : this.month + 1;
              info.year = this.month + 1 === 13 ? this.year + 1 : this.year;
              if (j === 6) {
                this.monthEndDate = `${this.year}${
                  info.month < 10 ? '0' + info.month : info.month
                }${info.date < 10 ? '0' + info.date : info.date}`;
              }
            } else {
              // this month
              info.date = lineFirst + j;
              info.month = this.month;
              info.year = this.year;
            }
          }
          // set other
          info.timestamp = +new Date(
            `${info.year}/${info.month}/${info.date} 00:00:00+0800`
          );
          info.disable
            = info.timestamp <= this.disableBeforeStamp
            || info.timestamp > this.disableAfterStamp;
          info.checked = false;
          if (
            info.timestamp
            === +new Date(
              `${this.thisYear}/${this.thisMonth}/${this.date} 00:00:00+0800`
            )
          ) {
            info.isToday = true;
          }
          this.monthData[i].push(info);
        }
      }
      this.$emit('chooseDate', {}, this.choosenDate);
      this.$emit('getStartEndDate', {
        sDate: this.monthStartDate,
        eDate: this.monthEndDate
      });
    },

    setArrangement(arrangement) {
      for (let i = 0; i < this.monthData.length; i++) {
        for (let j = 0; j < this.monthData[i].length; j++) {
          this.$set(
            this.monthData[i][j],
            'text',
            arrangement[this.monthData[i][j].timestamp].name
          );
          this.$set(
            this.monthData[i][j],
            'id',
            arrangement[this.monthData[i][j].timestamp].id
          );
        }
      }
    },

    chooseDate(date) {
      if (date.disable) return;
      if (date.checked) {
        date.checked = false;
        for (let i = date.index + 1; i < this.choosenDate.length; i++) {
          this.choosenDate[i].index -= 1;
        }
        this.choosenDate.splice(date.index, 1);
      } else {
        date.checked = true;
        this.choosenDate.push(date);
        date.index = this.choosenDate.length - 1;
      }
      this.$emit('chooseDate', date, this.choosenDate);
    },

    inArray(str = '', array = [], key) {
      let length = array.length;
      for (let i = 0; i < length; i++) {
        if (key) {
          if (str === array[i][key]) {
            return true;
          }
        } else {
          if (str === array[i]) {
            return true;
          }
        }
      }
      return false;
    }
  },

  mounted() {
    this.generateMonthData();
  }
};
</script>

<style lang="less" scoped>
.calendar {
  background-color: #fff;
  display: inline-block;
  padding: 15px 20px 15px;
  border: 1px solid #e8e8e8;
  .angle {
    display: inline-block;
    width: 15px;
    height: 15px;
    border-top: 2px solid #000;
    border-left: 2px solid #000;
  }
  .cal-head {
    padding: 5px 10px 10px;
    .left-con {
      float: left;
      padding: 10px 20px;
      cursor: pointer;
      .angle {
        transform: rotate(-45deg);
      }
    }
    .right-con {
      float: right;
      padding: 10px 20px;
      cursor: pointer;
      .angle {
        transform: rotate(135deg);
      }
    }
    .info {
      padding-top: 3px;
      text-align: center;
      font-size: 17px;
    }
  }
  table {
    border: none;
    tr {
      td,
      th {
        text-align: center;
        width: 70px;
      }
      th {
        padding: 0 0 20px;
      }
      td {
        padding: 0 0 28px;
        font-size: 20px;
        position: relative;
        span {
          color: #000;
          cursor: pointer;
          user-select: none;
          display: inline-block;
          width: 60%;
          border-radius: 15px;
          &:hover {
            background-color: #609d2f;
          }
        }
        .rest {
          background-color: #ffb018;
          color: #fff;
        }
        .nine {
          background-color: #54bc00;
          color: #fff;
        }
        .ten {
          background-color: #398000;
          color: #fff;
        }
        .eleven {
          background-color: #295b00;
          color: #fff;
        }
        .text {
          position: absolute;
          font-size: 12px;
          width: 100%;
          top: 32px;
          left: 0;
          color: #555;
          overflow: hidden;
          height: 15px;
        }
        .today {
          background-color: #ffcc90;
        }
        .uncheckable {
          cursor: default;
          color: #999;
          background-color: transparent !important;
        }
      }
      .choosen {
        span {
          background-color: #ea6151 !important;
          color: #fff;
        }
      }
    }
  }
}
</style>
