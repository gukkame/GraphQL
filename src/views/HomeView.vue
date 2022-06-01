<script setup>
import LastActivity from "../components/lastActivity.vue";
import ProgressXP from "../components/progressXp.vue";
</script>
<template>
  <body>
    <nav class="topnav">
      <div>
        <h2>GraphQL</h2>
        <h2>Made by gukka!</h2>
      </div>
    </nav>

    <main>
      <form @submit.prevent="submitUsername()" class="searchBar">
        <div>
          <input
            class="searchBarInp"
            type="text"
            placeholder="Search User..."
          />
        </div>
        <input class="searchBarBtn" type="submit" value="Search!" />
      </form>

      <div class="container" v-if="msg.length == 0">
        <div class="profile">
          <h2>{{ displayUserName }}</h2>
          <h3>{{ lvl }} Level</h3>
          <h3>{{ xptotal }} XP total</h3>
        </div>
        <h1><strong> Last Activity</strong></h1>
        <div class="lastActivity">
          <LastActivity :lastActivity="lastActivity" />
        </div>
        <br />
        <div class="progressTimeXp">
          <ProgressXP :progressXp="lastActivity" />
        </div>
      </div>
      <div v-else-if="msg == 'N'"></div>
      <div v-else>
        <h3>{{ msg }}</h3>
      </div>
    </main>
  </body>
</template>
<script>
import axios from "axios";

export default {
  components: {
    LastActivity,
    ProgressXP,
  },
  data() {
    return {
      username: "",
      displayUserName: "",
      data: [],
      msg: "N",
      doneTaskIds: [],
      lvl: 0,
      xptotal: 0,
      lastActivity: [],
      internalProperty: "",

      progress: [],
      transactions: [],
    };
  },

  methods: {
    async submitUsername() {
      const val = document.querySelector("input").value;
      this.lvl = 0;
      this.xptotal = 0;
      this.displayUserName = val;
      this.username = "";

      const endpoint = "https://01.kood.tech/api/graphql-engine/v1/graphql";
      const headers = {
        "content-type": "application/json",
      };
      const graphqlQuery = {
        query: `query ($username: String){
      user(where: {login: {_eq:$username}}, limit: 50, offset: 0) {
    id
    login
#     I get info about each task, when i finished, name of project and taskID
     progresses(where:{_and: [{isDone:{_eq: true},path: {_iregex: "div-01/(?!piscine-js-2/)"}}] }
    ) {
      isDone
      objectId
      updatedAt
      results {
        object {
          type
          name
        }
      }
    }
#     Get info about task points(audit,tasks), task name, taskID
    transactions(where: {type: {_eq: "xp"}, path: {_iregex: "div-01/(?!piscine-js-2/)"}}) {
      userId
      objectId
      type
      amount
      object {
        type
        name
      }
    }
  }
}`,
        variables: {
          username: this.displayUserName,
        },
      };
      const graphqlQuery3 = {
        query: `query ($username: String){
      user(where: {login: {_eq:$username}}, limit: 50, offset: 0) {
    id
    login
#     I get info about each task, when i finished, name of project and taskID
     progresses(where: {_and: [{isDone: {_eq: true}, path: {_iregex: "div-01/(?!piscine-js/)"},_or:{
          path: {_iregex: "div-01/(?!rust)"}
       }}]},offset:0 
    ) {
      isDone
      objectId
      updatedAt
      results {
        object {
          type
          name
        }
      }
    }
#     Get info about task points(audit,tasks), task name, taskID
    transactions(where: {type: {_eq: "xp"}, path: {_iregex: "div-01/(?!piscine-js-2/)"}}) {
      userId
      objectId
      type
      amount
      object {
        type
        name
      }
    }
  }
}`,
        variables: {
          username: this.displayUserName,
        },
      };
      const graphqlQuery2 = {
        query: `query ($username: String){user(where: {login: {_eq:$username}}, limit: 1, offset: 0) {
    id
    login
    progresses(where:{_and: [{isDone:{_eq: true},path: {_iregex: "div-01/(?!piscine-js-2/)"},results:{object:{name:{_eq:"declarations"}}}}] },limit: 1,
    ){
      path
      isDone
      results {
        object {
          type
          name
        }
      }
    }
  }
}`,
        variables: {
          username: this.displayUserName,
        },
      };

      let response2 = await axios({
        url: endpoint,
        method: "post",
        headers: headers,
        data: graphqlQuery2,
      })
        .then((result) => {
          return result;
        })
        .catch((error) => {
          console.error(error);
        });
      let response1;
      if (response2.data.data.user[0].progresses.length == 0) {
        response1 = await axios({
          url: endpoint,
          method: "post",
          headers: headers,
          data: graphqlQuery,
        })
          .then((result) => {
            return result;
          })
          .catch((error) => {
            console.error(error);
          });
      } else {
        response1 = await axios({
          url: endpoint,
          method: "post",
          headers: headers,
          data: graphqlQuery3,
        })
          .then((result) => {
            return result;
          })
          .catch((error) => {
            console.error(error);
          });
      }

      if (
        response1.data.data !== undefined &&
        response1.data.data.user[0] !== undefined
      ) {
        this.msg = "";
      } else {
        this.msg = "No users found!";
        return;
      }

      let progress = response1.data.data.user[0].progresses;

      let transactions = response1.data.data.user[0].transactions;

      // Runs if user have made more than 50 transations
      if (transactions.length == 50) {
        for (let i = 50; i <= 300; i = i + 50) {
          let response2 = await axios({
            url: endpoint,
            method: "post",
            headers: headers,
            data: {
              query:
                `query {user(where: {login: {_eq: ` +
                this.displayUserName +
                ` }}, limit: 50, offset: 0) {transactions( where: {type: {_eq: "xp"}, path: {_iregex: "div-01/(?!piscine-js-2/)"}},offset: ` +
                i +
                `) {userId objectId type amount object { type name }}}} `,
            },
          })
            .then((result1) => {
              let trns2 = result1.data.data.user[0].transactions;
              return trns2;
            })
            .catch((error) => {
              console.error(error);
            });
          transactions = transactions.concat(response2);
        }
      }
      this.getTaskXp(progress, transactions);
      this.lastActivityData(progress, transactions);
    },
    taskIdXP(progress, transactions) {
      this.doneTaskIds = [];

      var xp = [];
      progress.forEach((obj) => {
        this.doneTaskIds.push(obj.objectId);
      });
      transactions.forEach((obj) => {
        var i = 0;
        xp.forEach((task) => {
          if (task[0] == obj.objectId && task[1] == obj.amount) {
            i++;
          }
        });
        if (i == 0) {
          xp.push([obj.objectId, obj.amount]);
        }
      });
      return xp;
    },
    lastActivityData(progress, transactions) {
      let xp = this.taskIdXP(progress, transactions);

      //Compares taskId, xp point amount and takes the biggest one and pushes to allDoneTaskXp
      this.lastActivity = [];
      var allDoneTaskXp = [];
      this.doneTaskIds.forEach((taskId) => {
        var arr = [];
        //Gets tasks I have done
        xp.forEach((taskxp) => {
          if (taskxp[0] == taskId) {
            arr.push(taskxp);
          }
        });

        if (arr.length > 1) {
          if (arr[0][1] >= arr[1][1]) {
            allDoneTaskXp.push(arr[0]);
          } else {
            allDoneTaskXp.push(arr[1]);
          }
        } else if (arr.length != 0) {
          allDoneTaskXp.push(arr[0]);
        }
      });
      allDoneTaskXp.forEach((id_xp) => {
        progress.forEach((taskinfo) => {
          if (id_xp[0] == taskinfo.objectId) {
            let arr = {
              name: taskinfo.results[0].object.name,
              taskXp: id_xp[1],
              finishedAt: taskinfo.updatedAt,
            };
            this.lastActivity.push(arr);
          }
        });
      });
      console.log("Last activity data", this.lastActivity);
    },

    getTaskXp(progress, transactions) {
      let xp = this.taskIdXP(progress, transactions);
      //Compares taskId, xp point amount and takes the biggest one and pushes to allDoneTaskXp
      var allDoneTaskXp = [];
      this.doneTaskIds.forEach((taskId) => {
        var arr = [];
        xp.forEach((taskxp) => {
          if (taskxp[0] == taskId) {
            arr.push(taskxp);
          }
        });

        if (arr.length > 1) {
          if (arr[0][1] >= arr[1][1]) {
            allDoneTaskXp.push(arr[0][1]);
          } else {
            allDoneTaskXp.push(arr[1][1]);
          }
        } else if (arr.length != 0) {
          allDoneTaskXp.push(arr[0][1]);
        }
      });
      //Sums all xp and calculates to lvl
      for (let i = 0; i < allDoneTaskXp.length; i++) {
        this.xptotal += allDoneTaskXp[i];
      }
      this.lvl = this.calculateLevel(this.xptotal);
    },
    // Calculates what level this amount of XP would be at
    calculateLevel(xp) {
      let level = 0;
      while (this.levelNeededXP(++level) < xp) {}
      return level - 1;
    },

    // Returns the amount of XP needed for any given level
    levelNeededXP(level) {
      return Math.round(level * (176 + 3 * level * (47 + 11 * level)));
    },
  },
};
</script>
