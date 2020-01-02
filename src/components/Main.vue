<template>
  <v-content class="mt-2 px-5">
    <v-row>
      <v-col class="d-flex flex-column" cols="3">
        <v-card class="mb-5">
          <v-card-title>
            <span>Purposes</span>
          </v-card-title>
          <v-card-text class="py-0">
            <v-combobox
              v-model="selectedPurposes"
              :items="purposeList"
              label="Selected Purposes"
              multiple
              outlined
              small-chips
            />
          </v-card-text>
          <v-card-actions class="my-0">
            <v-spacer />
            <v-btn color="primary" text @click="onSelectedPurposesChange">Update</v-btn>
          </v-card-actions>
        </v-card>

        <v-card>
          <v-card-title>
            <span>WMA Rules</span>
            <v-spacer />
            <v-tooltip right>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" icon @click="ruleDialog = true">
                  <v-icon color="success">mdi-plus</v-icon>
                </v-btn>
              </template>
              <span>Add</span>
            </v-tooltip>
          </v-card-title>
          <v-card-text v-if="rules.length > 0">
            <v-row class="px-1">
              <v-col class="my-0 py-0" cols="2">
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon v-on="on" small>mdi-calendar</v-icon>
                  </template>
                  <span>Months Before</span>
                </v-tooltip>
              </v-col>
              <v-col class="my-0 py-0" cols="2">
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon v-on="on" small>mdi-star-half</v-icon>
                  </template>
                  <span>Weight</span>
                </v-tooltip>
              </v-col>
            </v-row>
            <v-row v-for="(item, i) in rules" :key="i">
              <v-col class="ms-2 my-0 py-0 d-flex align-center" cols="2">
                <span class="caption" v-text="rules[i].months" />
              </v-col>
              <v-col class="my-0 py-0 d-flex align-center" cols="2">
                <span class="caption" v-text="rules[i].weight" />
              </v-col>
              <v-spacer class="mx-0 px-0" />
              <v-tooltip right>
                <template v-slot:activator="{ on }">
                  <v-btn color="blue" v-on="on" icon small @click="onEditRule(i)">
                    <v-icon small>mdi-pencil</v-icon>
                  </v-btn>
                </template>
                <span>Edit</span>
              </v-tooltip>
              <v-tooltip right>
                <template v-slot:activator="{ on }">
                  <v-btn class="me-4" v-on="on" color="red" icon small @click="onRemoveRule(i)">
                    <v-icon small>mdi-close</v-icon>
                  </v-btn>
                </template>
                <span>Remove</span>
              </v-tooltip>

              <v-col class="my-0 py-0" cols="12">
                <v-divider />
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions v-if="rules.length > 0">
            <v-spacer />
            <v-btn color="primary" text @click="onRun(false)">Run</v-btn>
          </v-card-actions>
        </v-card>

        <v-card class="mt-5">
          <v-card-title>History</v-card-title>
          <v-card-text>
            <v-row v-if="history.length > 0">
              <v-col class="my-0 py-0 d-flex flex-row align-center" cols="12">
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon class="ms-1 me-1" v-on="on" small>mdi-clock</v-icon>
                  </template>
                  <span>Time</span>
                </v-tooltip>
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon class="ms-12" v-on="on" small>mdi-close</v-icon>
                  </template>
                  <span>Average RMSE</span>
                </v-tooltip>
              </v-col>

              <v-col class="my-0 py-0" cols="12" v-for="(item, i) in history" :key="i">
                <div class="d-flex flex-row align-center">
                  <span class="ms-1 caption" v-text="item.time" />
                  <span class="ms-5 caption" v-text="caculateAvg(i)" />
                  <v-spacer />
                  <v-tooltip right>
                    <template v-slot:activator="{ on }">
                      <v-btn color="grey" icon small v-on="on" @click="onShowHistoryDetail(i)">
                        <v-icon small>mdi-dots-vertical</v-icon>
                      </v-btn>
                    </template>
                    <span>Details</span>
                  </v-tooltip>
                  <v-tooltip right>
                    <template v-slot:activator="{ on }">
                      <v-btn
                        class="me-1"
                        color="blue"
                        icon
                        small
                        v-on="on"
                        @click="onRunHistory(i)"
                      >
                        <v-icon small>mdi-refresh</v-icon>
                      </v-btn>
                    </template>
                    <span>Rerun</span>
                  </v-tooltip>
                  <v-tooltip right>
                    <template v-slot:activator="{ on }">
                      <v-btn
                        class="me-1"
                        color="red"
                        icon
                        small
                        v-on="on"
                        @click="onRemoveHistory(i)"
                      >
                        <v-icon small>mdi-close</v-icon>
                      </v-btn>
                    </template>
                    <span>Remove</span>
                  </v-tooltip>
                </div>
                <v-divider class="mb-1 py-0" v-if="i != history.length - 1" />
              </v-col>
            </v-row>
            <span v-else>There is no history yet!</span>
          </v-card-text>
        </v-card>
      </v-col>

      <v-col class="d-flex flex-column" cols="9">
        <v-card>
          <v-card-title>Visitors By Purposes</v-card-title>
          <v-card-text>
            <GChart type="LineChart" :data="chartData1" :options="chartOptions" />
          </v-card-text>
        </v-card>

        <!-- <v-card class="mt-5" v-if="chartData3.length > 0">
          <v-card-title>Predict By Rules</v-card-title>
          <v-card-text>
            <GChart type="LineChart" :data="chartData3" :options="chartOptions" />
          </v-card-text>
        </v-card>-->

        <v-card class="mt-5">
          <v-card-title>Summarize</v-card-title>
          <v-card-text>
            <GChart type="LineChart" :data="chartData2" :options="chartOptions" />
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="ruleDialog" width="500">
      <v-card>
        <v-card-title class="primary--text" v-text="isEditingRule?'Edit Rule':'Add A New Rule'" />
        <v-card-text>
          <v-text-field v-model="months" label="Months Before" type="number"></v-text-field>
          <v-text-field v-model="weight" label="Weight" type="number"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="ruleDialog = false; isEditingRule = false;">Cancel</v-btn>
          <v-btn color="primary" text @click="onSaveRule">Done</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="historyDialog" width="800" scrollable>
      <v-card v-if="history.length > 0">
        <v-card-title>
          <span class="primary--text" v-text="history[selectedHistoryIndex].time" />
          <v-spacer />
          <v-btn color="red" icon @click="historyDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12">
              <span class="body-1">Each Purposes' RMSE</span>
              <div class="mt-2 px-5 mb-2 d-flex flex-row justify-space-between align-center">
                <div
                  class="me-5 d-flex flex-column align-center"
                  v-for="(item, i) in history[selectedHistoryIndex].devations"
                  :key="i"
                >
                  <span class="caption" v-text="purposeList[i]" />
                  <span class="caption" v-text="item" />
                </div>
              </div>
              <v-divider class="mb-5" />
              <span class="body-1">WMA Rules</span>
              <div class="mt-2 px-5 d-flex flex-row justify-space-around">
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon v-on="on">mdi-calendar</v-icon>
                  </template>
                  <span>Months Before</span>
                </v-tooltip>
                <v-tooltip right>
                  <template v-slot:activator="{ on }">
                    <v-icon v-on="on">mdi-star-half</v-icon>
                  </template>
                  <span>Weight</span>
                </v-tooltip>
              </div>

              <div v-for="(item, i) in history[selectedHistoryIndex].rules" :key="i">
                <div class="mt-2 px-5 d-flex flex-row justify-space-around">
                  <span class="caption" v-text="item.months" />
                  <span class="caption" v-text="item.weight" />
                </div>
                <v-divider v-if="i != history[selectedHistoryIndex].rules.length - 1" />
              </div>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-snackbar v-model="snackbar" :color="snackbarColor" left bottom>
      <div class="d-flex flex-row justify-start align-center">
        <v-icon v-text="snackbarIcon" dark />
        <span class="ps-5" v-text="snackbarMessage" />
      </div>
    </v-snackbar>
  </v-content>
</template>

<script>
export default {
  data() {
    return {
      snackbar: false,
      snackbarMessage: "",
      snackbarColor: "success",
      snackbarIcon: "mdi-alert",
      historyDialog: false,
      selectedHistoryIndex: 0,
      ruleDialog: false,
      isEditingRule: false,
      editingRuleIndex: 0,
      rules: [],
      months: 1,
      weight: 1,
      purposeSumSource: [],
      chartData1: [],
      chartData2: [],
      chartData3: [],
      chartOptions: {
        title: "",
        height: 350,
        animation: {
          startup: true,
          duration: 2000,
          easing: "inAndOut"
        }
      },
      purposeList: [
        "Business",
        "Pleasure",
        "Visit Relatives",
        "Conference",
        "Study",
        "Exhibition",
        "Medical Treatment",
        "Others",
        "Unstated"
      ],
      selectedPurposes: ["Business"],
      history: []
    };
  },
  props: {
    source: Array
  },
  methods: {
    showSnackbar(icon, color, msg) {
      this.snackbar = true;
      this.snackbarIcon = icon;
      this.snackbarColor = color;
      this.snackbarMessage = msg;
    },
    onEditRule(index) {
      this.isEditingRule = true;
      this.ruleDialog = true;
      this.editingRuleIndex = index;

      this.months = this.rules[index].months;
      this.weight = this.rules[index].weight;
    },
    onSaveRule() {
      for (let i in this.rules) {
        if (this.isEditingRule) {
          if (
            this.rules[i].months == this.months &&
            i != this.editingRuleIndex
          ) {
            this.showSnackbar(
              "mdi-alert",
              "error",
              "The rule is already added!"
            );
            return;
          }
        } else {
          if (this.rules[i].months == this.months) {
            this.showSnackbar(
              "mdi-alert",
              "error",
              "The rule is already added!"
            );
            return;
          }
        }
      }

      if (this.isEditingRule) {
        this.rules[this.editingRuleIndex].months = this.months;
        this.rules[this.editingRuleIndex].weight = this.weight;
        this.showSnackbar("mdi-check", "success", "Changes saved!");
      } else {
        this.rules.push({
          months: this.months,
          weight: this.weight
        });

        this.showSnackbar("mdi-check", "success", "New rule added!");
      }

      this.isEditingRule = false;
      this.ruleDialog = false;
    },
    onRemoveRule(index) {
      this.rules.splice(index, 1);
    },
    drawPurposes() {
      let maxMonths = 0;
      for (let rule of this.rules) {
        if (parseInt(rule.months) > maxMonths) {
          maxMonths = parseInt(rule.months);
        }
      }

      this.chartData1 = [];

      let indexs = [];
      for (let purpose of this.purposeList) {
        if (this.selectedPurposes.indexOf(purpose) == -1) {
          indexs.push(this.purposeList.indexOf(purpose));
        }
      }

      indexs.sort();
      indexs.reverse();

      let cloneSumSource = JSON.parse(JSON.stringify(this.purposeSumSource));
      for (let i in cloneSumSource) {
        for (let j of indexs) {
          cloneSumSource[i].splice(j + 1, 1);
        }

        if (i == 0 || i > maxMonths) {
          this.chartData1.push(cloneSumSource[i]);
        }
      }
    },
    drawSummarizes() {
      this.chartData2 = [];
      this.chartData2.push(["Period", "Summarize"]);

      let datas = [];
      let indexs = [];
      for (let purpose of this.purposeList) {
        if (this.selectedPurposes.indexOf(purpose) == -1) {
          indexs.push(this.purposeList.indexOf(purpose));
        }
      }

      indexs.sort();
      indexs.reverse();

      let cloneSumSource = JSON.parse(JSON.stringify(this.purposeSumSource));
      for (let s of cloneSumSource) {
        for (let i of indexs) {
          s.splice(i + 1, 1);
        }

        datas.push(s);
      }

      for (let i in datas) {
        if (i != 0) {
          let summarize = 0;

          for (let j in datas[i]) {
            if (j != 0) {
              summarize += parseInt(datas[i][j]);
            }
          }
          this.chartData2.push([datas[i][0], summarize]);
        }
      }
    },
    onShowHistoryDetail(index) {
      this.selectedHistoryIndex = index;
      this.historyDialog = true;
    },
    onRemoveHistory(index) {
      this.history.splice(index, 1);
    },
    addNewHistory(devations) {
      let date = new Date();
      let hours = date.getHours();
      let mins = date.getMinutes();
      let secs = date.getSeconds();

      if (hours < 10) {
        hours = `0${hours}`;
      }
      if (mins < 10) {
        mins = `0${mins}`;
      }
      if (secs < 10) {
        secs = `0${secs}`;
      }
      let time = `${hours}:${mins}:${secs}`;
      let h = {
        time: time,
        rules: JSON.parse(JSON.stringify(this.rules)),
        devations: devations
      };
      this.history.splice(0, 0, h);
    },
    onRun(isHistory) {
      this.drawPurposes();

      // Calculate Predict Data
      let maxMonths = 0;
      let weights = 0;
      for (let rule of this.rules) {
        if (parseInt(rule.months) > maxMonths) {
          maxMonths = parseInt(rule.months);
        }
        weights += parseInt(rule.weight);
      }

      let t = [];
      let data = [];
      for (let i in this.purposeSumSource) {
        if (i == 0) {
          data.push(this.purposeSumSource[i]);
        } else {
          if (i > maxMonths) {
            let business = 0;
            let pleasure = 0;
            let visit = 0;
            let conference = 0;
            let study = 0;
            let exhibition = 0;
            let medical = 0;
            let others = 0;
            let unstated = 0;

            for (let rule of this.rules) {
              let months = rule.months;
              let weight = rule.weight;

              business += this.purposeSumSource[i - months][1] * weight;
              pleasure += this.purposeSumSource[i - months][2] * weight;
              visit += this.purposeSumSource[i - months][3] * weight;
              conference += this.purposeSumSource[i - months][4] * weight;
              study += this.purposeSumSource[i - months][5] * weight;
              exhibition += this.purposeSumSource[i - months][6] * weight;
              medical += this.purposeSumSource[i - months][7] * weight;
              others += this.purposeSumSource[i - months][8] * weight;
              unstated += this.purposeSumSource[i - months][9] * weight;
            }

            if (weights != 0) {
              business = Math.floor(business / weights);
              pleasure = Math.floor(pleasure / weights);
              visit = Math.floor(visit / weights);
              conference = Math.floor(conference / weights);
              study = Math.floor(study / weights);
              exhibition = Math.floor(exhibition / weights);
              medical = Math.floor(medical / weights);
              others = Math.floor(others / weights);
              unstated = Math.floor(unstated / weights);
            }

            let d = [
              this.purposeSumSource[i][0],
              business,
              pleasure,
              visit,
              conference,
              study,
              exhibition,
              medical,
              others,
              unstated
            ];

            data.push(d);

            let row = [];
            for (let j = 1; j < 10; j++) {
              if (this.purposeSumSource[i][j] == 0) {
                row.push(1);
              } else {
                row.push(
                  Math.pow(
                    (d[j] - this.purposeSumSource[i][j]) /
                      this.purposeSumSource[i][j],
                    2
                  ).toFixed(2)
                );
              }
            }
            t.push(row);
          }
        }
      }

      // Calculate RMSE
      let devations = [0, 0, 0, 0, 0, 0, 0, 0, 0];
      for (let row of t) {
        for (let i in row) {
          devations[i] += parseFloat(row[i]);
        }
      }

      for (let i in devations) {
        devations[i] = devations[i] / t.length;
        devations[i] = Math.sqrt(devations[i]).toFixed(2);
      }

      if (!isHistory) {
        this.addNewHistory(devations);
      }

      // Filter Selected Purposes
      let indexs = [];
      for (let purpose of this.purposeList) {
        if (this.selectedPurposes.indexOf(purpose) == -1) {
          indexs.push(this.purposeList.indexOf(purpose));
        }
      }
      indexs.sort();
      indexs.reverse();

      this.chartData3 = [];
      let cloneSumSource = JSON.parse(JSON.stringify(data));
      for (let s of cloneSumSource) {
        for (let i of indexs) {
          s.splice(i + 1, 1);
        }

        this.chartData3.push(s);
      }

      // Add Predict Data To The Chart
      let addtionalColumns = [];
      for (let i in this.chartData3[0]) {
        if (i != 0) {
          addtionalColumns.push(this.chartData3[0][i] + " Predict");
        }
      }
      for (let i in this.chartData1) {
        if (i == 0) {
          for (let a of addtionalColumns) {
            this.chartData1[0].push(a);
          }
        } else {
          for (let j in this.chartData3[i]) {
            if (j != 0) {
              this.chartData1[i].push(this.chartData3[i][j]);
            }
          }
        }
      }

      // Draw Summarize Chart
      this.chartData2 = [];
      this.chartData2.push(["Period", "Summarize", "Summarize Predict"]);
      for (let i in this.chartData1) {
        if (i != 0) {
          let row = [];
          let sum = 0;
          let sumPredict = 0;
          for (let j = 1; j <= addtionalColumns.length; j++) {
            if (j != 0) {
              sum += parseInt(this.chartData1[i][j]);
              sumPredict += parseInt(
                this.chartData1[i][j + addtionalColumns.length]
              );
            }
          }
          row.push(this.chartData1[i][0]);
          row.push(sum);
          row.push(sumPredict);
          this.chartData2.push(row);
        }
      }
    },
    onRunHistory(index) {
      this.rules = this.history[index].rules;
      this.onRun(true);
    },
    onSelectedPurposesChange() {
      // this.drawSummarizes();
      this.onRun(true);
    },
    caculateAvg(index) {
      let sum = 0;
      for (let devation of this.history[index].devations) {
        sum += parseFloat(devation);
      }
      return (sum / this.history[index].devations.length).toFixed(2);
    }
  },
  created() {
    let residences = [];
    let periods = [];
    for (let item of this.source) {
      if (!residences.includes(item.Residence)) {
        residences.push(item.Residence);
      }

      if (!periods.includes(item.Period)) {
        periods.push(item.Period);
      }
    }

    let items = [];
    for (let period of periods) {
      let obj = {};
      obj["Period"] = period;
      for (let purpose of this.purposeList) {
        obj[purpose] = 0;
      }

      for (let item of this.source) {
        if (item.Period == period) {
          for (let purpose of this.purposeList) {
            obj[purpose] += parseInt(item[purpose]);
          }
        }
      }

      items.push(obj);
    }

    this.purposeSumSource = [["Period"]];
    for (let purpose of this.purposeList) {
      this.purposeSumSource[0].push(purpose);
    }

    for (let item of items) {
      let temp = [];
      temp.push(item["Period"]);
      for (let purpose of this.purposeList) {
        temp.push(item[purpose]);
      }
      this.purposeSumSource.push(temp);
    }

    this.drawPurposes();
    this.drawSummarizes();
  }
};
</script>

<style>
</style>