<template>
  <v-app ref="barcode">
    <v-container class="scroll-y">
      <v-layout align-center justify-center id="layout">
        <v-flex xs12>
          <v-col class="text-right">
            <!--
            <v-btn color="primary" dark class="mb-2" v-print="printObj" @click="printBarcode">
              <v-icon left dark>mdi-printer</v-icon>列印
            </v-btn>
            -->
            <v-btn color="primary" dark class="mb-2" @click="printBarcode">
              <v-icon left dark>mdi-printer</v-icon>列印
            </v-btn>

          </v-col>
          <v-btn v-scroll="onScroll" v-show="fab" fab dark fixed bottom right color="primary"
              @click="toTop"
          >
            <v-icon>keyboard_arrow_up</v-icon>
          </v-btn>
        </v-flex>
      </v-layout>

      <v-layout align-center justify-center>
        <v-flex xs12>
          <v-card id="printMe">
          <!--<div v-for="(field, index) in barcode_data" :key="index" class="container_barcode">-->
          <div v-for="(field, index) in temp_barcode_data" :key="index" class="container_barcode">
            <!--<div>中國醫藥大學新竹附設醫院</div>-->
            <ul>
              <li>{{field.stockInTag_reagName }}</li>
            </ul>

            <barcode :value="field.stockInTag_reagID"></barcode>
            <div style="font-size: 10px; display: flex; justify-content: flex-start; margin-left:20px;">
              LOT: {{ field.stockInTag_batch }}
            </div>
            <div class="word">
              <span v-if="field.stockInTag_rePrint=='入庫'" style="font-size: 12px; margin-left:20px; position: relative; top: -10px;">
                入庫日期: {{ field.stockInTag_Date }}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              </span>
              <span style="font-size: 12px; margin-left:20px; position: relative; top: -10px;" v-else>
                領料日期: {{ field.stockInTag_Date }}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              </span>
              <span style="font-size: 32px; position: relative; top: 15px; right:-50px;">
                {{ field.stockInTag_alpha }}
              </span>
            </div>
            <div v-if="field.stockInTag_rePrint=='入庫'" style="font-size: 12px; font-weight:600; display: flex; justify-content: flex-start; margin-left:20px; position: relative; top: -15px;">
              入庫人員: {{ field.stockInTag_Employer }}
            </div>
            <div style="font-size: 12px; font-weight:600; display: flex; justify-content: flex-start; margin-left:20px; position: relative; top: -15px;" v-else>
              領料人員: {{ field.stockInTag_Employer }}
            </div>
            <div style="font-size: 12px; display: flex; justify-content: flex-start; margin-left:20px; position: relative; top: -15px;">
              保存溫度: {{ field.stockInTag_reagTemp }}
            </div>
            <v-spacer></v-spacer>
          </div>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import VueBarcode from 'vue-barcode';
import * as easings from 'vuetify/lib/services/goto/easing-patterns';

import axios from 'axios';
import Common from '../mixin/common.js'

export default {
  name: 'BarCodeRePrintTag',

  mixins: [Common],

  components: {
    'barcode': VueBarcode,
  },

  props: ['barcode_data'],

  //directives: {
  //  print
  //},

  mounted() {
    console.log("BarCodeRePrintTag, mounted()...");

    this.startTimer();
  },

  created() {
    console.log("BarCodeRePrintTag, created()...");

    this.initAxios();
  },

  computed: {
    asrsEmp() {
      return this.asrs ? '入庫人員' : '領料人員';
    },

    asrsDate() {
      return this.asrs ? '入庫日期' : '領料日期';
    },
  },

  watch: {
    barcode_data (val) {
      console.log("this.barcode_data[0]: ", this.barcode_data[0])
      //if ('stockInTag_cnt' in this.barcode_data[0]) {   //入庫
      //if (this.barcode_data[0]['stockInTag_rePrint'].includes('入')) {   //入庫, 2023-07-20 mark
      /* 2023-08-08 mark
      if ((this.barcode_data[0].hasOwnProperty('stockInTag_rePrint') && this.barcode_data[0].stockInTag_rePrint=='入庫') || (!this.barcode_data[0].hasOwnProperty('stockInTag_rePrint') && this.barcode_data[0].hasOwnProperty('stockInTag_alpha'))) {   //入庫, 2023-07-20 modify
        this.asrs = true;
        console.log("入庫...", this.asrs )

        //this.temp_barcode_data =  JSON.parse(JSON.stringify(this.barcode_data));

        //let uniqueRegFields = [...new Set(this.temp_barcode_data)];
        //console.log("watch: ", this.temp_barcode_data, uniqueRegFields)
        //this.load_SingleTable_ok=false;
        //this.getLastAlphaForUniqueStockIn(uniqueRegFields);
        this.temp_barcode_data =  JSON.parse(JSON.stringify(this.barcode_data));  //2023-08-08 add

        //this.asrs = true;
      } else {     //出庫
        this.asrs = false;
        console.log("出庫...", this.asrs)
      */
        let temp_len = this.barcode_data.length;
        this.temp_barcode_data= [];
        for (let i=0; i < temp_len; i++) {
          let obj= {}
          if (this.barcode_data[i].hasOwnProperty('stockOutTag_Employer')) {  //出庫
            let objOut= {
              //stockOutTag_InID: this.barcode_data[i].stockOutTag_InID,
              stockInTag_reagID: this.barcode_data[i].stockOutTag_reagID,
              stockInTag_batch: this.barcode_data[i].stockOutTag_batch,
              stockInTag_Date: this.barcode_data[i].stockOutTag_Out_Date,        //在列印條碼時, 此對應道出庫日期
              stockInTag_Employer: this.barcode_data[i].stockOutTag_Employer,
              stockInTag_reagTemp: this.barcode_data[i].stockOutTag_reagTemp,
              stockInTag_alpha: this.barcode_data[i].stockOutTag_alpha,
              stockInTag_cnt: this.barcode_data[i].stockOutTag_cnt,
              stockInTag_reagName: this.barcode_data[i].stockOutTag_reagName,
              stockInTag_rePrint: this.barcode_data[i].stockInTag_rePrint,
              isIn: false,
            }
            this.temp_barcode_data.push(objOut);
          } else {  //入庫
            let objIn= {
              stockInTag_reagID: this.barcode_data[i].stockInTag_reagID,
              stockInTag_batch: this.barcode_data[i].stockInTag_batch,
              stockInTag_Date: this.barcode_data[i].stockInTag_Date,        //在列印條碼時, 此對應道出庫日期
              stockInTag_Employer: this.barcode_data[i].stockInTag_Employer,
              stockInTag_reagTemp: this.barcode_data[i].stockInTag_reagTemp,
              stockInTag_alpha: this.barcode_data[i].stockInTag_alpha,
              stockInTag_cnt: this.barcode_data[i].stockInTag_cnt,
              stockInTag_reagName: this.barcode_data[i].stockInTag_reagName,
              stockInTag_rePrint: this.barcode_data[i].stockInTag_rePrint,
              isIn: true,
            }
            this.temp_barcode_data.push(objIn);
          }
        }
        //this.temp_barcode_data.push(obj);

        //let uniqueRegFields = [...new Set(this.temp_barcode_data)];     // 2023-08-08 mark
        //console.log("watch: ", this.temp_barcode_data, uniqueRegFields)
        //this.load_SingleTable_ok=false;   //2023-08-08 mark
        //this.getLastAlphaForUniqueStockOut(uniqueRegFields);

        //this.asrs = false;
      /* 2023-08-08 mark
      }
      */
      //this.temp_barcode_data =  JSON.parse(JSON.stringify(this.barcode_data));  //2023-08-08 mark

      let uniqueRegFields = [...new Set(this.temp_barcode_data)];
      console.log("watch: ", this.temp_barcode_data, uniqueRegFields)
      // 2023-08-08 mark the following line
      //this.temp_barcode_data = this.temp_barcode_data.map(v => ({...v, isIn: this.asrs})) //新增object內的key(isIn), true: 入庫資料
      //
      console.log("----bar1, ", this.barcode_data);
      console.log("----bar2, ", this.temp_barcode_data);
      //this.load_SingleTable_ok=false;
    },

    load_SingleTable_ok(val) {
      console.log("load_SingleTable_ok: ", val);

      if (val) {
        this.load_SingleTable_ok=false;
        this.assignLastAlpha();
      }
    },

    load_2thTable_ok(val) {
      console.log("load_2thTable_ok: ", val);

      if (val) {
        this.load_2thTable_ok=false;
        this.assignLastAlpha();
      }
    },
  },

  data() {
    return {
      fab: false,

      tosterOK: false,

      myTimer: '',            //在component內設定timer, timer的handle
      myTimerId: '',          //timer的id
      svg_change: true,
      temp_barcode_data: [],
      asrs: true,   //true:入庫

      last_alpha_records: [],
      load_SingleTable_ok: false,
      load_2thTable_ok: false,
    };
  },

  destoyed() {
    clearInterval(this.myTimer);
  },

  beforeRouteLeave(to, from, next) {
    clearInterval(this.myTimer);
    next();
  },

  methods: {
    getLastAlphaForUniqueStockIn(uniqueArray) {
      console.log("BarCodeRePrintTag, getLastAlphaForUniqueStockIn(), ", uniqueArray);

      const path = '/getLastAlphaForUniqueStockIn';
      let payload = Object.assign([], uniqueArray);
      axios.post(path, payload)
      .then(res => {
        console.log("getLastAlphaForUniqueStockIn, GET ok.....", res.data.outputs);
        //this.last_status=res.data.status;
        this.last_alpha_records=res.data.outputs;
        //this.tosterOK = false;  //false: 關閉錯誤訊息畫面
        this.load_SingleTable_ok=true;
      })
      .catch((error) => {
        console.error(error);
        //this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        this.load_SingleTable_ok=false;
      });
    },

    assignLastAlpha() {
      console.log("BarCodeRePrintTag, assignLastAlpha()...");

      let temp1_len=this.last_alpha_records.length;
      let temp2_len=this.temp_barcode_data.length;
      let temp_date='';
      console.log("temp1_len, temp2_len: ", temp1_len, temp2_len);

      this.temp_barcode_data.sort(function (a, b) {
          return a.stockInTag_reagID.localeCompare(b.stockInTag_reagID) ||
          //a.stockInTag_Date.localeCompare(b.stockInTag_Date);
          a.stockInTag_batch.localeCompare(b.stockInTag_batch);
      });

      console.log("this.temp_barcode_data: ", this.temp_barcode_data);

      for (let i=0; i<temp1_len; i++) {
        for (let j=0; j<temp2_len; j++) {
          if (this.temp_barcode_data[j].stockInTag_reagID==this.last_alpha_records[i].reagent_id) {
            this.temp_barcode_data[j].stockInTag_alpha=this.last_alpha_records[i].lastAlpha;
            /*
            let tt=(j+1)<temp2_len?j+1:j;
            if (this.temp_barcode_data[j].stockInTag_Date != this.temp_barcode_data[tt].stockInTag_Date) {
              let cc=this.last_alpha_records[i].lastAlpha.charCodeAt(0) + 1;
              this.last_alpha_records[i].lastAlpha=String.fromCharCode(cc);
            } else {
              if (this.temp_barcode_data[j].stockInTag_batch != this.temp_barcode_data[tt].stockInTag_batch) {
                let cc=this.last_alpha_records[i].lastAlpha.charCodeAt(0) + 1;
                this.last_alpha_records[i].lastAlpha=String.fromCharCode(cc);
              }
            }
            */
          }
        }
      }
    },

    getLastAlphaForUniqueStockOut(uniqueArray) {
      console.log("BarCodeRePrintTag, getLastAlphaForUniqueStockOut(), ", uniqueArray);

      const path = '/getLastAlphaForUniqueStockOut';
      let payload = Object.assign([], uniqueArray);
      axios.post(path, payload)
      .then(res => {
        console.log("getLastBatchAlphaForStockOut, GET ok.....", res.data.outputs);
        //this.last_status=res.data.status;
        this.last_alpha_records=res.data.outputs;
        //this.tosterOK = false;  //false: 關閉錯誤訊息畫面
        this.load_2thTable_ok=true;
      })
      .catch((error) => {
        console.error(error);
        //this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        this.load_2thTable_ok=false;
      });
    },
    /*
    assignLastAlphaForStockOut() {
      let temp1_len=this.last_alpha_records.length;
      let temp2_len=this.temp_barcode_data.length;
      console.log("this.last_alpha_records", this.temp_barcode_data, this.last_alpha_records)
      for (let i=0; i<temp1_len; i++) {
        for (let j=0; j<temp2_len; j++) {
          if (this.temp_barcode_data[j].stockOutTag_reagID==this.last_alpha_records[i].reagent_id)
            this.temp_barcode_data[j].stockOutTag_alpha=this.last_alpha_records[i].lastAlpha;
        }
      }
    },
    */
    printBarcode() {
      console.log("BarCodeRePrintTag, printBarcode(), ", this.temp_barcode_data);

      this.exportToCSV();
      this.$emit('pressPrint', true);
    },

    exportToCSV() {
      console.log("BarCodeRePrintTag, exportToCSV()...");

      let barcode_Desserts = Object.assign([], this.temp_barcode_data);
      console.log("StockInTagPrint, exportToCSV, Axios post data..., ", this.temp_barcode_data, barcode_Desserts);
      // 2023-08-10, modify the following block
      //console.log("barcode_Desserts, temp_barcode_data: ", barcode_Desserts, this.temp_barcode_data);
      /*
      let object_Desserts = this.temp_barcode_data.map(({
        stockInTag_reagID,
        stockInTag_batch,
        stockInTag_Date,
        stockInTag_Employer,
        stockInTag_reagTemp,
        stockInTag_alpha,
        stockInTag_cnt,
        stockInTag_rePrint,
        isIn,
        stockInTag_reagName,
      }) => ({
        stockInTag_reagID,
        stockInTag_batch,
        stockInTag_Date,
        stockInTag_Employer,
        stockInTag_reagTemp,
        stockInTag_alpha,
        stockInTag_cnt,
        stockInTag_rePrint,
        isIn,
        //stockInTag_reagName,
        name1: stockInTag_reagName.length > 25 ? stockInTag_reagName.slice(0,25) : stockInTag_reagName,
        name2: stockInTag_reagName.length > 25 ? stockInTag_reagName.slice(25) : ' '
      }))
      */

      let object_Desserts = Object.assign([], this.temp_barcode_data);
      //console.log("1. object_Desserts: ", object_Desserts);
      ////delete object_Desserts.stockInTag_rePrint
      ////console.log("2. object_Desserts: ", object_Desserts);
      //add name1, name2 key/value
      object_Desserts.forEach((item) => {
        item.name1 = item.stockInTag_reagName.length > 25 ? item.stockInTag_reagName.slice(0,25) : item.stockInTag_reagName;
        item.name2 = item.stockInTag_reagName.length > 25 ? item.stockInTag_reagName.slice(25) : ' '
      })
      //console.log("3. object_Desserts: ", object_Desserts);
      // end block
      console.log("StockInTagPrint, exportToCSV, Axios post data..., ", this.temp_barcode_data, barcode_Desserts, object_Desserts);

      const path = '/exportToCSVForStockInOut';
      var payload= {
        blocks: object_Desserts,
        count: object_Desserts.length,
      };
      axios.post(path, payload)
      .then((res) => {
        console.log("export into csv status: ", res.data.status, res.data.outputs)
        if (res.data.status) {
          this.tosterOK = false;  //false: 關閉錯誤訊息畫面
          //this.snackbar_color='#008184';
          //this.snackbar=true;
          //this.snackbar_info= '庫存記錄('+ res.data.outputs + ')轉檔完成!';
          //this.snackbar_icon_color= "#ffffff";
        } else {
          this.tosterOK = true;   //true: 顯示錯誤訊息畫面
          //this.snackbar_color='red accent-2';
          //this.snackbar=true;
          //this.snackbar_info= '存檔錯誤!';
          //this.snackbar_icon_color= '#adadad';
        }
      })
      .catch((error) => {
        console.error("axios error, code: ",error);
        this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        //this.snackbar_color='red accent-2';
        //this.snackbar=true;
        //this.snackbar_info= '存檔錯誤!';
        //this.snackbar_icon_color= '#adadad';
      });
    },

    startTimer() {
      this.myTimer = setInterval(() => {
        //if (this.svg_change) {
          let bars = document.getElementsByClassName('container_barcode');

          for (let j=0; j<bars.length; j++) {
            let dot = bars[j].childNodes[1].childNodes[0];
            dot.setAttribute("height", "90px");

            this.svg_change=false;
          }
        //}

        let elem = document.querySelector('#layout');
        if(!elem) return;
        let myPos=elem.getBoundingClientRect().top;
        //console.log('calling handleScroll: ', myPos);
        this.fab = myPos <= 10;
      }, 500);

      this.myTimerId = this.myTimer._id;
    },

    onScroll (e) {
      //if (typeof window === 'undefined') return
      //const top = window.pageYOffset || e.target.scrollTop || 0
      //this.fab = top > 20
    },

    toTop () {
      //this.$vuetify.goTo(0)
      document.getElementById("layout").scrollIntoView();
    },
  },
}
</script>

<style scoped>
ul {
  list-style-type: none;
}

li{
  max-width:200px;
  word-wrap:break-word;
  font-size: 14px;
  bottom: -20px;
  position: relative;
  margin-left:20px;
}

container_barcode {
  text-align:center;
}

newSvgAttr {
  height: 80px !important;
}

.word {
  display:table-cell;
  vertical-align:middle;
  /*display: flex;
  justify-content: flex-start;
  float: left;*/
  height: 36px;
}
</style>