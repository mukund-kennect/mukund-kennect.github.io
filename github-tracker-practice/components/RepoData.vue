<template>

  <v-container fluid v-if="totalIssues.length && allData.length">
    <!-- <p>Collecting Issues : {{ totalIssues.length }}</p> -->

    <div class="mb-5">
      <v-expansion-panels> 
        <v-expansion-panel v-for="data in filteredAllData" :key="data.year" :title="data.year" class="font-weight-bold">
          <v-expansion-panel-text>
            <v-expansion-panels>
              <v-expansion-panel v-for="(mData, key) in data.monthlyData.filter(x => x !== null)" :key="key"  class="font-weight-medium">
                <v-expansion-panel-title>
                  {{ mData.monthName }}
                  , open : {{ mData.openIsseus }}
                  , closed : {{ mData.closed }}
                  , ratio : {{ mData.ratio }}
                  , closure rate : {{ mData.closureRate }}
                  , pending : {{ mData.pendingIssues }}
                </v-expansion-panel-title>
                <v-expansion-panel-text>
                  <v-list lines="four">
                    <v-list-item v-for="(wData, key) in mData.weeklyData.filter(x => x !== null)"
                       :key="key" class="mb-4 mt-2"
                    >
                    <v-list-item-title class="font-weight-bold">
                      Week {{ key + 1 }}
                      </v-list-item-title>
                      <v-list-item-subtitle class="text-subtitle-2 mb-1">
                    <p> open : {{ wData.openIsseus }}</p>
                    <p>closed : {{ wData.closed }}</p>
                    <p>ratio : {{ wData.ratio }}</p>
                    <p>closure rate : {{ wData.closureRate }}</p>
                    <p>pending : {{ wData.pendingIssues }}</p>
                    </v-list-item-subtitle>
                  </v-list-item>
                  </v-list>
                </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-expansion-panel-text>
        </v-expansion-panel>
      </v-expansion-panels>
    </div>

    <v-table theme="dark" fixed-header height="550px">
      <thead>
        <tr >
          <th class="text-left">
            <v-btn flat @click="sortBy('number')" class="font-weight-bold">Number
              <v-icon end icon="mdi-arrow-up"></v-icon>
            </v-btn>

          </th>

          <th class="text-left">
            <v-btn flat @click="sortBy('title')" class="font-weight-bold">Title
              <v-icon end icon="mdi-arrow-up"></v-icon>
            </v-btn>
          </th>
          <th class="text-left">
            <v-btn flat @click="sortBy('created_at')" class="font-weight-bold">Created at
              <v-icon end icon="mdi-arrow-up"></v-icon>
            </v-btn>
          </th>
          <th class="text-left">
            <v-btn flat @click="sortBy('state')" class="font-weight-bold">State
              <v-icon end icon="mdi-arrow-up"></v-icon>
            </v-btn>
          </th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="item in totalIssues" :key="item.number">
          <td>{{ item.number }}</td>
          <td>{{ item.title }}</td>
          <td>{{ item.created_at }}</td>
          <td>{{ item.state }}</td>
        </tr>
      </tbody>

    </v-table>



  </v-container>
  <div v-else>
    <div v-if="isRepoAvailable">
    <v-progress-linear indeterminate color="gray" rounded class="mb-2"></v-progress-linear>
    <h2 class="text-h4 text-center">Collecting Data....</h2>
   </div>
    <div v-else>
      <h2 class="text-h4 text-center">No Data Found...</h2>
    </div>
  </div>

</template>

<script>

export default {
  props: ["username", "reponame"],
  data() {
    return {
      isRepoAvailable: true,
      isIssueAvailable:true,
      currentPage: 0,
      totalPages: 1,
      totalIssues: [],
      allData: [],
      githubUsername: this.username,
      githubRepo: this.reponame,
      remainingOpenISsues: 0,
    };
  },
  watch: {
    currentPage: function (value) {
      if (value == this.totalPages) {
        setTimeout(
          this.filterGithubData
          , 5000)
      }
    }
  },
  computed: {
    filteredAllData: function () {
      return this.allData.filter(function (item) {
        return item !== null;
      });
    }
  },
  methods: {

    sortBy(parameter) {
      this.totalIssues.sort((a, b) => a[parameter] < b[parameter] ? -1 : 1);
    },
    addIssues(issue) {
      console.log('Added the issue');
      this.totalIssues.push(issue);
    },

    calculateRatio(num_1, num_2) {
      for (let num = num_2; num > 1; num--) {
        if (num_1 % num == 0 && num_2 % num == 0) {
          num_1 = num_1 / num;
          num_2 = num_2 / num;
        }
      }
      var ratio = num_1 + ":" + num_2;
      return ratio;
    },
    filterGithubData() {
      console.log("Called Function");

      const listOfMonths = [
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
      for (const issue of this.totalIssues) {

        const date = new Date(issue.created_at);
        const yearFromDate = Number(date.getFullYear());
        if (!this.allData[yearFromDate]) {
          this.allData[yearFromDate] = {};
        }
        this.allData[yearFromDate]["year"] = String(yearFromDate);

        const monthFromDate = Number(date.getMonth());

        if (!this.allData[yearFromDate]["monthlyData"]) {
          this.allData[yearFromDate]["monthlyData"] = [];
        }
        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate] = {};
          this.allData[yearFromDate]["monthlyData"][monthFromDate].monthName =
            listOfMonths[monthFromDate];
          this.allData[yearFromDate]["monthlyData"][monthFromDate].openIsseus = 1;
          this.allData[yearFromDate]["monthlyData"][monthFromDate].closed = 0;
        }
        else {
          this.allData[yearFromDate]["monthlyData"][monthFromDate].openIsseus += 1;
        }
        const datte = new Date(date.getFullYear(), date.getMonth(), 1);
        const weekNumber = (0 | (datte.getDay() - 2 + date.getDate()) / 7) + 1;

        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData']) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'] = [];
        }
        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber]) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber] = {};
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].openIsseus = 1;
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].closed = 0;
        } else {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].openIsseus += 1;
        }


      }
      for (const issue of this.totalIssues) {

        const date = new Date(issue.closed_at);
        const yearFromDate = Number(date.getFullYear());
        if (!this.allData[yearFromDate]) {
          this.allData[yearFromDate] = {};
        }
        this.allData[yearFromDate]["year"] = String(yearFromDate);

        const monthFromDate = Number(date.getMonth());

        if (!this.allData[yearFromDate]["monthlyData"]) {
          this.allData[yearFromDate]["monthlyData"] = [];
        }
        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate] = {};
          this.allData[yearFromDate]["monthlyData"][monthFromDate].monthName =
            listOfMonths[monthFromDate];
          this.allData[yearFromDate]["monthlyData"][monthFromDate].closed = 1;
          if (!this.allData[yearFromDate]["monthlyData"][monthFromDate].openIsseus) {
            this.allData[yearFromDate]["monthlyData"][monthFromDate].openIsseus = 0;
          }
        }
        else {
          this.allData[yearFromDate]["monthlyData"][monthFromDate].closed += 1;
        }

        const datte = new Date(date.getFullYear(), date.getMonth(), 1);
        const weekNumber = (0 | (datte.getDay() - 2 + date.getDate()) / 7) + 1;
        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData']) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'] = [];
        }
        if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber]) {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber] = {};
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].closed = 1;
          if (!this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].openIsseus) {
            this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].openIsseus = 0;
          }
        } else {
          this.allData[yearFromDate]["monthlyData"][monthFromDate]['weeklyData'][weekNumber].closed += 1;
        }

      }


      for (const yearlyData of this.allData) {
        if (yearlyData && yearlyData.year >= 2008) {
          for (const monthlyData of yearlyData.monthlyData) {
            if (monthlyData) {
              monthlyData.ratio = this.calculateRatio(monthlyData.openIsseus, monthlyData.closed);
              monthlyData.pendingIssues = this.remainingOpenISsues + (monthlyData.openIsseus - monthlyData.closed);
              const closureRate = (monthlyData.closed) / (monthlyData.openIsseus + this.remainingOpenISsues);
              monthlyData.closureRate = closureRate.toFixed(2);
              for (const weeklyData of monthlyData.weeklyData) {
                if (weeklyData) {
                  weeklyData.ratio = this.calculateRatio(weeklyData.openIsseus, weeklyData.closed);
                  weeklyData.pendingIssues = this.remainingOpenISsues + (weeklyData.openIsseus - weeklyData.closed);
                  const closureRate = (weeklyData.closed) / (weeklyData.openIsseus + this.remainingOpenISsues);
                  weeklyData.closureRate = closureRate.toFixed(2);
                  this.remainingOpenISsues += (weeklyData.openIsseus - weeklyData.closed);

                }
              }

            }
          }

        }


      }

    },

    pageNumberFinder(data) {
      const res = data.match(/(https?:\/\/[^>;]*)/g);
      const params = new URL(res[1]).searchParams;
      return params.get('page');
    },

    getData() {
      fetch(`https://api.github.com/repos/${this.githubUsername}/${this.githubRepo}/issues?per_page=100&state=all&sort=created&page=${this.currentPage}`)
        .then((response) => {
           if(!response.ok || response.json().length>0){
            this.isRepoAvailable=false;
           }
           if (response.headers.has('link') && response.headers.get('link') != null) {
            return this.pageNumberFinder(JSON.stringify(response.headers.get('link')));
          } else {
            return 1;
          }
        })
        .then(data => this.totalPages = data)
        .then((totalPages) => {
          while (this.currentPage < totalPages) {
            this.currentPage++;
            fetch(`https://api.github.com/repos/${this.githubUsername}/${this.githubRepo}/issues?per_page=100&state=all&direction=desc&page=${this.currentPage}`)
              .then((response) => {
                return response.json();
              })
              .then((data) => {
                for (let issue of data) {
                  if (issue.pull_request == null) {
                    this.addIssues(issue);
                  }
                }
              })
              .catch(function (error) {
                console.log('Error : ', error);
              });
          }
        })
        .catch(function (error) {
          console.log('Error : ', error);
        })
        .finally(() => {
          console.log('Data Received Successfully');
        });

    }
  },
  mounted() {
    this.getData()
  }

}


</script>

<style>

</style>

