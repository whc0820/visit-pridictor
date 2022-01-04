<template>
  <v-content class="mt-2 px-5">
    <v-row>
      <v-col class="d-flex flex-column d-sm-12" cols="12" lg="4">
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
                  <v-icon color="green">mdi-plus</v-icon>
                </v-btn>
              </template>
              <span>Add</span>
            </v-tooltip>
          </v-card-title>
          <v-card-text>
            <span
              class="mb-2"
              v-if="rules.length > 0"
              v-text="`Starts Month: ${periods[rules[rules.length - 1].months]}`"
            />
            <v-simple-table v-if="rules.length > 0" height="150" fixed-header dense>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">Months Before</th>
                    <th class="text-left">Weight</th>
                    <th class="text-left">Percentage</th>
                    <th class="text-left">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, i) in rules" :key="i">
                    <td v-text="item.months" />
                    <td v-text="item.weight" />
                    <td>
                      <v-progress-circular
                        class="hidden-xs-only"
                        size="20"
                        rotate="-90"
                        :value="item.percentage"
                        :color="getPercentageColor(item.percentage)"
                      />
                      <span class="ms-2" v-text="`${item.percentage}%`" />
                    </td>
                    <td class="d-flex flex-row justify-start">
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
                          <v-btn v-on="on" color="red" icon small @click="onRemoveRule(i)">
                            <v-icon small>mdi-close</v-icon>
                          </v-btn>
                        </template>
                        <span>Remove</span>
                      </v-tooltip>
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
            <div class="d-flex flew-row align-center" v-else>
              <span>There is no rule yet!</span>
            </div>
          </v-card-text>
          <v-card-actions v-if="rules.length > 0">
            <v-spacer />
            <v-btn color="red" text @click="rules=[]">Clear</v-btn>
            <v-btn color="primary" text @click="onRun(false)">Run</v-btn>
          </v-card-actions>
        </v-card>

        <v-card class="mt-5">
          <v-card-title>
            <span>History</span>
            <v-spacer />
            <input id="file" type="file" ref="file" style="display: none" @change="onFileChange" />
            <v-tooltip right>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" color="green" icon @click="$refs.file.click()">
                  <v-icon>mdi-upload</v-icon>
                </v-btn>
              </template>
              <span>Upload</span>
            </v-tooltip>
            <v-tooltip v-if="history.length > 0" right>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" color="green" icon @click="saveFileAsJson(history)">
                  <v-icon>mdi-download</v-icon>
                </v-btn>
              </template>
              <span>Download</span>
            </v-tooltip>
          </v-card-title>
          <v-card-text>
            <v-simple-table v-if="history.length > 0" height="150" dense fixed-header>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">Time</th>
                    <th class="text-left">Sum. RMSE</th>
                    <th class="text-left">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, i) in history" :key="i">
                    <td class="text-left" v-text="item.time" />
                    <td class="text-left" v-text="item.summarize" />
                    <td class="d-flex flex-row">
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
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
            <span v-else>There is no history yet!</span>
          </v-card-text>
          <v-card-actions>
            <template v-if="history.length > 0">
              <v-spacer />
              <v-btn color="red" text @click="history=[]">Clear</v-btn>
            </template>
            <template v-else>
              <v-spacer />
              <v-btn color="deep-purple" disabled text @click="onAutoPredict">Auto Run</v-btn>
            </template>
          </v-card-actions>
        </v-card>
      </v-col>

      <v-col class="d-flex flex-column d-sm-12" cols="12" lg="8">
        <v-card>
          <v-card-title>Visitors By Purposes</v-card-title>
          <v-card-text>
            <GChart type="LineChart" :data="chartData1" :options="chartOptions" />
          </v-card-text>
        </v-card>
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

    <v-dialog v-model="historyDialog" width="950" scrollable>
      <v-card v-if="history.length > 0">
        <v-card-title>
          <span class="primary--text" v-text="history[selectedHistoryIndex].time" />
          <v-spacer />
          <v-tooltip right>
            <template v-slot:activator="{ on }">
              <v-btn v-on="on" color="red" icon @click="historyDialog = false">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </template>
            <span>Close</span>
          </v-tooltip>
        </v-card-title>
        <v-card-text>
          <v-row>
            <v-col class="d-flex flex-column" cols="12">
              <span class="body-1">Summarized RMSE</span>
              <span class="caption" v-text="history[selectedHistoryIndex].summarize" />
            </v-col>
            <v-col cols="12">
              <span class="body-1">Purposes' RMSE</span>
              <v-simple-table class="px-2" dense>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th
                        class="text-left"
                        v-for="(item, i) in purposeList"
                        :key="i"
                        v-text="item"
                      />
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <th
                        class="text-left"
                        v-for="(item, i) in history[selectedHistoryIndex].deviations"
                        :key="i"
                        v-text="item"
                      />
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-col>
            <v-col cols="12" lg="6" md="8" sm="10">
              <span class="body-1">WMA Rules</span>
              <v-simple-table class="px-2" dense fixed-header>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th class="text-left">Months Before</th>
                      <th class="text-left">Weight</th>
                      <th class="text-left">Percentage</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in history[selectedHistoryIndex].rules" :key="item.months">
                      <td v-text="item.months" />
                      <td v-text="item.weight" />
                      <td>
                        <v-progress-circular
                          class="hidden-xs-only"
                          size="20"
                          rotate="-90"
                          :value="item.percentage"
                          :color="getPercentageColor(item.percentage)"
                        />
                        <span class="ms-2" v-text="`${item.percentage}%`" />
                      </td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-snackbar v-model="snackbar" :color="snackbarColor" bottom right>
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
      periods: [],
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
      history: [],
      array: "no data yet!"
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
    calculateRulesPercentage() {
      let weights = 0;
      for (let rule of this.rules) {
        weights += parseInt(rule.weight);
      }

      if (weights == 0) {
        for (let rule of this.rules) {
          rule.percentage = (100 / this.rules.length).toFixed(1);
        }
      } else {
        for (let rule of this.rules) {
          rule.percentage = ((rule.weight / weights) * 100).toFixed(1);
        }
      }
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

      this.calculateRulesPercentage();
      this.rules.sort((a, b) => {
        return a.months - b.months;
      });

      this.isEditingRule = false;
      this.ruleDialog = false;
    },
    onRemoveRule(index) {
      this.rules.splice(index, 1);
      this.calculateRulesPercentage();
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
    addNewHistory(predicts, deviations) {
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

      let maxMonths = 0;
      for (let rule of this.rules) {
        if (rule.months > maxMonths) {
          maxMonths = rule.months;
        }
      }

      let source = [];
      for (let i = 0; i < this.purposeSumSource.length; i++) {
        if (i > maxMonths) {
          let s = 0;
          for (let j in this.purposeSumSource[i]) {
            if (j > 0) {
              s += parseInt(this.purposeSumSource[i][j]);
            }
          }
          source.push(s);
        }
      }

      let predict = [];
      for (let i in predicts) {
        if (i > 0) {
          let s = 0;
          for (let j in predicts[i]) {
            if (j > 0) {
              s += parseFloat(predicts[i][j]);
            }
          }
          predict.push(s);
        }
      }

      let sum = 0;
      for (let i in source) {
        sum += Math.pow(predict[i] - source[i], 2);
      }

      let sumRMSE = Math.sqrt(sum / source.length).toFixed(2);

      let h = {
        time: time,
        rules: JSON.parse(JSON.stringify(this.rules)),
        summarize: sumRMSE,
        deviations: deviations
      };
      // this.history.splice(0, 0, h);
      this.history.push(h);
      this.history.sort((a, b) => {
        return a.summarize - b.summarize;
      });
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
              // if (this.purposeSumSource[i][j] == 0) {
              //   row.push(1);
              // } else {
              row.push(
                Math.pow(d[j] - this.purposeSumSource[i][j], 2).toFixed(2)
              );
              // }
            }
            t.push(row);
          }
        }
      }

      // Calculate RMSE
      let deviations = [0, 0, 0, 0, 0, 0, 0, 0, 0];
      for (let row of t) {
        for (let i in row) {
          deviations[i] += parseFloat(row[i]);
        }
      }

      for (let i in deviations) {
        deviations[i] = deviations[i] / t.length;
        deviations[i] = Math.sqrt(deviations[i]).toFixed(2);
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

      if (!isHistory) {
        this.addNewHistory(data, deviations);
      }
    },
    onRunHistory(index) {
      this.rules = this.history[index].rules;
      this.onRun(true);
    },
    onSelectedPurposesChange() {
      this.onRun(true);
    },
    onFileChange(event) {
      let f = event.target.files[0];
      let fr = new FileReader();
      fr.onload = () => {
        let uploadJson = JSON.parse(fr.result);

        let formatKey = ["time", "rules", "summarize", "deviations"];
        let isValid = true;
        for (let rule of uploadJson) {
          let keys = Object.keys(rule);
          for (let i in keys) {
            if (keys[i] != formatKey[i]) {
              isValid = false;
              break;
            }
          }
        }

        if (isValid) {
          this.history = uploadJson;
        } else {
          this.showSnackbar(
            "mdi-alert",
            "error",
            "Failed To Read Upload File!"
          );
        }
        document.getElementById("file").value = "";
      };
      fr.readAsText(f);
    },
    saveFileAsJson(obj) {
      const data = JSON.stringify(obj);
      const blob = new Blob([data], { type: "text/plain" });
      const e = document.createEvent("MouseEvents");
      const a = document.createElement("a");
      a.download = "history.json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ["text/json", a.download, a.href].join(":");
      e.initEvent(
        "click",
        true,
        false,
        window,
        0,
        0,
        0,
        0,
        0,
        false,
        false,
        false,
        false,
        0,
        null
      );
      a.dispatchEvent(e);
    },
    getPercentageColor(percentage) {
      if (percentage < 25) {
        return "green";
      } else if (percentage < 50) {
        return "orange";
      } else if (percentage < 75) {
        return "red";
      } else {
        return "deep-purple";
      }
    },
    generateAllCases() {
      let array = [];
      for (let a = 0; a <= 5; a++) {
        for (let b = 0; b <= 5; b++) {
          for (let c = 0; c <= 5; c++) {
            // for (let d = 0; d <= 10; d++) {
            //   for (let e = 0; e <= 10; e++) {
            //     for (let f = 0; f <= 10; f++) {
            let weights = a + b + c;
            if (weights == 0) {
              array.push([
                {
                  months: 1,
                  weight: a,
                  percentage: ((weights / 3) * 100).toFixed(1)
                },
                {
                  months: 2,
                  weight: b,
                  percentage: ((weights / 3) * 100).toFixed(1)
                },
                {
                  months: 3,
                  weight: c,
                  percentage: ((weights / 3) * 100).toFixed(1)
                }
                // {
                //   months: 4,
                //   weight: d,
                //   percentage: (weights / 4).toFixed(1)
                // },
                // {
                //   months: 5,
                //   weight: e,
                //   percentage: (weights / 6).toFixed(1)
                // },
                // {
                //   months: 6,
                //   weight: f,
                //   percentage: (weights / 6).toFixed(1)
                // }
              ]);
            } else {
              array.push([
                {
                  months: 1,
                  weight: a,
                  percentage: ((a / weights) * 100).toFixed(1)
                },
                {
                  months: 2,
                  weight: b,
                  percentage: ((b / weights) * 100).toFixed(1)
                },
                {
                  months: 3,
                  weight: c,
                  percentage: ((c / weights) * 100).toFixed(1)
                }
                // {
                //   months: 4,
                //   weight: d,
                //   percentage: ((d / weights) * 100).toFixed(1)
                // },
                // {
                //   months: 5,
                //   weight: e,
                //   percentage: ((e / weights) * 100).toFixed(1)
                // },
                // {
                //   months: 6,
                //   weight: f,
                //   percentage: ((f / weights) * 100).toFixed(1)
                // }
              ]);
              //     }
              //   }
              // }
            }
          }
        }
      }
      return array;
    },
    async onAutoPredict() {
      this.array = await this.generateAllCases();
      this.recursive(0);
    },
    async recursive(i) {
      this.rules = this.array[i];
      await this.onRun(false);

      if (i != this.array.length) {
        this.recursive(i + 1);
      }
    }
  },
  created() {
    let residences = [];
    this.periods = [];
    for (let item of this.source) {
      if (!residences.includes(item.Residence)) {
        residences.push(item.Residence);
      }

      if (!this.periods.includes(item.Period)) {
        this.periods.push(item.Period);
      }
    }

    let items = [];
    for (let period of this.periods) {
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