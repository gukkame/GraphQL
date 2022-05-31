
<template>
  <main>
    <div class="tbl-header">
      <table>
        <thead>
          <tr>
            <th>Task</th>
            <th>XP</th>
            <th>Date</th>
          </tr>
        </thead>
      </table>
    </div>

    <div class="tbl-content">
      <table>
        <tbody>
          <tr v-for="data in orderedlist" :key="data._id">
            <td>{{ data.name }}</td>
            <td>{{ data.taskXp }}</td>
            <td>{{ moment(data.finishedAt) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>
<script>
import moment from "moment";
import _ from "lodash";

export default {
  name: "lastActivity",
  props: {
    lastActivity: Array,
  },
  methods: {
    moment(date) {
      return moment(date).format("DD-MM-YYYY, hh:mm A");
    },
  },
  computed: {
    orderedlist() {
      let a = _.orderBy(this.lastActivity, ["finishedAt"], ["desc"]);
      return a;
    },
  },
};
</script>
<style>
@import "@/assets/table.css";
</style>