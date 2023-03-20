<template>
  <span>
    <!-- As a heading -->
    <nav class="navbar navbar-light bg-light">
      <div class="container-fluid">
        <!-- <span class="navbar-brand mb-0 h3">钢卷切割方案</span> -->
        <h3 class="my-0">钢卷切割方案</h3>
      </div>
    </nav>

    <div class="container text-start">
      <!-- header / intro section -->
      <div class="row">
        <div class="col">
          <ul class="nav nav-tabs">
            <li class="nav-item">
              <!-- hide the border around button onclick, using onmousedown -->
              <button class="nav-link active" id="1d-tab" v-on:click="setMode('1d')">
                切条
              </button>
            </li>

            <!-- <li class="nav-item">
              <button class="nav-link" id="2d-tab" v-on:click="setMode('2d')">
                切块
              </button>
            </li> -->
          </ul>
        </div>
      </div>

      <div class="row tab-content">
        <!-- input sections | left -->
        <div class="col-6">
          <div class="row mt-4">
            <div class="col-3">
              <input type="radio" id="exactCutsRadio" name="exactCutsRadio" class="form-check-input" value="weightCuts"
                v-model="cutStyle" />
              <label class="custom-control-label" for="exactCutsRadio">
                按重量切割
                <span class="information" data-toggle="tooltip" data-placement="top" title="根据客户预定的重量切割">?</span>
              </label>
            </div>
            <div class="col-3">
              <input type="radio" id="minWasteRadio" name="minWasteRadio" class="form-check-input" value="ruleCuts"
                v-model="cutStyle" />

              <label class="custom-control-label" for="minWasteRadio">
                按规格切割
                <span class="information" data-toggle="tooltip" data-placement="top" title="根据客户的规格切割">?</span>
              </label>
            </div>
          </div>
          <!-- input small rects | left+top -->
          <div class="col" v-if="cutStyle === 'ruleCuts'">
            <!-- <div class="col tab-content"> -->
            <h3 class="mt-2">待选方案</h3>
            <div class="row d-lg-flex mx-0 pt-lg-1">
              <div class="col-12 col-md-6 col-lg-4 py-1 px-2 "
                v-for="(child_rolls, index) in mode_data.childs_for_select" v-bind:key="index">
                <table cellpadding="0" cellspacing="0" class="w-90 border-1" >
                  <thead>
                    <tr class="border">
                      <td class="px-1">#</td>
                      <td class="px-1" width="50%">宽度(mm)</td>
                      <td class="px-1">数量</td>
                      <td class="px-1 border-0">
                        <div v-on:click="selectdRow(index)" class="btn btn-outline-danger btn-sm m-0 py-0 px-1">
                          ✔️
                        </div>
                      </td>
                    </tr>
                  </thead>
                  <thead>
                    <tr class="border">
                      <td class="px-1">#</td>
                      <td class="px-1"> 损耗(mm)</td>
                      <td class="px-1">
                        {{
                        child_rolls.worstWidth
                        }}
                      </td>
                      <td class="px-1 border-0"></td>
                    </tr>
                  </thead>
                  <tbody>
                    <tr class="border" v-for="(child, i) in child_rolls.sub_child_solver" v-bind:key="i">
                      <td class="px-1 text-secondary">
                        {{ i + 1 }}
                      </td>
                      <td>
                        <input class="px-1" type="text" v-model="child.width" />
                      </td>
                      <td>
                        <input class="px-1" type="text" v-model="child.quantity" />
                      </td>
                      <td class="px-1 border-0">
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
          <!-- input small rects | left+top -->
          <div class="row">
            <div class="col">
              <h3 class="mt-2">{{ mode_data.childTitle }}</h3>
              <p class="text-secondary mt-0 mb-2">
                {{ mode_data.childMessage }}
              </p>

              <div class="row">
                <div class="col">
                  <!-- adds a row to Sheets to Cut -->
                  <button class="my-1 btn btn-outline-success btn-sm float-start" v-on:click="addRowToChilds">
                    <b>+ 增加</b>
                  </button>
                </div>

                <div class="col">
                  <button class="my-1 btn btn-outline-danger btn-sm float-end" v-on:click="clearChildData">
                    <b>x 清除</b>
                  </button>
                </div>
              </div>

              <p class="text-danger mb-1">{{ mode_data.childErrors }}</p>

              <table cellpadding="0" cellspacing="0" class="w-100 border-0">
                <thead>
                  <tr class="border">
                    <td class="px-1">#</td>
                    <td class="px-1">宽</td>
                    <!-- display height if mode is 2d -->
                    <td v-if="mode === '2d'" class="px-1">高</td>
                    <td class="px-1">数量</td>
                    <td v-if="cutStyle === 'weightCuts'" class="px-1">预定重量（吨）</td>
                    <!-- <td>Color</td> -->
                    <td class="px-1">重量（吨）</td>
                    <td v-if="mode === '1d' && mode_data.result" class="px-3"></td>
                    <td class="px-1 border-0"></td>
                  </tr>
                </thead>
                <tbody>
                  <tr class="border" v-for="(child, index) in mode_data.childs" v-bind:key="index">
                    <td class="px-1 text-secondary">
                      {{ index + 1 }}
                    </td>
                    <td>
                      <input class="px-1" type="text" v-model="child.width" />
                    </td>

                    <td v-if="mode === '2d'">
                      <input class="px-1" type="text" v-model="child.height" />
                    </td>
                    <td>
                      <input class="px-1" type="text" v-model="child.quantity" />
                    </td>
                    <td v-if="cutStyle === 'weightCuts'">
                      <input class="px-1" type="text" v-model="child.sch_quantity" />
                    </td>
                    <td>
                      <input disabled="true" class="px-1" type="text" v-model="child.weight" />
                    </td>
                    <td v-if="mode === '1d' && mode_data.result" class="px-1" :style="getColor(child.width)"></td>

                    <td class="px-1 border-0">
                      <div v-on:click="removeRow(index, false)" class="btn btn-outline-danger btn-sm m-0 py-0 px-1">
                        x
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <hr class="mb-2" />

          <!-- input sheets | left+bottom -->
          <div class="row">
            <div class="col">
              <h3 class="my-0">{{ mode_data.parentTitle }}</h3>
              <p class="text-secondary mb-2">{{ mode_data.parentMessage }}</p>

              <div class="row">
                <div class="col">
                  <button :disabled="cutButtonDisabled" class="my-2 btn btn-primary btn-sm" @click="calSheets()">
                    <b>✓ 计算数量</b>
                  </button>
                </div>
                <div class="col">
                  <button class="my-1 btn btn-outline-success btn-sm float-start" v-on:click="addRowToParents">
                    <b>+ 增加</b>
                  </button>
                </div>
                <div class="col">
                  <button class="my-1 btn btn-outline-danger btn-sm float-end" v-on:click="clearParentData">
                    <b>x 清除</b>
                  </button>
                </div>
              </div>

              <p class="text-danger mb-1">{{ mode_data.parentErrors }}</p>

              <table cellpadding="0" cellspacing="0" class="w-100 border-0">
                <thead>
                  <tr class="border">
                    <td class="px-1">#</td>
                    <td class="px-1">卷宽度（mm）</td>
                    <td class="px-1">卷数量</td>
                    <td class="px-1">卷重量（吨）</td>
                    <td class="px-1 border-0">
                    </td>
                  </tr>
                </thead>
                <tbody>
                  <tr class="border" v-for="(parent, index) in mode_data.parents" v-bind:key="index">
                    <td class="px-1 text-secondary">
                      #
                    </td>

                    <td v-if="mode === '2d'">
                      <input class="px-1" type="text" v-model="parent.height" />
                    </td>

                    <td>
                      <input class="px-1" type="text" v-model="parent.width" v-bind:on-focusout="getSideWeight()" />
                    </td>
                    <td>
                      <input class="px-1" type="text" v-model="parent.quantity" />
                    </td>
                    <td>
                      <input class="px-1" type="text" v-model="parent.weight" />
                    </td>
                    <td class="px-1 border-0">
                      <div v-on:click="removeRow(index, true)" class="btn btn-outline-danger btn-sm m-0 py-0 px-1">
                        x
                      </div>
                    </td>
                  </tr>
                </tbody>
                <thead>
                  <tr class="border">
                    <td class="px-1">#</td>
                    <td class="px-1">总重量（吨）</td>
                    <td class="px-1">
                      <input type="text" v-model="this.all_weight" v-bind:on-focusout="getSideWeight()" />
                    </td>
                    <td class="px-1">

                    </td>
                    <td class="px-1"></td>
                  </tr>
                </thead>
                <thead>
                  <tr class="border">
                    <td class="px-1">#</td>
                    <td class="px-1">边丝（mm）</td>
                    <td class="px-1">
                      <input type="text" v-model="this.side" v-bind:on-focusout="getSideWeight()" />
                    </td>
                    <td class="px-1">
                      边丝重量
                      <input disabled="true" type="text" style="width:30%" v-model="this.side_worst" />（吨）
                    </td>
                    <td class="px-1"></td>
                  </tr>
                </thead>

              </table>
            </div>
          </div>
        </div>

        <!-- output+diagram section | right -->
        <div class="col-6">
          <div class="row">
            <div class="col">
              <button :disabled="cutButtonDisabled" class="my-2 btn btn-primary btn-sm" @click="cutSheets()">
                <b>✓ 计算</b>
              </button>
            </div>

            <div class="col">
              <button :disabled="cutButtonDisabled" class="my-2 btn btn-outline-danger btn-sm float-end"
                @click="reset()">
                <b>x 重置</b>
              </button>
            </div>
          </div>

          <h4 v-if="mode_data.result" class="text-capitalize">
            方案图示: {{ mode_data.result.statusName }}
          </h4>

          <div id="d3_area">
            <svg class="w-100"></svg>
          </div>

          <span v-if="mode_data.result">
            <div v-if="mode === '1d'" class="row my-2">
              <div class="col">
                <h4>切割方案</h4>
                <div class="row">
                  <div class="col">
                    <p class="m-0">
                      共需要卷的数量 = {{ mode_data.result.solutions.length }}
                    </p>
                  </div>

                  <div class="col">
                    <button v-on:click="downloadCsv()" class="p-0 btn btn-link float-end">
                      下载
                    </button>
                  </div>
                </div>

                <table cellpadding="0" cellspacing="0" class="w-100 border-0">
                  <thead>
                    <tr class="border">
                      <td class="px-1">卷</td>
                      <td class="px-1">使用率</td>
                      <td class="px-1">边丝/重量</td>
                      <td class="px-1">切割明细</td>
                    </tr>
                  </thead>
                  <tbody>
                    <tr class="border" v-for="(bigRoll, index) in mode_data.result.solutions" v-bind:key="index">
                      <td class="px-1 text-secondary">
                        {{ index + 1 }}
                      </td>

                      <td class="px-1">
                        {{ getPercentageUtilization(bigRoll[0], bigRoll[1]) }} %
                      </td>
                      <td class="px-1">
                        {{ getRound(bigRoll[0]) }}/{{ bigRoll[2]}}
                      </td>
                      <td class="px-1">
                        <!-- join without space, so paste in excel would not change it to date -->
                        {{ bigRoll[1].join(",") }}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div v-if="mode === '1d'" class="row my-2">
              <div class="col">
                <h4>切割方案汇总</h4>
                <div class="row">
                  <div class="col">
                    <p class="m-0">
                      共需要卷的数量 = {{ mode_data.result.solutions.length }}
                    </p>
                  </div>
                </div>

                <table cellpadding="0" cellspacing="0" class="w-100 border-0">
                  <thead>
                    <tr class="border">
                      <td class="px-1">卷</td>
                      <td class="px-1">使用率</td>
                      <td class="px-1">边丝/重量</td>
                      <td class="px-1">切割明细及重量</td>
                    </tr>
                  </thead>
                  <tbody>
                    <tr class="border" v-for="(bigRoll, index) in mode_data.result.solutions" v-bind:key="index">
                      <td class="px-1 text-secondary">
                        {{ index + 1 }}
                      </td>

                      <td class="px-1">
                        {{ getPercentageUtilization(bigRoll[0], bigRoll[1]) }} %
                      </td>
                      <td class="px-1">
                        {{ getRound(bigRoll[0]) }}/{{ bigRoll[2]}}
                      </td>
                      <td class="px-1">
                        <!-- join without space, so paste in excel would not change it to date -->
                        {{
                            bigRoll[3].join(",  ")
                        }}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <!-- dimentions of rects of solutions -->
            <div v-if="mode === '2d'">
              <div v-for="(sol, index) in mode_data.result.solutions" v-bind:key="index">
                <div v-for="(rect, index) in sol" v-bind:key="index">
                  {{ rect[0] }},{{ rect[1] }}-{{ rect[2] }},{{ rect[3] }}
                </div>
                <hr />
              </div>
            </div>
          </span>
        </div>
      </div>

      <hr />

      <footer class="footer mb-3">© {{ getCurrentYear() }}</footer>
    </div>
  </span>
</template>

<script>
import * as d3 from "d3";
import * as axios from "axios";
export default {
  name: "CspTool",
  props: {
    msg: String,
  },
  components: {},
  data: function () {
    return {
      // by default, 1d mode will open. But onclick 1d / 2d buttons, mode can be switched
      mode: "1d",
      message: "",
      cutStyle: "weightCuts",
      side: 3,
      side_worst: 0.0,
      all_weight: 0.0,
      // remembers the state of Cut button
      cutButtonDisabled: false,
      rule_selectd_index:-1,
      mode1d: {
        childs: [
          { width: "", quantity: "", weight: 0, sch_quantity: 0 }, // 1d mode doesn't have height
        ],
        parents: [{ width: "", quantity: "", weight: 0 }],
        childErrors: null,
        parentErrors: null,
        // data from server, to display
        result: null,

        // information displayed on each mode
        childTitle: "需要切割的规格（mm）",
        childMessage:
          "添加切割的长度和数量",
        parentTitle: "卷的长度（mm）",
        parentMessage:
          "输入卷的长度和毛边的宽度单位mm",
      },

      mode2d: {
        childs: [{ width: "", height: "", quantity: "" }],
        parents: [{ width: "", height: "", quantity: "Comming soon" }],
        childErrors: null,
        parentErrors: null,
        // data from server, to display
        result: null,

        // information displayed on each mode
        childTitle: "配置每块的长宽（mm）",
        childMessage:
          "目前设置方形",
        parentTitle: "配置卷",
        parentMessage: "目前只设置一个卷",
      },

      // currently displayed mode's data (mode1d or mode2d from above)
      mode_data: null,

      // from https://flatuicolors.com/palette/defo
      colors: [
        "#f1c40f", // Sun Flower
        "#1abc9c", // Torquise
        "#f39c12", // Orange
        "#2ecc71", // Emerald
        "#27ae60", // Nephritis
        "#e67e22", // Carrot
        "#d35400", // Pumpkin
        "#16a085", // Green Sea
        "#3498db", // Peter River
        "#2980b9", // Belize Hole
        "#e74c3c", // Alizarin
        "#c0392b", // Pomegranate
        "#9b59b6", // Amethyst
        "#8e44ad", // Wisteria
        "#ecf0f1", // Clouds
        '#bdc3c7', // Silver
        '#95a5a6', // Concrete <- Clouds & Silver are close
        '#34495e', // West Asphalt <- don't use because it is very close to Midnight blue
        '#2c3e50', // Midnight Blue <- use for wasted part
      ],

      wasteColor: "#7f8c8d", // Asbestos
    };
  },
  beforeMount() {
    // console.log('#beforeMount');

    // set mode before mount
    this.setMode("1d");
  },

  mounted() {
    this.hideButtonClickBorder();
  },

  methods: {
    /**
     * bootstrap buttons stay highlighted with a border after a click
     * this method hides that
     */
    hideButtonClickBorder: function () {
      let buttons = document.getElementsByTagName("button");
      for (let i = 0; i < buttons.length; i++) {
        buttons[i].addEventListener("mousedown", function (event) {
          return event.preventDefault();
        });
      }
    },

    /**
     * this method is called when the mode is switched between 1d and 2d
     * this method saves the data of currently displayed mode and displays the data
     * that was previously entered in the other mode because that mode just become the current.
     *
     * it also draws the results of that mode, if they were retrieved
     */
    setMode: function (newMode) {
      this.mode = newMode;

      if (newMode === "1d") {
        // hide 2d mode's options
        if (this.mode_data != null) this.mode2d = this.mode_data;

        this.mode_data = this.mode1d;

        // make 1D tab active
        this.removeActiveClass("2d-tab");
        this.addActiveClass("1d-tab");

        // if there is anything to draw, draw it
        this.draw1d();
      } else if (newMode === "2d") {
        // hide 1d mode's options
        if (this.mode_data != null) this.mode1d = this.mode_data;

        this.mode_data = this.mode2d;

        this.removeActiveClass("1d-tab");
        this.addActiveClass("2d-tab");

        this.draw2d();
      }

      console.log("Mode: ", this.mode);
      console.log("Mode_data: ", this.mode_data);
    },

    /**
        add active Bootstrap class to make the Tab look as selected
        */
    addActiveClass: function (id) {
      let element = document.getElementById(id);
      if (!element) return;

      element.classList.add("active");
    },

    removeActiveClass: function (id) {
      let element = document.getElementById(id);
      if (!element) return;

      element.classList.remove("active");
    },

    /**
     * checks if the number entered at current row & column is a valid digit
     * since input field has type text (for styling reason), this function enforces
     * that input is positive number
     */
    validNum: function (row, key, is_parent) {
      // is this from parent table?
      let item = null;
      if (is_parent) item = this.mode_data.parents[row][key];
      else item = this.mode_data.childs[row][key];

      let validChars = "";
      for (let i = 0; i < item.length; i++) {
        const c = item[i];

        if ("0123456789".includes(c)) {
          // c is a digit? in string type
          validChars += c;
        }
      }

      const num = validChars === "" ? 0 : parseInt(validChars);

      if (is_parent) this.mode_data.parents[row][key] = num;
      else this.mode_data.childs[row][key] = num;
    },

    addRowToChilds: function () {
      this.mode_data.childs.push(["", "0", "0", "0"]); // add an empty row
    },

    addRowToParents: function () {
      this.mode_data.parents.push(["", "", "0"]); // add an empty row
    },

    hideResult: function () {
      this.mode_data.result = null;
    },

    calSheets: function () {
      this.rule_selectd_index = -1;
      this.getSideWeight();
      // 根据重量切割
      if (this.cutStyle == "weightCuts") {
        this.side_worst = 3;
        let parent_width = 0.0;

        for (let index = 0; index < this.mode_data.parents.length; index++) {
          const element = this.mode_data.parents[index];
          if (element.weight == null || element.weight == 0) {
            alert("请输入母卷的重量，单位（吨）！");
            return false;
          }
          if (element.width == null || element.width == 0) {
            alert("请输入母卷的长度，单位（mm）！");
            return false;
          }
          if (element.quantity == null || element.quantity == 0) {
            alert("请输入母卷的数量！");
            return false;
          }
        }
        for (let index = 0; index < this.mode_data.childs.length; index++) {
          const element = this.mode_data.childs[index];
          if (element.sch_quantity == null || element.sch_quantity == 0.0) {
            alert("请输入子卷预定的重量，单位（吨）！");
            return false;
          }
          if (element.width == null || element.width == 0) {
            alert("请输入子卷的宽度，单位（mm）！");
            return false;
          }
        }
        this.mode_data.parents.forEach((parent) => {
          parent_width += parseFloat(parent.width)*parseFloat(parent.quantity);
        });
        for (let i = 0; i < this.mode_data.childs.length; i++) {
          const child = this.mode_data.childs[i];
          let s = (parseFloat(child.width) / parseFloat(parent_width)) * parseFloat(this.all_weight);
          let n = Math.round(parseFloat(child.sch_quantity) / s);
          child.quantity = n;
        }
      }
      // 根据规格切割
      if (this.cutStyle == "ruleCuts") {
        let url = 'http://150.158.76.64:9999/api/stocks_1d_by_len';
        let s_weight = 0.0;
        this.mode_data.parents.forEach((parent) => {
          s_weight += parseFloat(parent.weight) * parseFloat(parent.quantity);
        });
        this.all_weight = s_weight;
        // this.mode_data.childs_for_select
        const dataToSend = this.prepareDataToSend1DForRule();
        console.log("dataToSend", dataToSend);
        axios
          .post(url, dataToSend)
          .then((response) => {
            this.disableCutButton(false);
            if (response.data.data.solutions.length == 0) {
              alert("请修改子卷宽度，计算后无满足的方案！");
              return false;
            }
            console.log("response:", response);
            // let selectd_rule_list = [];
            response.data.data.solutions.forEach((sol) => {
              if (sol.sub_child_solver.length > 0) {
                sol.sub_child_solver.forEach((rule) => {
                  rule.width = rule.width/1000;
                })
              }
              // 计算浪费宽度
              let worstWidth = 0;
              sol.solutions.solutions.forEach((sud_sol) => {
                  worstWidth += parseFloat(sud_sol.un_used);
              });
              sol.worstWidth =  (worstWidth/1000) + this.side;
            });
            this.mode_data.childs_for_select = response.data.data.solutions;
            console.log(this.mode_data.childs_for_select);
          })
          .catch((error) => {
            this.disableCutButton(false);
            console.log("Network/Server error");
            console.error(error);
          });
      }
    },
    /**
     * called when the Cut button is pressed.
     * Validates the inputs, show errors
     * Send request to the server if input is valid
     */
    cutSheets: function () {
      console.log(`Cut Style: ${this.cutStyle}`);

      // hide the result from previous request
      this.hideResult();
      // clear the drawing
      this.clearTheDrawing();

      // TODO: disable the cut button & remember which mode the button is disabled for

      const isValid = this.validate();

      if (!isValid) {
        console.log("NOT Valid");
        return;
      }

      console.log("request is valid");

      // TODO send to server
      this.sendReq2();
    },
    /**
     * Hides the previous error msgs if any,
     * validates childs and parents array
     * in parents array, only validates Width and Height
     */
    validate: function () {
      // empties the previous errors
      this.hideErrorMsgs();

      /*
            all the rows in childs must contain numbers > 0
            */
      const labels =
        this.mode === "2d"
          ? ["width", "height", "quantity"] // for 2d mode, height is required
          : ["width", "quantity"]; // for 1d mode, height is not required

      // compute child errors
      for (let i = 0; i < this.mode_data.childs.length; i++) {
        const child = this.mode_data.childs[i];

        console.log("Validating: ", child);

        for (let j = 0; j < labels.length; j++) {
          let val = child[labels[j]]; /// width, height, quantity
          val = parseInt(val);

          if (!Number.isInteger(val) || val < 0) {
            this.mode_data.childErrors = `> 行 #${i + 1}: "${labels[j]
              }" 必须填写\n`;
            // return on first error. To only show the one (first) error at a time
            return false;
          }
        }
      }

      // compute parent errors
      for (let i = 0; i < this.mode_data.parents.length; i++) {
        const parent = this.mode_data.parents[i];

        // go over width, height, quantity of each row
        // [FOR NOW: don't go over quantity. Because we don't support quantity for parent sheets]
        for (let j = 0; j < labels.length - 1; j++) {
          let val = parent[labels[j]]; // value is currently a string
          val = parseInt(val);
          if (!Number.isInteger(val) || val < 1) {
            this.mode_data.parentErrors = `> 行 #${i + 1}: "${labels[j]
              }" 必须填写\n`;

            console.log("Valdiation Error: ", this.mode_data.parentErrors);
            console.log("Attr name: ", labels[j]);
            console.log("Value: ", val);
            console.log("Row #: ", j);
            // return on first error. To only show the one (first) error at a time
            return false;
          }
        }
      }

      return !this.mode_data.childErrors && !this.mode_data.parentErrors;
    },

    hideErrorMsgs: function () {
      this.mode_data.childErrors = null;
      this.mode_data.parentErrors = null;
    },

    prepareDataToSend1D: function () {

      let newChilds = [];

      this.mode_data.childs.forEach((child) => {
        newChilds.push([parseFloat(child.quantity), parseFloat(child.width)]);
      });

      let newParents = [];
      this.mode_data.parents.forEach((parent) => {
        newParents.push([parseFloat(parent.quantity), parseFloat(parent.width)]);
      });

      return {
        child_rolls: newChilds,
        parent_rolls: newParents,
        side: parseFloat(this.side),
        cutStyle: this.cutStyle, // exactCuts or minWaste
      };
    },

    sendReq: function () {
      let url = 'http://150.158.76.64:5555/stocks_1d';
      this.disableCutButton(true);
      // process the data as server's requirements
      const dataToSend =
        this.mode === "1d"
          ? this.prepareDataToSend1D()
          : this.prepareDataToSend2D();
      console.log("dataToSend", dataToSend);
      axios
        .post(url, dataToSend)
        .then((response) => {
          console.log(response);

          this.disableCutButton(false);
          this.displayResult(response);
        })
        .catch((error) => {
          this.disableCutButton(false);
          console.log("Network/Server error");
          console.error(error);
        });

      // TODO catch exceptions and show display errors
    },

    prepareDataToSend1DForWeight: function () {
      let newChilds = [];
      this.mode_data.childs.forEach((child) => {
        newChilds.push({ "quantity": parseInt(child.quantity), "width": parseInt(parseFloat(child.width) * 1000) });
      });

      let newParents = [];
      this.mode_data.parents.forEach((parent) => {
        let worst_weight = Math.round((parseFloat(this.side)/parseFloat(parent.width))*(parseFloat(parent.weight)*1000));
        newParents.push({ "quantity": parseInt(parent.quantity), "width": parseInt(parseFloat(parent.width) * 1000) - parseInt(parseFloat(this.side) * 1000), "weight": parseInt(parseFloat(parent.weight) * 1000-worst_weight) });
      });

      return {
        child_rolls: newChilds,
        parent_rolls: newParents,
        side: 0,
        out_side: parseInt(parseFloat(this.side) * 1000),
        seed: Math.round(Math.random() * 10)
      };
    },
    prepareDataToSend1DForRule: function () {
      let newChilds = [];
      this.mode_data.childs.forEach((child) => {
        newChilds.push({ "width": parseInt(parseFloat(child.width) * 1000) });
      });

      let newParents = [];
      this.mode_data.parents.forEach((parent) => {
        let worst_weight = Math.round((parseFloat(this.side)/parseFloat(parent.width))*(parseFloat(parent.weight)*1000));
        newParents.push({ "quantity": parseInt(parent.quantity), "width": parseInt(parseFloat(parent.width) * 1000), "weight": parseInt(parseFloat(parent.weight) * 1000-worst_weight) });
      });

      return {
        child_rolls: newChilds,
        parent_rolls: newParents,
        side: 0,
        out_side: parseInt(parseFloat(this.side) * 1000),
        seed: Math.round(Math.random() * 10)
      };
    },
    sendReq2: function () {
      let url = 'http://150.158.76.64:9999/api/stocks_1d_by_weight';

      this.disableCutButton(true);
      // process the data as server's requirements
      const dataToSend = this.prepareDataToSend1DForWeight();
      console.log("dataToSend", dataToSend);
      axios
        .post(url, dataToSend)
        .then((response) => {
          console.log("==============================");
          console.log(response);
          this.disableCutButton(false);
          this.displayResult(response);
        })
        .catch((error) => {
          console.log(error);
          this.disableCutButton(false);
          alert("连接服务失败，请检查网络");
          return false;
        });
      // TODO catch exceptions and show display errors
    },

    // disable / enable cut button between requests
    disableCutButton: function (disabled) {
      this.cutButtonDisabled = disabled;
    },

    /**
     * called before sending a validated input to the server.
     * Adds multiple quantity sheets as multiple sheets and removes the quantity value.
     */
    prepareDataToSend2D: function () {
      let newChilds = [];
      this.mode_data.childs.forEach((child) => {
        const quantity = child.quantity;
        // const newChild = { width: child.width, height: child.height }
        // at the moment, server's acceptable format is
        // [ [width, height], [width, height], ...]
        const newChild = [parseInt(child.width), parseInt(child.height)];
        // add as many childs as there are quantities of that child
        for (let q = 0; q < quantity; q++) newChilds.push(newChild);
      });

      let newParents = [];
      this.mode_data.parents.forEach((parent) => {
        // At the moment, parent's quantity is not allowed, so keep it 1
        const quantity = 1;
        // const quantity = parent[2] // 0: width, 1: height, 2: quantity

        const newParent = [parseInt(parent.width), parseInt(parent.height)];
        // add as many childs as there are quantities of that child
        for (let q = 0; q < quantity; q++) newParents.push(newParent);
      });

      // the server expects data in this format
      return { child_rects: newChilds, parent_rects: newParents };
    },

    // response: from server
    displayResult: function (response) {
      console.log("data > ", response.data);
      if (response.data.data && response.data.data.status_name) {
        if (response.data.data.status_name == "Error") {
          alert("服务计算超出母卷的长度，请重试！");
          return false;
        }
      }

      this.mode_data.result = response.data;

      if (this.mode_data.result && this.mode_data.result.status_name) {

        this.mode_data.result.statusName =
          this.mode_data.result.status_name.toLowerCase();
      }

      if (this.mode === "1d") {
        let rolls = [];
        this.mode_data.result.data.solutions.forEach((soluton) => {
          let subs = [];
          soluton.subs.forEach((s) => {
            subs.push(parseFloat(s) / 1000);
          });
          let subs_weight = [];
          soluton.sub_weights.forEach((s) => {
            subs_weight.push(parseFloat(s) / 1000);
          });
          rolls.push([parseFloat(soluton.un_used / 1000), subs, parseFloat(soluton.un_used_weight / 1000),subs_weight]);
        });

        this.mode_data.result.solutions = rolls;
        let child_index = 0;
        this.mode_data.result.data.sub_weights.forEach((weight) => {
          this.mode_data.childs[child_index].weight = parseFloat(weight / 1000);
          child_index += 1;
        });

        this.checkValidity1D();
        this.draw1d();
      } else this.draw2d();
    },

    /**
        checks the validity of the output of 1D algo

        IMPORTANT: convert all local inputs to integers before comparing
        */
    checkValidity1D: function () {
      let isValid = true;

      // // input sheets
      // const childRolls = this.mode_data.childs;
      // const parentRolls = this.mode_data.parents;
      // for (let i = 0; i < parentRolls.length; i) {
      //   let parentWidth = parentRolls[i].width; // at the moment, there is only one parent. TODO update here
      //   parentWidth = parseFloat(parentWidth - this.side);

      //   // algorithm output from server
      //   let bigRolls = this.mode_data.result.solutions;
      //   // go over all the bigRolls and count the number of each small width
      //   let outputQuantities = {};

      //   for (let i = 0; i < bigRolls.length; i++) {
      //     const unusedWidth = bigRolls[i][0];
      //     const bRoll = bigRolls[i][1];

      //     // totalWidth will contain sum of all widths in current bRoll. Must be equal to parentWidth
      //     let totalWidth = Math.round(unusedWidth);
      //     console.log(
      //       `bRoll`,
      //       bRoll
      //     );
      //     for (let j = 0; j < bRoll.length; j++) {
      //       const smallRoll = bRoll[j];
      //       totalWidth += smallRoll;
      //       // count the number of rolls of each width. this is for check 2 below.
      //       // if this width has already been added, update the count (quantity)
      //       if (Object.prototype.hasOwnProperty.call(outputQuantities, smallRoll))
      //         outputQuantities[smallRoll] += 1;
      //       else outputQuantities[smallRoll] = 1;
      //     }

      //     if (totalWidth !== parentWidth) {
      //       console.error(
      //         `#checkValidity1D: bigRolls[${i}] totalWidth != parentWidth`,
      //         totalWidth,
      //         "!=",
      //         parentWidth
      //       );
      //       isValid = false;
      //     }
      //   }

      //   // go over all childs and count quantity of specific width
      //   let inputQuantities = {};
      //   for (let i = 0; i < childRolls.length; i++) {
      //     let width = childRolls[i].width;
      //     width = parseFloat(width);
      //     let quantity = childRolls[i].quantity;
      //     quantity = parseFloat(quantity);

      //     // if this width has already been added, update the count
      //     if (Object.prototype.hasOwnProperty.call(inputQuantities, width)) {
      //       // let alreadyAddedQuantity = inputQuantities[width];
      //       // inputQuantities[width] = alreadyAddedQuantity + quantity;

      //       inputQuantities[width] += quantity;
      //     } else {
      //       inputQuantities[width] = quantity;
      //     }
      //   }

      //   console.log("inputQuantities: ", inputQuantities);

      //   const inputWidths = Object.keys(inputQuantities);

      //   // for every width in inputQuantities, the quantity must be equal to quantity of
      //   // corresponding width in outputQuantities
      //   for (let i = 0; i < inputWidths.length; i++) {
      //     const width = inputWidths[i];

      //     const inQuantity = inputQuantities[width];
      //     const outQuantity = outputQuantities[width];

      //     if (inQuantity !== outQuantity) {
      //       isValid = false;
      //     } else {
      //       // remove this width from outputQuantities
      //       delete outputQuantities[width];
      //     }
      //   }

      //   /*
      //         Check 3: is there extra non intended width that user didn't input?

      //         by now, all the widths in outputQuanitites must have been deleted & it must be empty
      //         */
      //   const outputWidths = Object.keys(outputQuantities);
      //   if (outputWidths.length > 0) {
      //     console.error(
      //       `#checkValidity1D: unintended buggy widths in output:`,
      //       outputWidths,
      //       outputQuantities
      //     );
      //     isValid = false;
      //   }
      // }


      // if (!isValid) {
      //   alert("Alert! Results contains extra cuts/items. Use with caution");
      // }

      return isValid;
    },

    /**
        for output of 1D algo:
            1. sorts the bigRolls in ascending order of waste (unused width)
            2. sort the cuts to each bigRoll (small rolls in nested array to bigRoll) in descending order
        */
    sortBigRolls: function (bigRolls) {

      console.log("BigRoll=======");
      console.log(bigRolls);

      bigRolls = bigRolls.sort(function (a, b) {
        return a[0] - b[0];
      });
      for (let i = 0; i < bigRolls.length; i++) {
        // 计算子卷的个数和重量
        let unique_arr = Array.from(new Set(bigRolls[i][1]));
        let for_each_unique_arr = Array.from(new Set(bigRolls[i][1]));
        for_each_unique_arr.forEach((j,index_j)=>{
          let num = 0;
          let weight = 0;
          bigRolls[i][1].forEach((k,index_k)=>{
            if (j == k){
              num+=1;
              weight += bigRolls[i][3][index_k];
            }
          })
           unique_arr[index_j] = j + "*" + num + "/" + weight;
        })
        bigRolls[i][3] = unique_arr;
        // 排序
        let smallRolls = bigRolls[i][1];
        smallRolls = smallRolls.sort(function (a, b) {
          return a - b;
        });
        bigRolls[i][1] = smallRolls;
      }
      return bigRolls;
    },

    /**
        draws the shapes according to data and requirements of 1d algorithm
        */
    draw1d: function () {
      // clear old drawing
      this.clearTheDrawing();

      if (!this.mode_data.result) {
        console.log(
          `Cannot draw anything. "result" is: ${this.mode_data.result} for mode: ${this.mode}`
        );
        return;
      }

      const unSortedBigRolls = this.mode_data.result.solutions;
      console.log("bigRolls before sorting", unSortedBigRolls);

      const bigRolls = this.sortBigRolls(unSortedBigRolls);
      console.log("bigRolls after sorting", bigRolls);

      this.mode_data.result.solutions = bigRolls;

      const colorDict = this.getColorDict();

      const parentWidth = this.mode_data.parents[0].width - this.side;

      // get the current width alloted to #d3_area, it's dynamic
      const graphWidth = document.getElementById("d3_area").clientWidth;
      let xScale = d3
        .scaleLinear()
        .domain([0, parentWidth])
        .range([0, graphWidth]);

      // let yScale = d3.scaleOrdinal()
      // 	.domain(d3.range(0, dataLen))// <-num big rolls
      // 	.rangeBands(0, 300);
      let yScale = d3
        .scaleBand()
        .domain(d3.range(bigRolls.length))
        // .range([0, 20 * bigRolls.length])
        .range([0, 300]);

      // create svg element:
      let svg = d3.select("#d3_area svg");

      var margin = { top: 20, right: 20, bottom: 20, left: 20 };

      let svgWidth = 300 - margin.left - margin.right;
      let svgHeight = 300 - margin.top - margin.bottom;

      svg
        .attr("width", svgWidth + margin.left + margin.right)
        .attr("height", svgHeight + margin.top + margin.bottom)
        .style("background-color", "#f3f4f7")
        .style("border", "1px solid #f3f4f7")
      // .style("border-left-width", "thick")
      // .style("border-right-width", "thick");
      // .append("g")
      // .attr("transform", "translate(" + margin.left + "," + margin.top + ")")
      // .attr('fill', 'blue');

      let x1 = 0;
      let x2 = 0;
      let y1 = 0;
      for (let i = 0; i < bigRolls.length; i++) {
        const unusedWidth = bigRolls[i][0];
        const smallRolls = bigRolls[i][1];

        /*
                each bigRoll has it's own row. All of it's cuts have same Y
                */
        x1 = 0;
        y1 = yScale(i);

        for (let j = 0; j < smallRolls.length; j++) {
          const smallRoll = smallRolls[j];

          const width = xScale(smallRoll);
          x2 = x1 + width;

          // add the rectangular strip / bar
          let g = svg.append("g").attr("transform", `translate(${x1},${y1})`); // one vertical bar

          g.append("rect")
            .attr("fill", colorDict[smallRoll]) // <- apply color associated with this width
            .attr("width", width - 1)
            .attr("height", yScale.bandwidth() - 1);

          // add text
          g.append("text")
            .attr("fill", "white")
            .attr("x", 3) // this x is relative to the parent g
            .attr("y", yScale.bandwidth() / 2)
            .attr("dy", "0.35em")
            .text(smallRoll);

          // for next rect, x1 will update to x2 of current rect
          x1 = x2;
        }

        if (unusedWidth > 0) {
          // add unusedWith as rectangular bar
          x2 = x1 + xScale(unusedWidth);
          let g = svg.append("g").attr("transform", `translate(${x1},${y1})`); // one vertical bar

          g.append("rect")
            .attr("fill", this.wasteColor)
            .attr("width", xScale(unusedWidth) - 1)
            .attr("height", yScale.bandwidth() - 1);

          // add text
          g.append("text")
            .attr("fill", "white")
            .attr("x", 3)
            .attr("y", yScale.bandwidth() / 2)
            .attr("dy", "0.35em")
            .text(Math.round(unusedWidth));
        }
      }
      return svg.node();
    },

    /**
        draws the shapes according to data and requirements of 1d algorithm
        */
    draw2d: function () {
      // clear old drawing
      this.clearTheDrawing();

      if (!this.mode_data.result) {
        console.log(
          `Cannot draw anything. "result" is: ${this.mode_data.result} for mode: ${this.mode}`
        );
        return;
      }

      const solutions = this.mode_data.result.solutions;

      for (let i = 0; i < solutions.length; i++) {
        const sol = solutions[i];

        for (let j = 0; j < sol.length; j++) {
          const rect = sol[j];

          let x1 = rect[0];
          let y1 = rect[1];
          let x2 = rect[2];
          let y2 = rect[3];

          const color = this.colors[j % this.colors.length];
          this.drawRect(x1, y1, x2, y2, color);
        }

        // only draw first solution for now
        // TODO: FIXME: Does drawing all solution help?
        break;
      }
    },

    // before drawing new result, clear old canvas drawing
    clearTheDrawing: function () {
      d3.selectAll("#d3_area svg > *").remove();
      // also hide the border of svg
      d3.select("#d3_area svg").style("border", "");
    },

    drawRect: function (x1, y1, x2, y2, color) {
      console.log("Drawing Rect... Color: ", color);
      console.log("Draw rect", x1, y1, x2, y2);

      const width = Math.abs(x2 - x1);
      const height = Math.abs(y2 - y1);
      // const coords = [{ x1, y1, width, height, color }];

      // const rectTitle = `${width} x ${height}`;

      const parentWidth = this.mode_data.parents[0].width;
      const parentHeight = this.mode_data.parents[0].height;
      const dataLen = this.mode_data.parents.length;

      let xScale = d3.scaleLinear().domain([0, parentWidth]).range([0, 300]); // <- TODO here put the dynamic width of chart

      let yScale;

      if (this.mode === "1d") {
        yScale = d3
          .scaleOrdinal()
          .domain(d3.range(0, dataLen)) // <-num big rolls
          .rangeBands(0, 300);
      } else if (this.mode === "2d") {
        yScale = d3
          .scaleLinear()
          .domain([0, parentHeight]) // sum of all stock's heights
          .range([0, 300]); // display in 300 pixels height?
      }

      // const scaleFactor =
      // this.mode === '1d'?
      // Math.floor(300 / maxDimOfParent)
      // 1:
      // Math.floor(300 / maxDimOfParent);

      // const rectX = x1 * scaleFactor;
      // const rectY = this.mode == '1d'? y1: y1 * scaleFactor;
      // const rectW = width * scaleFactor;
      // const rectH = this.mode == '1d'? height: height * scaleFactor;

      // console.log('Scaled rect', rectX, rectY, rectX+rectW, rectY+rectH, scaleFactor);
      console.log(
        "D3 Scaled rect",
        xScale(x1),
        yScale(y1),
        xScale(width),
        yScale(height)
      );

      // create svg element:
      let svg = d3.select("#d3_area svg");

      var margin = { top: 20, right: 20, bottom: 20, left: 20 };

      let svgWidth = 300 - margin.left - margin.right;
      let svgHeight = 300 - margin.top - margin.bottom;

      let g = svg
        .attr("width", svgWidth + margin.left + margin.right)
        .attr("height", svgHeight + margin.top + margin.bottom)
        .style("border", "1px solid #34495e")
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      // Add the path using this helper function
      g
        // .data(coords)
        .append("rect")
        // .style('fill', function (coords) { return coords.color })
        .style("fill", color)
        // .style("background-color", "black")
        // .attr('x', rectX)
        .attr("x", xScale(x1)) //function (coords) { return xScale(coords.x1) })
        // .attr('y', rectY)
        .attr("y", yScale(y1)) //function (coords) {return yScale(coords.y1) })
        // .attr('width', rectW)
        .attr("width", xScale(width)) //function (coords) { return xScale(coords.width) })
        // .attr('height', rectH)
        .attr("height", yScale(height)) //function (coords) { return yScale(coords.height) })
        .attr("stroke", "#34495e")
        .text(`${width} x ${height}`);

      // add the label to the shape
      // const labelX = rectX + Math.abs( rectW / 2 - 15);
      // const labelY = rectY + Math.abs( rectH / 2 + 5);
      const labelX = xScale(x1) + Math.abs(xScale(width) / 2); // - 15);
      const labelY = yScale(y1) + Math.abs(yScale(height) / 2); // + 5);

      //   let label =
      g.append("text")
        .attr("x", labelX)
        .attr("y", labelY)
        .attr("stroke", "#34495e");
      // .attr('stroke', 'black')
      // .style("font-size", 15);

      return;
    },

    /**
     * clears the data in the child table and returns it to it's initial state
     * i.e. 1 empty row
     */
    clearChildData: function (askConfirm) {
      if (askConfirm) {
        const answer = confirm("Are you sure you want to empty this Table?");
        if (!answer) return;
      }

      const emptyChildData =
        this.mode === "1d"
          ? [{ width: "", quantity: "" }]
          : [{ width: "", height: "", quantity: "" }];

      this.mode_data.childs = emptyChildData;

      // also clear errors displayed in Child table (if any)
      this.mode_data.childErrors = null;

      // also clear the result and drawing
      this.mode_data.result = null;
      this.clearTheDrawing();
    },

    clearParentData: function (askConfirm) {
      if (askConfirm) {
        const answer = confirm("Are you sure you want to empty this Table?");
        if (!answer) return;
      }

      const emptyParentData =
        this.mode === "1d"
          ? [{ width: "", quantity: "", weight: 0.0 }]
          : [{ width: "", height: "", quantity: "先不需要输入" }];

      this.mode_data.parents = emptyParentData;

      // also clear errors displayed in Parent table (if any)
      this.mode_data.parentErrors = null;

      // also clear the result and drawing
      this.mode_data.result = null;
      this.clearTheDrawing();
    },

    getSideWeight: function () {

      let side = this.side;
      let side_worst_fun = 0;
      if (side > 0) {
        this.mode_data.parents.forEach((parent) => {
          if (parseFloat(parent.width) > 0 && parseFloat(parent.weight) > 0 && parseFloat(parent.quantity) > 0) {
            side_worst_fun += (parseFloat(this.side) / parseFloat(parent.width)) * parseFloat(parent.weight) * parseFloat(parent.quantity);
          }
        });
      }
      this.side_worst = Math.round(side_worst_fun * 1000) / 1000;

      let s_weight = 0.0;
      this.mode_data.parents.forEach((parent) => {
        if ((parseFloat(parent.weight) > 0) && (parseFloat(parent.quantity) > 0)) {
          s_weight += parseFloat(parent.weight) * parseFloat(parent.quantity);
        }

      });
      this.all_weight = s_weight;
    },

    selectdRow: function (idx) {
      let selectdChild = this.mode_data.childs_for_select[idx];
      for (let index = 0; index < selectdChild.sub_child_solver.length; index++) {
        const ele = selectdChild.sub_child_solver[index];
        this.mode_data.childs[index].quantity = ele.quantity;
      }
      this.rule_selectd_index =  idx;
      this.mode_data.result = {"data":{"solutions":null,"sub_weights":null}};
      this.mode_data.result.data.solutions = this.mode_data.childs_for_select[idx].solutions.solutions;
      this.mode_data.result.data.sub_weights = this.mode_data.childs_for_select[idx].solutions.sub_weights;
      let rolls = [];
      this.mode_data.result.data.solutions.forEach((soluton) => {
          let subs = [];
          soluton.subs.forEach((s) => {
            subs.push(Math.round(parseFloat(s)) / 1000);
          });
          let subs_weight = [];
          soluton.sub_weights.forEach((s) => {
            subs_weight.push(Math.round(parseFloat(s)) / 1000);
          });

          rolls.push([parseFloat(soluton.un_used / 1000), subs, parseFloat(soluton.un_used_weight / 1000),subs_weight]);
        });
        console.log(rolls);
        this.mode_data.result.solutions = rolls;
        let child_index = 0;
        // let all_parent_width = 0;
        // this.mode_data.parents.forEach((parent) => {
        //   all_parent_width += parseFloat(parent.width) * parseFloat(parent.quantity);
        // });
        this.mode_data.result.data.sub_weights.forEach(()=>{
          this.mode_data.childs[child_index].weight = 0;
          child_index += 1;
        });
        child_index = 0;
        this.mode_data.result.data.sub_weights.forEach((w) => {
          if (parseFloat(this.mode_data.childs[child_index].weight)) {
            this.mode_data.childs[child_index].weight = parseFloat(this.mode_data.childs[child_index].weight)+w/1000;
          }else{
            this.mode_data.childs[child_index].weight = w/1000;
          }

          // let sub_widht = parseFloat(this.mode_data.childs[child_index].width) * parseFloat(this.mode_data.childs[child_index].quantity);
          // let all_weight = parseFloat(this.all_weight);
          // this.mode_data.childs[child_index].weight = Math.round(sub_widht/all_parent_width * all_weight * 1000)/1000;
          child_index += 1;
        });
        this.checkValidity1D();
        this.draw1d();
    },

    /**
            Removes the row at the index specified. if it's only row, add an empty row after removing
        */
    removeRow: function (idx, is_parent) {
      if (is_parent) {
        if (this.mode_data.parents.length > 1) {
          this.mode_data.parents.splice(idx, 1);
        } else {
          this.clearParentData(false);
        }
        return;
      }

      // in case of child
      if (this.mode_data.childs.length > 1) {
        this.mode_data.childs.splice(idx, 1);
      } else {
        // there is only one item in the child table,
        // calling clearChildData() will both clear the table and add back an empty row
        this.clearChildData(false);
      }
    },

    /**
        to each of the widths of small rolls in the result of 1d
        assign a unique color
        */
    getColorDict: function () {
      const bigRolls = this.mode_data.result.solutions;

      // assign same color to equal sized small rolls
      let uniqueSmallRollsSet = new Set([]);
      for (let i = 0; i < bigRolls.length; i++) {
        const smallRolls = bigRolls[i][1];

        smallRolls.forEach((roll) => {
          uniqueSmallRollsSet.add(roll);
        });
      }

      let uniqueSmallRolls = Array.from(uniqueSmallRollsSet);
      let colorDict = {};

      for (let i = 0; i < uniqueSmallRolls.length; i++) {
        // colorDict[ width_of_roll ] = '#color'
        colorDict[uniqueSmallRolls[i]] = this.colors[i % this.colors.length];
      }

      // console.log('colorDict:', colorDict)
      return colorDict;
    },

    /**
        returns the color assigned to this width in colorDict
        this is for 1d mode at the moment
        */
    getColor: function (width) {
      const colorDict = this.getColorDict();
      // return colorDict[width];
      return { backgroundColor: `${colorDict[width]}` };
    },

    /**
        clear all the inputs and drawings
        */
    reset: function () {
      const answer = confirm(
        "Are you sure you want to delete all inputs? Cannot be undone"
      );
      if (!answer) return;

      this.clearChildData(false); // it will also clear the drawing
      this.clearParentData(false);
    },

    getRound: function (unusedWidth) {
      let worstWidth = unusedWidth * 100; // preserve 2 digits after decimal
      worstWidth = Math.round(worstWidth); // remove the decimal part
      worstWidth /= 100; // back to original percentage
      return worstWidth;
    },

    getWorstWeight: function (worstWidth, num) {
      let parent_width = this.mode_data.parents[0].width;
      // 计算宽度的重量
      if (parent_width > 0) {
        for (let i = 0; i < this.mode_data.childs.length; i++) {
          const child = this.mode_data.childs[i];
          let percentage = ((child.width * child.quantity) / (parent_width * num)) * this.all_weight;
          percentage *= 1000; // preserve 2 digits after decimal
          percentage = Math.round(percentage); // remove the decimal part
          percentage /= 1000; // back to original percentage
          child.weight = percentage;
        }
      }
      let percentage = (worstWidth / (parent_width * num)) * this.all_weight;
      percentage *= 1000; // preserve 2 digits after decimal
      percentage = Math.round(percentage); // remove the decimal part
      percentage /= 1000; // back to original percentage

      return percentage;
    },

    getPercentageUtilization: function (unusedWidth, list_length) {
      let parentWidth = unusedWidth;
      list_length.forEach((n) => {
        parentWidth += n;
      });
      let usedWidth = Math.abs(parentWidth - unusedWidth);
      let percentage = (usedWidth * 100) / parentWidth;

      percentage *= 100; // preserve 2 digits after decimal
      percentage = Math.round(percentage); // remove the decimal part
      percentage /= 100; // back to original percentage

      return percentage;
    },

    // to display in footer
    getCurrentYear: function () {
      return new Date().getFullYear();
    },

    /**
        implemented only for 1D, download the details of cut in CSV
        TODO: implement for 2D
        */
    downloadCsv: function () {
      if (!this.mode_data.result || !this.mode_data.result.solutions) {
        console.log("downloadCsv: bigRolls are empty..");
        return;
      }

      // prepare data
      let dataForCsv = [["Stock", "Usage", "Width of Cuts"]];
      let numSmallRolls = 0;

      const bigRolls = this.mode_data.result.solutions;
      for (let i = 0; i < bigRolls.length; i++) {
        const unusedWidth = bigRolls[i][0];
        const smallRolls = bigRolls[i][1];

        numSmallRolls += smallRolls.length;

        // ['Stock #', 'Usage', 'Width of Cuts']
        const nextRow = [
          i + 1,
          this.getPercentageUtilization(unusedWidth, smallRolls) + "%",
          smallRolls.join(","),
        ];
        dataForCsv.push(nextRow);
      }

      // convert to CSV format
      // source: https://stackoverflow.com/a/14966131/3578289
      const csvContent =
        "data:text/csv;charset=utf-8," +
        dataForCsv.map((e) => e.join(",")).join("\n");
      // console.log('csvContent: ', csvContent);

      // download the file
      let encodedUri = encodeURI(csvContent);
      // console.log('encodedUri: ', encodedUri);
      let link = document.createElement("a");
      link.setAttribute("href", encodedUri);

      // unique and identifiable filename
      const stockWidth = this.mode_data.parents[0].width;
      const d = new Date();
      const dateString = `${d.getFullYear()}-${d.getMonth() + 1
        }-${d.getUTCDate()}-${d.getHours()}${d.getMinutes()}-${d.getSeconds()}`;

      const filename = `CSP_stock_${stockWidth}_cuts_${numSmallRolls}_${dateString}.csv`;
      link.setAttribute("download", filename);

      document.body.appendChild(link); // Required for FF
      link.click(); // This will download the data file named "my_data.csv".
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import "../assets/style.scss";
</style>
