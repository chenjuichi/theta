<template>
  <v-app>
  <v-container fluid>
    <v-snackbar v-model="snackbar" :color="snackbar_color" :right='snackbar_right' :top='snackbar_top'>
      {{ snackbar_info }}
      <template v-slot:action="{ attrs }">
        <v-btn icon :color="snackbar_icon_color" @click="snackbar= false">
          <v-icon dark>mdi-close-circle</v-icon>
        </v-btn>
      </template>
    </v-snackbar>
    <v-row align="center" justify="center" v-if="currentUser.perm >= 1">
      <v-card class="overflow-hidden mx-auto mt-3" width="95vw">
        <v-toolbar flat color="#7DA79D" height="90vw" >
          <v-row dense style="margin-bottom: -36px; margin-top: -12x;">
            <v-col cols="12" md="2" class="mr-1" style="position: relative; top: -15px;">
              <v-text-field
                v-model="stockInTag_reagID"
                label="資材碼"

                style="position:relative; top: 15px;"
                :value="stockInTag_reagID"
                @keyup.native.enter="handleUpdateItem($event)"
                @keyup.native.backspace="handleBackspace($event)"
                @paste="handlePaste"
              ></v-text-field>

              <!--<v-select
                :items="reagentForSelect"
                label="資材碼"
                style="position:relative; top: 10px;"
                dense
                outlined
                item-color="red"
                v-model="stockInTag_reagID"
              ></v-select>-->
            </v-col>

            <v-col cols="12" md="3" class="mr-2">
              <v-text-field
                v-model="stockInTag_reagName"
                label="品名"
                :value="stockInTag_reagName"
                readonly
                style="width: 300px !important; max-width: 270px !important;"
              ></v-text-field>
            </v-col>

            <v-col cols="12" md="2"
              class="mr-2"
              style="position: relative; top: 5px; left:20px; font-weight: bold; width: 120px !important; max-width: 120px !important;">
              {{stockInTag_station}}站/{{stockInTag_layout}}層/{{stockInTag_pos}}格
            </v-col>

            <v-col cols="12" md="1" align="right" style="position: relative; top: 5px;">
              <!--<span class="text-decoration-underline">&nbsp;&nbsp;入庫數量</span>-->
              <span class="stockinStr" style="display:table-cell; vertical-align:middle; font-weight: bold; font-size: 1em;">入庫數量</span>
            </v-col>

            <v-col cols="12" md="2" align="left" class="pl-0 mx-1" style="max-width:120px; width=120px;">
              <vue-numeric-input
                v-model="stockInTag_cnt"
                :min="1"
                :max="stockInTag_max_cnt"
                :step="1"
                size="small"
                align="center"
              ></vue-numeric-input>
              <span class="stockinStr" style="position: relative; top: -20px; right: -105px; font-weight: bold; font-size: 1em;">
                {{stockInTag_unit}}
              </span>
            </v-col>

            <v-col cols="12" md="1" align="right" class="pl-0 mx-1" style="max-width: 70px; width=120px; text-align: center; position:relative; top:-10px;">
              <!--<img v-show="stockInTag_reagID!=''" v-on:click="redirect_to_mqtt" :src="home_url" alt="Loading" style="height: 3vw;" v-bind:alt="pic">-->
              <img v-show="stockInTag_reagID!=''" @click="redirect_to_mqtt" :src="home_url" alt="Loading" style="height: 3vw;">
            </v-col>

            <v-col cols="12" md="1" align="center" class="pl-0 mx-1" style="text-align: center; position:relative; top:-12px;">
              <!--<v-btn v-show="isOK" @click="redirect_ok" color="success" rounded style="font-weight: bold; font-size: 0.9em;">-->
              <v-btn
                v-show="isOK"
                @click="mqttForStationOff"
                color="success"
                rounded
                style="font-weight: bold; font-size: 0.9em; position:relative; top:10px;">
                <!--<v-icon>mdi-domain</v-icon>-->完成
              </v-btn>
            </v-col>
          </v-row>
        </v-toolbar>
        <v-sheet class="overflow-y-auto" max-height="600">
          <v-container>
            <v-list three-line>
              <v-list-item-group v-model="model" mandatory color="indigo" @change="listActionClick_m(model)">
                <!--<template v-for="(item, index) in items">-->
                  <!--<v-list-item :key="item.stockInTag_reagName">-->
                <!--<template v-for="(item, index) in filteredItems">-->
                <template v-for="(item, index) in items">
                  <v-list-item>
                    <template v-slot:default="{ active }">
                      <v-list-item-content>
                        <!--<v-list-item-title class="font-weight-bold" v-text="`品名:${item.stockInTag_reagName}    入庫數量: ${item.stockInTag_cnt}`"></v-list-item-title>-->
                        <v-list-item-title class="font-weight-bold">
                          <span>資材碼:{{item.stockInTag_reagID}}</span>&nbsp;&nbsp;
                          <span>品名:{{item.stockInTag_reagName}}</span>&nbsp;&nbsp;
                          <span class="text-decoration-underline">入庫數量: {{item.stockInTag_cnt}}&nbsp;{{item.stockInTag_unit}}</span>
                        </v-list-item-title>
                        <v-list-item-subtitle class="mb-3" v-text="`效期:${item.stockInTag_reagPeriod}  保存溫度: ${item.stockInTag_reagTemp}  批號: ${item.stockInTag_batch}`"></v-list-item-subtitle>

                        <v-list-item-subtitle class="text--primary font-weight-bold" v-text="`入庫人員: ${item.stockInTag_Employer}  入庫日期: ${item.stockInTag_Date}`"
                        ></v-list-item-subtitle>
                      </v-list-item-content>

                      <v-list-item-action>
                      <!--<v-list-item-action @click="listActionClick(index, active)">-->
                        <!--<v-list-item-action-text v-text="item.stockInTag_batch"></v-list-item-action-text>-->
                        <!--<img v-show="active" :src="home_url" alt="Loading" style="height: 2.5vw;">-->
                        <v-icon v-if="!active" color="grey lighten-1">
                          mdi-star-outline
                        </v-icon>
                        <v-icon v-else color="yellow darken-3">
                          mdi-star
                        </v-icon>
                        <div v-show="active">{{item.grid_station}}站/{{item.grid_layout}}層/{{item.grid_pos}}格</div>
                      </v-list-item-action>
                    </template>
                  </v-list-item>

                  <v-divider v-if="index < items.length - 1" :key="index"></v-divider>
                </template>
              </v-list-item-group>
            </v-list>
          </v-container>
        </v-sheet>
      </v-card>

      <v-dialog
        v-model="isEmptyRecord"
        transition="dialog-bottom-transition"
        persistent
        max-width="600"
      >
        <v-card>
          <v-toolbar
            color="primary"
            dark
          >訊息!</v-toolbar>
          <v-card-text>
            <div class="text-h4 pa-12">入庫完成或暫無待入庫試劑...</div>
          </v-card-text>
          <v-card-actions class="justify-end">
            <v-spacer></v-spacer>
            <v-btn text @click="routeNavbar"> 離開 </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>

    <v-row align="center" justify="space-around" v-else>
        <v-dialog
          v-model="permDialog"
          transition="dialog-bottom-transition"
          max-width="500"
        >
          <v-card>
            <v-toolbar
              color="primary"
              dark
            >錯誤訊息!</v-toolbar>
            <v-card-text>
              <div class="text-h4 pa-12">使用這項功能, 請通知管理人員...</div>
            </v-card-text>
            <v-card-actions class="justify-end">
              <v-spacer></v-spacer>
              <v-btn text @click="permCloseFun"> 取消 </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
    </v-row>
  </v-container>
   </v-app>
</template>

<script>
import axios from 'axios';
import VueNumericInput from 'vue-numeric-input';

//import logo from '../../assets/image/icons8-light-on.gif'
import logo from '../../assets/image/icons8-light-on-unscreen.gif'
import logoR from '../../assets/image/icons8-light-on-R-m.gif'
import logoY from '../../assets/image/icons8-light-on-Y-m.gif'
import logoG from '../../assets/image/icons8-light-on-G-m.gif'
import _ from 'lodash'

import Common from '../../mixin/common.js'

export default {
  name: 'StockIn',

  mixins: [Common],

  components: {
    VueNumericInput,
  },

  mounted() {
    // if back button is pressed
    window.onpopstate = () => {
      console.log("press back button, good bye...");

      const userData = JSON.parse(localStorage.getItem('loginedUser'));
      userData.setting_items_per_page = this.pagination.itemsPerPage;
      localStorage.setItem('loginedUser', JSON.stringify(userData));
    };
    //
    //this.stockInTag_cnt=this.items[this.model].stockInTag_cnt;
    //this.stockInTag_max_cnt=this.items[this.model].stockInTag_cnt;

    //this.stockInTag_min_cnt=1;
    //this.stockInTag_reagID=this.items[this.model].stockInTag_reagID;
    //this.stockInTag_reagName=this.items[this.model].stockInTag_reagName;
    //this.items[this.model].active=true;
    //
  },

  data: () => ({
    currentUser: {
      	//empID: null,
        //name: null,
				//dep: null,
        //perm: 0,    //member: 2, admin: 1, none:0
    },
    permDialog: false,

    snackbar: false,
    snackbar_color: 'success',
    snackbar_right: true,
    snackbar_top: true,
    snackbar_info: '',
    snackbar_icon_color: '#adadad',

    default_home_url: logo,
    home_url: logo,
    home_url_R: logoR,
    home_url_Y: logoY,
    home_url_G: logoG,

    stockInTag_reagID: '',
    stockInTag_reagName: '',

    stockInTag_station: '',
    stockInTag_layout: '',
    stockInTag_pos: '',

    stockInTag_cnt: 0,
    current_cnt:0,
    stockInTag_min_cnt: 0,
    stockInTag_max_cnt: 0,
    stockInTag_unit: '',

    currentIndex: 0,
    currentLedStation: 1,
    currentLedLayout: 1,
    currentLedPos: 1,
    currentLedRange_begin: 1,
    currentLedRange_end: 2,

    current_unit: '',   //add

    pre_topic: 0,

    pagination: {
      //itemsPerPage: 10,   //預設值, rows/per page
      //page: 1,
    },

    grids: [],
    temp_grids : [],

    isSort: true,   //textfield輸入資材碼
    isOK: false,    //是否顯示入庫完成的按鍵
    isEmptyRecord: false,
    emptyRecordReady: false,

    //selected: [2],
    model: 0,

    items: [],
    temp_items : [],

    reagentForSelect: [],
    stockin_record: {},

    mqtt_topic:['station1','station2','station3'],

    load_SingleTable_ok: false, //for get employer table data
    load_2thTable_ok: false,    //for get reagent table data
    load_3thTable_ok: false,
    load_4thTable_ok: false,
    load_5thTable_ok: false,
  }),

  computed: {
    filteredItems() {
    //透過filter()函式的第一參數, 進行callback函式所指定的過濾條件，並返回一個新陣列
    //透過includes(), 回傳是否包含該元素的Boolean值，true代表包含，false則代表不包含
      let temp_array=[];
      temp_array=
        _.orderBy(this.items.filter(item => {
        return item.stockInTag_reagID.toLowerCase().includes(this.stockInTag_reagID.toLowerCase());
      }), 'stockInTag_reagID');
      //console.log("filter: ", temp_array)
      return temp_array;
    },
  },

  watch: {
    stockInTag_reagID (val) {
      this.fromReagIdDisp();
    },

    currentIndex(newVal, oldVal) {
      console.log("new/old currentIndex: ", newVal, oldVal)
    },

    load_SingleTable_ok(val) {
      console.log("load_SingleTable_ok, desserts: ", val);

      if (val) {
        this.items = Object.assign([], this.temp_items);
        this.load_SingleTable_ok=false;

        this.listStockInGrids();
      }
    },

    load_2thTable_ok(val) {
      console.log("load_2thTable_ok, products: ", val)

      if (val) {
        this.grids = Object.assign([], this.temp_grids);

        this.load_2thTable_ok=false;

        this.addGrids();
      }
    },

    load_3thTable_ok(val) {
      console.log("load_3thTable_ok: ", val);

      if (val) {
        this.load_3thTable_ok=false;
        this.addStockInItem();
      }
    },

    load_4thTable_ok(val) {
      console.log("load_4thTable_ok: ", val);

      if (val) {
        this.load_4thTable_ok=false;
      }
    },

    load_5thTable_ok(val) {
      console.log("load_5thTable_ok: ", val);

      if (val) {
        this.load_5thTable_ok=false;
        this.isOK=false;
        console.log("StockIn, load_5thTable_ok, isOk: ", this.isOK);
        this.home_url=this.default_home_url;    //2023-1-12 add

        if (this.emptyRecordReady)      //2023-02-14 add
          this.isEmptyRecord = true;    //2023-02-14 add
      }
    },
  },

  created () {
    this.currentUser = JSON.parse(localStorage.getItem("loginedUser"));
    if (this.currentUser.perm == 0) {
      this.permDialog=true;
      //console.log("router undefine!")
    }

    this.pagination.itemsPerPage=this.currentUser.setting_items_per_page

    this.load_SingleTable_ok=false;
    this.initAxios();

    this.listStockInItems();

    //this.initialize();
  },

  methods: {
    initialize () {
      this.load_SingleTable_ok=false;
      this.listStockInItems();
    },

    listStockInItems() {
      const path = '/listStockInItems';
      console.log("listStockInItems, Axios get data...")
      axios.get(path)
      .then((res) => {
        this.temp_items = res.data.outputs;
        console.log("GET ok, total records:", res.data.outputs.length);
        if (!res.data.status) {
          this.isEmptyRecord=true;
        } else {
          this.reagentForSelect = this.temp_items.map(function(p) {     //
            return p.stockInTag_reagID;
          });
          this.reagentForSelect = [...new Set(this.reagentForSelect)];  //去除重複項目
          this.load_SingleTable_ok=true;
        }
      })
      .catch((error) => {
        console.error(error);
        console.log("通訊錯誤!");
        this.snackbar_color='red accent-2';
        this.snackbar=true;
        this.snackbar_info= '通訊錯誤!';
        this.snackbar_icon_color= '#adadad';
        this.load_SingleTable_ok=false;
      });
    },

    listStockInGrids() {
      const path = '/listStockInGrids';
      console.log("listStockInGrids, Axios get data...")
      axios.get(path)
      .then((res) => {
        this.temp_grids = res.data.outputs;
        console.log("GET ok, total records:", res.data.outputs.length);
        this.load_2thTable_ok = true;
      })
      .catch((error) => {
        console.error(error);
        console.log("通訊錯誤!");
        this.snackbar_color='red accent-2';
        this.snackbar=true;
        this.snackbar_info= '通訊錯誤!';
        this.snackbar_icon_color= '#adadad';
        this.load_2thTable_ok = false;
      });
    },

    routeNavbar() {
      this.isEmptyRecord = false;
      this.$router.push('Navbar');
    },

    addStockInItem() {  //新增 後端table資料
      console.log("---click addStockInItem data---");

      const path='/addStockInItem';
      let object = Object.assign({}, this.stockin_record);
      console.log("InTagID: ", object, this.stockInTag_cnt)
      let cnt = object.stockInTag_cnt!=this.stockInTag_cnt ? Math.abs(object.stockInTag_cnt-this.stockInTag_cnt):this.stockInTag_cnt;

      var payload= {
        InTagID: object.id,
        InTagCount: object.stockInTag_cnt,
      };
      axios.post(path, payload)
      .then(res => {
        console.log("add StockIn data, status: ", res.data.status);
        //this.tosterOK = res.data.status? false:true;  ////false: 關閉錯誤訊息畫面
        this.load_4thTable_ok = true;
      })
      .catch(err => {
        console.error(err);
        //this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        this.load_4thTable_ok = false;
      });
    },

    handlePaste (evt) {
      console.log('on paste', evt.clipboardData.getData('text'))

      //console.log('on paste', evt.clipboardData.getData('text/plain'))
    },

    handleBackspace (e) {
      console.log("press backspace...");
      this.stockInTag_reagID='';
    },

    handleUpdateItem (e) {
      console.log("press return...");

      let matchResult = this.items.find(x => x.stockInTag_reagID === this.stockInTag_reagID);
      if (typeof(matchResult) == 'undefined') {
        console.log("stockOutTag_reagID is undefined...");

        this.snackbar_color='red accent-2';
        this.snackbar=true;
        this.snackbar_info= '領料資材碼錯誤!';
        this.snackbar_icon_color= '#adadad';
        this.load_SingleTable_ok=false;
      }
      //else {
        this.fromReagIdDisp();
      //}
    },

    async mqttForStation() {
      console.log("mqttForStation...");

      let path='/mqtt/station';
      let temp_layout=this.currentLedLayout;
      let range_begin=this.currentLedRange_begin;
      let range_end=this.currentLedRange_end;
      console.log("station: " +"layout: " + temp_layout + " begin: " + range_begin + " end: " + range_end, this.model, this.items[this.model])
      let temp_sw= 'flash';
      let myTopic=this.mqtt_topic[parseInt(this.currentLedStation) - 1]
      let payload= {
        topic: myTopic,
        layout: temp_layout,
        pos_begin: range_begin,
        pos_end: range_end,
        msg: temp_sw,
      };

      try {
        let res = await axios.post(path, payload);
        console.log("mqtt ok", res.data.status);
        this.pre_topic=myTopic;     //2023-1-5 add
        this.load_3thTable_ok=true;
      } catch (err) {
        console.error(err)
        console.log("通訊錯誤!");
        this.snackbar_color='red accent-2';
        this.snackbar=true;
        this.snackbar_info= '通訊錯誤!';
        this.snackbar_icon_color= '#adadad';
        this.load_3thTable_ok=false;
      }
    },

    async mqttForStationOff() {
      let path='/mqtt/station';
      let temp_layout='0';
      let range_begin='0';
      let range_end='0';
      let temp_sw= 'off';
      let myTopic=this.pre_topic;
      let payload= {
        topic: myTopic,
        layout: temp_layout,
        pos_begin: range_begin,
        pos_end: range_end,
        msg: temp_sw,
      };

      try {
        let res = await axios.post(path, payload);
        console.log("off led, mqtt ok", res.data.status);
        this.load_5thTable_ok=true;
      } catch (err) {
        console.error(err)
        console.log("通訊錯誤!");
        this.snackbar_color='red accent-2';
        this.snackbar=true;
        this.snackbar_info= '通訊錯誤!';
        this.snackbar_icon_color= '#adadad';
        this.load_5thTable_ok=false;
      }
    },

    listActionClick_m(index) {
      if (this.items.length==0) {
        console.log("HELLO1...", index, this.items[index]);

        this.home_url=this.default_home_url;
        //this.isOK=false;  //2023-01-06 mark

        this.stockInTag_cnt=0;
        this.stockInTag_max_cnt=0;
        this.stockInTag_min_cnt=0;

        this.stockInTag_unit='';

        this.stockInTag_reagID='';
        this.stockInTag_reagName='';

        this.stockInTag_station='';
        this.stockInTag_layout='';
        this.stockInTag_pos='';
      } else {
        console.log("HELLO2...", index, this.items[index]);

        this.currentIndex=index;
        this.currentLedStation=this.items[index].grid_station;
        this.currentLedLayout=this.items[index].grid_layout;
        this.currentLedPos=this.items[index].grid_pos;
        this.currentLedRange_begin=this.items[index].range0;
        this.currentLedRange_end=this.items[index].range1;

        this.current_unit=this.items[index].stockInTag_unit; //2023-04-27 add

        console.log("id, current: ",this.items[index].id, this.currentIndex, this.currentLedStation, this.currentLedLayout, this.currentLedPos, this.currentLedRange_begin, this.currentLedRange_end)

        this.isSort=false;
        this.stockInTag_cnt=this.items[index].stockInTag_cnt;

        this.stockInTag_max_cnt=this.items[index].stockInTag_cnt;
        this.stockInTag_min_cnt=1;

        this.stockInTag_unit=this.items[index].stockInTag_unit;

        this.stockInTag_reagID=this.items[index].stockInTag_reagID;
        this.stockInTag_reagName=this.items[index].stockInTag_reagName;

        this.stockInTag_station=this.items[index].grid_station;
        this.stockInTag_layout=this.items[index].grid_layout;
        this.stockInTag_pos=this.items[index].grid_pos;
      }
    },

    fromReagIdDisp() {
      if (this.stockInTag_reagID != '' && this.isSort) {
        console.log("result 1-1...", this.items);
        const objIndex = this.items.findIndex((obj => obj['active'] == true));
        console.log("result 1-1-1...", objIndex);
        if (objIndex != -1)
          this.items[objIndex].active = false;
        const fromIndex = this.items.map(object => object.stockInTag_reagID).indexOf(this.stockInTag_reagID);
        const toIndex = 0;
        console.log("result 1-2...", fromIndex);
        if (fromIndex != -1) {
          const element = this.items.splice(fromIndex, 1)[0];
          this.items.splice(toIndex, 0, element);
          this.model=0;
          this.items[0].active = true;
          this.current_cnt=this.stockInTag_cnt
          this.stockInTag_cnt=this.items[0].stockInTag_cnt;
          this.stockInTag_max_cnt=this.items[0].stockInTag_cnt;
          this.stockInTag_min_cnt=1;
          this.stockInTag_unit=this.items[0].stockInTag_unit;

          this.stockInTag_reagName=this.items[0].stockInTag_reagName;
          //add
          this.stockInTag_station=this.items[0].grid_station;
          this.stockInTag_layout=this.items[0].grid_layout;
          this.stockInTag_pos=this.items[0].grid_pos;

          this.currentIndex=0;
          this.currentLedStation=this.items[0].grid_station;
          this.currentLedLayout=this.items[0].grid_layout;
          this.currentLedPos=this.items[0].grid_pos;

          this.currentLedRange_begin=this.items[0].range0;  //2023-1-12 add
          this.currentLedRange_end=this.items[0].range1;  //2023-1-12 add
          //
          //
        }
        //if (index != -1) {
        //  console.log("result 2...", index);
        //  this.model=index;
        //}
      } else {
        this.isSort=true;
        if (this.stockInTag_reagID == '') {
          this.stockInTag_cnt=0;
          this.stockInTag_max_cnt=0;
          this.stockInTag_min_cnt=0;
          this.stockInTag_unit='';

          this.stockInTag_reagName='';
          //add
          this.stockInTag_station='';
          this.stockInTag_layout='';
          this.stockInTag_pos='';
        }
      }
    },

    addGrids() {
      for (let i = 0; i < this.items.length; i++) {
        let obj = this.grids.find(o => o.grid_reagID === this.items[i].stockInTag_reagID)
        console.log("i, grid: ", i, obj)
        if (typeof(obj) !== 'undefined') {
          this.items[i].grid_station=obj.grid_station;
          this.items[i].grid_layout=obj.grid_layout;
          //this.items[i].grid_id=obj.grid_id,
          this.items[i].grid_id=obj.id,
          this.items[i].grid_pos=obj.grid_pos;
          this.items[i].seg_id=obj.seg_id;
          this.items[i].range0=obj.range0;
          this.items[i].range1=obj.range1;
        }
      };

      this.stockInTag_cnt=this.items[this.model].stockInTag_cnt;
      this.stockInTag_max_cnt=this.items[this.model].stockInTag_cnt;
      this.stockInTag_min_cnt=1;
      this.stockInTag_unit=this.items[this.model].stockInTag_unit;

      //
      this.stockInTag_reagID=this.items[this.model].stockInTag_reagID;
      this.stockInTag_reagName=this.items[this.model].stockInTag_reagName;

      this.stockInTag_station=this.items[this.model].grid_station;
      this.stockInTag_layout=this.items[this.model].grid_layout;
      this.stockInTag_pos=this.items[this.model].grid_pos;

      this.items[this.model].active=true;
    },

    redirect_to_mqtt() {
      console.log("redirect_to_mqtt, hello click image button...", this.model);
      console.log("current: ", this.currentLedLayout, "  ", this.currentLedRange_begin, "  ", this.currentLedRange_end);

      this.isOK=true; //2023-01-06 mark

      let temp_sw=this.items[this.model].grid_station;
      console.log("StockIn: isOK, temp_sw",this.isOK, temp_sw);
      if (temp_sw==1) {
        this.home_url=this.home_url_R;
      }
      if (temp_sw==2) {
        //console.log("color: ", temp_sw, this.home_url)
        this.home_url=this.home_url_Y;
      }
      if (temp_sw==3) {
        //console.log("color: ", temp_sw, this.home_url)
        this.home_url=this.home_url_G;
      }
      this.redirect_ok()
    },

    redirect_ok() {
      //this.isOK=false;    //2023-1-12 mark
      //this.home_url=this.default_home_url;    //2023-1-12 mark
      console.log("redirect_ok: ", this.items.length, this.model)
      if (this.stockInTag_cnt==this.items[this.model].stockInTag_cnt) {
        this.stockin_record=this.items[this.model];
        let removedEl = this.items.splice(this.model, 1); //remove object(index: this.model) from array(this.items)
        this.stockin_record=removedEl[0];

        this.model=(this.model-1>= 0) ? this.model-1 : 0;

        this.stockInTag_reagID = '';
        this.stockInTag_cnt=0;
        this.stockInTag_max_cnt=0;
        this.stockInTag_min_cnt=0;

        this.stockInTag_unit='';

        this.stockInTag_reagName='';
        //add
        this.stockInTag_station='';
        this.stockInTag_layout='';
        this.stockInTag_pos='';

        if (this.items.length > 0) {
          this.stockInTag_cnt=this.items[this.model].stockInTag_cnt;
          this.stockInTag_max_cnt=this.items[this.model].stockInTag_cnt;
          this.stockInTag_min_cnt=1;
          this.stockInTag_unit=this.items[this.model].stockInTag_unit;

          this.stockInTag_reagID=this.items[this.model].stockInTag_reagID;
          this.stockInTag_reagName=this.items[this.model].stockInTag_reagName;

          this.stockInTag_station=this.items[this.model].grid_station;
          this.stockInTag_layout=this.items[this.model].grid_layout;
          this.stockInTag_pos=this.items[this.model].grid_pos;

          this.items[this.model].active=true;
        } else {
          this.emptyRecordReady=true; //2023-02-14 add
          //this.isEmptyRecord=true;  //2023-02-14 mark
        }
      } else {
        this.items[this.model].stockInTag_cnt=this.items[this.model].stockInTag_cnt - this.stockInTag_cnt;
        this.stockInTag_cnt=this.items[this.model].stockInTag_cnt;
        this.stockInTag_max_cnt=this.items[this.model].stockInTag_cnt;
        this.stockin_record=this.items[this.model];
      }
      console.log("current, mqttForStation: ", this.currentLedLayout, "  ", this.currentLedRange_begin, "  ", this.currentLedRange_end);

      this.mqttForStation();
    },

    permCloseFun () {
      this.permDialog = false
      console.log("press permission Close Button...");
      this.$router.push('/navbar');
    },
  },
}
</script>

<style lang="scss" scoped>
html {
  overflow: hidden !important;
}

span.text-decoration-underline {
  margin-top: 13px;
}

span.stockinStr {
  margin-top: 13px;
}

.v-card {
  display: flex !important;
  flex-direction: column;
}

.v-card__text {
  flex-grow: 1;
  overflow: auto;
}

.v-card.on-hover.theme--dark {
  background-color: rgba(255, 255, 255, 0.8);
}

.v-card.on-hover.theme--dark > .v-card__text {
    color: #000;
}
</style>