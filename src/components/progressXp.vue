
<template>
  <main>
    <div id="container"></div>
    <span hidden v-if="progressXp">{{ orderedlist() }}</span>
  </main>
</template>
<script>
import moment from "moment";
import _ from "lodash";

export default {
  props: {
    progressXp: Array,
  },
  data() {
    return {
      all_data: [],
      xpByMonths: [],
      i: 0,
    };
  },

  methods: {
    moment(date) {
      return moment(date).format("DD-MM-YYYY, hh:mm A");
    },
    orderedlist() {
      if (this.progressXp.length != 0) {
        let a = _.orderBy(this.progressXp, ["finishedAt"], ["asc"]);
        this.devideByMonth(a);
        this.i = 1;
        return a;
      }
    },
    devideByMonth(data) {
      const monthNames = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];

      let oneMonth = [];
      let allMonths = [];
      let previousTask;
      let xp = 0;
      data.forEach((task, i) => {
        if (i == 0) {
          xp += task.taskXp;
        } else if (data.length - 1 == i) {
          if (task.finishedAt[6] != previousTask.finishedAt[6]) {
            var date = new Date(previousTask.finishedAt);
            oneMonth.push(monthNames[date.getMonth()], xp);
            allMonths.push(oneMonth);
            oneMonth = [];
          }
          xp += task.taskXp;
          var date = new Date(task.finishedAt);
          oneMonth.push(monthNames[date.getMonth()], xp);
          allMonths.push(oneMonth);
          oneMonth = [];
        } else {
          if (
            task.finishedAt[6] == previousTask.finishedAt[6] &&
            task.finishedAt[5] == previousTask.finishedAt[5]
          ) {
            xp += task.taskXp;
          } else {
            var date = new Date(previousTask.finishedAt);
            oneMonth.push(monthNames[date.getMonth()], xp);
            allMonths.push(oneMonth);
            oneMonth = [];
            xp += task.taskXp;
          }
        }
        previousTask = task;
      });
      let monthLevel = [];
      allMonths.forEach((month) => {
        let lvl = this.calculateLevel(month[1]);
        monthLevel.push([month[0], lvl]);
      });

      this.chart(allMonths);
      this.chart2(monthLevel);
    },
    // Calculates what level this amount of XP would be at
    calculateLevel(xp) {
      let level = 0;
      while (this.levelNeededXP(++level) < xp) {}
      return level - 1;
    },
    levelNeededXP(level) {
      return Math.round(level * (176 + 3 * level * (47 + 11 * level)));
    },
    chart(allMonths) {
      anychart.onDocumentReady(function () {
        let data = allMonths;
        var chart = anychart.line();
        var series = chart.line(data);

        chart.xScale().mode("continuous");
        chart.title("Progress over time (XP)");
        series.name("XP");
        var xAxis = chart.xAxis();
        xAxis.title("Month");
        var yAxis = chart.yAxis();
        yAxis.title("XP");

        var elem = document.getElementById("container");
        elem.remove();
        let div = document.createElement("div");
        div.id = "container";
        document.body.appendChild(div);
        chart.container("container");
        div.className = "chart";
        chart.draw();
      });
    },
    chart2(allMonths) {
      anychart.onDocumentReady(function () {
        let data = allMonths;
        var chart = anychart.line();
        var series = chart.line(data);
        chart.xScale().mode("continuous");
        chart.title("Progress over time (Level)");
        series.name("Level");

        var xAxis = chart.xAxis();
        xAxis.title("Month");
        var yAxis = chart.yAxis();
        yAxis.title("Level");

        var elem = document.getElementById("container2");
        if (elem != null) {
          elem.remove();
        }
        let div = document.createElement("div");
        div.id = "container2";
        div.className = "chart";
        document.body.appendChild(div);
        chart.container("container2");
        chart.draw();
      });
    },
  },
  computed: {},
};
</script>
<style>
@import "@/assets/chart.css";
</style>