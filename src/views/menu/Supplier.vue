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
        <v-card width="88vw" class="pa-md-4  mt-5 mx-lg-auto">
          <v-data-table
            :headers="headers"
            :items="desserts"
            class="elevation-1"
            :item-class="setRowStyle"
            :options.sync="pagination"
            :footer-props="{ itemsPerPageText: '每頁的資料筆數' }"
          >
            <template v-slot:top>
              <v-toolbar flat>
                <v-toolbar-title>供應商資料</v-toolbar-title>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="900px" content-class="add_modalbox">
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on" v-show="currentUser.perm<=2">
                      <v-icon left dark>mdi-table-plus</v-icon>
                      新增資料
                    </v-btn>
                  </template>
                  <v-card>
                    <v-card-title>
                      <span class="text-h5">{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                      <v-container>
                        <!-- 第1列-->
                        <v-row>
                          <v-col cols="12" md="3">
                            <v-text-field
                              v-model="editedItem.sup_id"
                              label="供應商代碼"

                              :readonly="formTitle === '編輯資料'"
                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text= "IDErrMsg"></small>
                          </v-col>
                          <v-col cols="12" md="6">
                            <v-text-field
                              v-model="editedItem.sup_name"
                              type="text"
                              label="供應商名稱"

                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text="supNameErrMsg"></small>
                          </v-col>
                          <v-col cols="12" md="3">
                          </v-col>
                        </v-row>
                        <!-- 第2列-->
                        <v-row>
                          <!--
                          <v-col cols="12" md="3">
                            <v-text-field
                              v-model="editedItem.sup_name"
                              type="text"
                              label="供應商名稱"
                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text="supNameErrMsg"></small>
                          </v-col>
                          -->

                          <v-col cols="12" md="3">
                            <v-text-field
                              label="聯絡人"
                              type="text"
                              prepend-icon="mdi-card-account-mail-outline"
                              v-model="editedItem.sup_contact"
                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text="supContactErrMsg"></small>
                          </v-col>
                          <v-col cols="12" md="6">
                            <v-text-field
                              v-model="editedItem.sup_address"
                              type="text"
                              label="聯絡地址"
                              prepend-icon="mdi-map-marker-outline"
                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text="supAddressErrMsg"></small>
                          </v-col>

                          <v-col cols="12" md="3">
                            <v-text-field
                              label="電話"
                              type="text"
                              prepend-icon="mdi-phone"
                              v-model="editedItem.sup_phone"
                              @focus="fieldFocus"
                            />
                            <small class="msgErr" v-text="supPhoneErrMsg"></small>
                          </v-col>
                        </v-row>
<!-- 第3列-->
                        <v-row>
                          <v-col cols="12">
                            <!--prepend-icon="mdi-format-list-bulleted-square"-->
                            <v-combobox
                              v-model="temp_product"
                              :items="items"
                              chips
                              clearable

                              label="主要產品"

                              multiple
                              solo
                            >
                              <template v-slot:selection="{ attrs, item, select, selected }">
                                <v-chip
                                  v-bind="attrs"
                                  :input-value="selected"
                                  close
                                  label
                                  outlined
                                  color="blue"
                                  @click="select"
                                  @click:close="removeChip(item)"
                                >
                                  {{ item }}
                                </v-chip>
                              </template>
                            </v-combobox>
                          </v-col>
                        </v-row>
                      </v-container>
                    </v-card-text>

                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="close">取消</v-btn>
                      <!--<v-btn v-bind:disabled="disbtn" color="blue darken-1" text @click="save">儲存</v-btn>-->
                      <v-btn color="blue darken-1" text @click="save" :disabled='checkDataForSaveButton'>確定</v-btn>

                      <v-tooltip v-model="tosterOK" bottom :position-x="toster_pos_x" :position-y="toster_pos_y" color="error">
                        <span>資材碼重複!</span>
                      </v-tooltip>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="500px">
                  <v-card>
                    <v-card-title class="text-h5">確定刪除這筆資料?</v-card-title>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="closeDelete">取消</v-btn>
                      <v-btn color="blue darken-1" text @click="deleteItemConfirm">刪除</v-btn>
                      <v-spacer></v-spacer>
                    </v-card-actions>
                  </v-card>
                </v-dialog>

                <v-dialog v-show="rightDialog"
                  v-model="rightDialog"
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
                      <v-btn text @click="rightCloseFun"> 取消 </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>

              </v-toolbar>
            </template>

            <template v-slot:item.sup_products="{ item }">
              <v-chip class="ma-2" small v-for="(pt, index) in item.sup_products" :key="index">
                {{ pt }}
              </v-chip>
            </template>

            <template v-slot:[`item.actions`]="{ item }">
              <v-icon class="mr-2" @click="editItem(item)" style="color: blue;" size="16">
                mdi-pencil
              </v-icon>
              <v-icon @click="deleteItem(item)" style="color: red;" size="16">
                mdi-delete
              </v-icon>
            </template>
            <!--
            <template v-slot:no-data>
              <v-btn color="primary" @click="initialize">Reset</v-btn>
            </template>
            -->

            <template v-slot:no-data>
              <strong><font color='red'>目前沒有資料</font></strong>
            </template>

          </v-data-table>
        </v-card>
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
import Common from '../../mixin/common.js'

export default {
  name: 'Supplier',

  mixins: [Common],

  mounted() {
    // if back button is pressed
    window.onpopstate = () => {
      console.log("press back button, good bye...");

      const userData = JSON.parse(localStorage.getItem('loginedUser'));
      userData.setting_items_per_page = this.pagination.itemsPerPage;
      localStorage.setItem('loginedUser', JSON.stringify(userData));
    };
  },

  data: () => ({
    currentUser: {},

    //chips: ['Microscan細菌鑑定試劑', '基因檢測試劑', '離心機'],
    /*
    items: ['基因檢測試劑', '核酸萃取試劑', '離心機',
            'C13檢測試劑', '能力試驗', '教育訓練','抗血清試劑',
            '血液諮詢', 'Microscan細菌鑑定試劑', '台塑生醫EV71-IgM(rapid-tset)'
    ],
    */
    items: [],
    temp_items: [],

    temp_product: [],
    permDialog: false,
    rightDialog: false,

    snackbar: false,
    snackbar_color: 'success',
    snackbar_right: true,
    snackbar_top: true,
    snackbar_info: '',
    snackbar_icon_color: '#adadad',

    dialog: false,
    dialogDelete: false,

    //disbtn: false,

    pagination: {
      //itemsPerPage: 10,   //預設值, rows/per page
      //page: 1,
    },

    tosterOK: false,
    toster_pos_x: 1000,
    toster_pos_y: 400,
    toster_delay:3,

    IDErrMsg: '',
    supNameErrMsg: '',
    supContactErrMsg: '',
    supPhoneErrMsg: '',
    supAddressErrMsg: '',

    //資料表頭
    headers: [
      //{ text: 'ID', sortable: false, value: 'id', width: '10%', align: 'start'},

      { text: '公司代號', sortable: false, value: 'sup_id', width: '8%', align: 'left' },
      { text: '公司名稱', sortable: true, value: 'sup_name', width: '17%', align: 'left' },
      { text: '聯絡地址', sortable: false, value: 'sup_address', width: '220px', align: 'left' },
      { text: '聯絡人', sortable: false, value: 'sup_contact', width: '70px', align: 'left' },
      { text: '電話', sortable: false, value: 'sup_phone', width: '100px', align: 'left' },
      { text: '主要產品', sortable: false, value: 'sup_products', width: '380px', align: 'center' },
      { text: 'Actions', sortable: false, value: 'actions', width: '80px' },
    ],

    desserts: [],
    temp_desserts: [],

    editedIndex: -1,
    editedItem: {
      //id: 0,
      sup_id: '',
      sup_name: '',
      sup_address: '',
      sup_contact: '',
      sup_phone: '',
      sup_products: []
    },
    defaultItem: {
      sup_id: '',
      sup_name: '',
      sup_address: '',
      sup_contact: '',
      sup_phone: '',
      sup_products: []
    },

    load_SingleTable_ok: false,
    load_2thTable_ok: false,
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? '新增資料' : '編輯資料'
    },

    checkDataForSaveButton() {
      if (!!this.editedItem.sup_id && !!this.editedItem.sup_name && !!this.editedItem.sup_contact &&
          // !!this.editedItem.sup_address && !!this.editedItem.sup_phone &&
          this.temp_product.length >0 &&
          this.IDErrMsg == '' && this.supNameErrMsg == '' && this.supContactErrMsg == '' &&
          this.supAddressErrMsg == '' && this.supPhoneErrMsg == '') {
        return false;
      } else {
        return true
      }
    }
  },

  watch: {
    dialog(val) {
      !val || this.fieldFocus();
      val || this.close();
    },

    dialogDelete(val) {
      val || this.closeDelete()
    },

    'editedItem.sup_id': function () {
      //let isRule = /^\d{4}$/;
      //let result = this.editedItem.sup_id.search(isRule);
      //console.log("this.editedItem.sup_id, typeof ", typeof(this.editedItem.sup_id));
      //let pattern1 =/^\d+$/
      let pattern1 = /^[A-Za-z0-9]{1,6}$/;  // 2023-04-18 modify LENGTH FROM 4 into 6

      let isNumber = pattern1.test(this.editedItem.sup_id);
      //let pattern2 =/^\s+$/
      //let isSpace = pattern2.test(this.editedItem.sup_id);
      //console.log("is: ",isNumber, isSpace, this.editedItem.sup_id.length)
      let matchResult = this.desserts.find(x => x.sup_id === this.editedItem.sup_id);

      this.IDErrMsg = '';
      //if (result == -1 && !(isNumber && this.editedItem.sup_id.length<5 && this.editedItem.sup_id.length>=0)) {
      // 2023-04-18 modify LENGTH FROM 4 into 6
      if (!(isNumber && this.editedItem.sup_id.length<7 && this.editedItem.sup_id.length>0)) {
        if (this.editedItem.sup_id.length!=0)
          this.IDErrMsg = '資料格式錯誤!';
      } else {
        if (typeof(matchResult) != 'undefined' && this.editedIndex == -1) { // 2023-04-18 modify
          this.IDErrMsg = '供應商代碼與 ' + matchResult.sup_name + ' 重複!';
        } else {
          this.IDErrMsg = '';
        }
      }
    },
    //2023-04-18 MODIFY
    'editedItem.sup_name': function () {
      let len=this.editedItem.sup_name.length
     /*
      let isRule = /^([a-zA-Z\u4e00-\u9fa5_\-.]+)$/;
      let result = this.editedItem.sup_name.search(isRule);
      console.log("tt result: ", result)
      let len=this.editedItem.sup_name.length
      result = (len==0)?0:result
      //if (len==0)
      //  result=0
      this.supNameErrMsg = '';
      //console.log("watch:", result, len)
      if (this.editedIndex ==-1) {
        this.supNameErrMsg = '';
        return;
      }
      */

      //if (result == -1 || len > 40) {
      if (len > 40) {
        this.supNameErrMsg = '資料格式錯誤!';
      } else {
        this.supNameErrMsg = '';
      }
    },

    //2023-04-18 MODIFY
    'editedItem.sup_contact': function () {
      let len = this.editedItem.sup_contact.length;
      /*
      let isRule = /^([a-zA-Z\u4e00-\u9fa5_.-]+)$/;
      let result = this.editedItem.sup_contact.search(isRule);
      result = (len==0)?0:result
      //if (len==0)
      //  result=0
      this.supContactErrMsg = '';
      //console.log("watch:", result, len)
      */
      //if (result == -1 || len > 10) {
      if (len > 10) {
        this.supContactErrMsg = '資料格式錯誤!';
      } else {
        this.supContactErrMsg = '';
      }
    },

    'editedItem.sup_phone': function () {
      let isRuleInt = /^(0[2-8]{1,2}-?|\(0[2-8]{1,2}\))\d{3,4}-?\d{4}|09\d{2}(\d{6}|-\d{3}-\d{3})$/
      let result = this.editedItem.sup_phone.search(isRuleInt);
      let myString = this.editedItem.sup_phone.replace(/\D/g,'');
      //console.log("myString: ", myString)
      this.supPhoneErrMsg = '';
      if (result == -1 || myString.length>10) {
        this.supPhoneErrMsg = '資料格式錯誤!';
      } else {
        this.supPhoneErrMsg = '';
      }
    },

    'editedItem.sup_address': function () {
      //let isRuleInt = /^(?['縣'|'市'])(?['鄉','鎮','市','區'])$/;
      //let result = this.editedItem.sup_address.search(isRuleInt);
      //console.log("address regex result: ", result);
      this.supAddressErrMsg = '';
      if (this.editedItem.sup_address.length > 80) {
        this.supAddressErrMsg = '資料格式錯誤!';
      } else {
        this.supAddressErrMsg = '';
      }
    },

    load_SingleTable_ok(val) {
      if (val) {
        this.desserts = Object.assign([], this.temp_desserts);
        this.load_SingleTable_ok=false;
      }
    },

    load_2thTable_ok(val) {
      if (val) {
        this.items = Object.assign([], this.temp_items);
        this.load_2thTable_ok=false;

        //this.load_SingleTable_ok=false;
        this.listSuppliers();
      }
    },

  },

  created() {
    this.currentUser = JSON.parse(localStorage.getItem("loginedUser"));
    if (this.currentUser.perm == 0) {
      this.permDialog=true;
    }

    this.pagination.itemsPerPage=this.currentUser.setting_items_per_page

    //this.load_SingleTable_ok=false;
    this.load_2thTable_ok=false;
    this.initAxios();
    this.listProducts();
    //this.initialize()
  },

  methods: {
    initialize() {
      this.load_2thTable_ok=false;
      this.listProducts();
    },

    listProducts() {
      const path = '/listProducts';
      console.log("listProducts, Axios get data...")
      axios.get(path)
      .then((res) => {
        this.temp_items = res.data.outputs;
        console.log("GET ok, total records:", res.data.outputs.length);
        this.load_2thTable_ok=true;
      })
      .catch((error) => {
        console.error(error);
        this.load_2thTable_ok=false;
      });
    },

    listSuppliers() {
      const path = '/listSuppliers';
      console.log("listSuppliers, Axios get data...")
      axios.get(path)
      .then((res) => {
        this.temp_desserts = res.data.outputs;
        console.log("GET ok, total records:", res.data.outputs.length);
        this.load_SingleTable_ok=true;
      })
      .catch((error) => {
        console.error(error);
        this.load_SingleTable_ok=false;
      });
    },

    setRowStyle(item) {
      return 'style-1';
    },

    fieldFocus() {
      this.IDErrMsg = '';
      this.supNameErrMsg = '';
      this.supContactErrMsg = '';
      this.supPhoneErrMsg = '';
      this.supAddressErrMsg = '';
    },

    editItem(item) {
      if (this.currentUser.perm >2) {
        this.rightDialog = true;
        return;
      }

      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      console.log("editItem, item: ", item, this.temp_product);

      this.temp_product=[];
      for (var i = 0; i < this.editedItem.sup_products.length; i++) {
        let tempS=this.items[this.editedItem.sup_products[i]-1];
        console.log("list0: ", this.editedItem.sup_products[i]);
        console.log("list1: ", this.editedItem.sup_products[i]-1);
        console.log("list2: ", this.items[this.editedItem.sup_products[i]-1]);
        //this.temp_product.push(this.items[this.editedItem.sup_products[i]-1]);
        //this.temp_product.push(this.editedItem.sup_products[i]);
        if (this.editedIndex==-1)
          this.temp_product.push(tempS);
        else
          this.temp_product.push(this.editedItem.sup_products[i]);

      }
      console.log("list3: ", this.temp_product);

      this.dialog = true;
    },

    deleteItem(item) {
      if (this.currentUser.perm >2) {
        this.rightDialog = true;
        return;
      }

      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.removeSupplier(this.editedItem.sup_id);

      if (!this.tosterOK) {
        this.desserts.splice(this.editedIndex, 1)
        console.log("deleteItem: ", this.editedItem);

        this.closeDelete()
      }
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })

      this.temp_product=[];
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })

      this.temp_product=[];
    },

    removeSupplier(id) {  //依reagent id來刪除後端table資料
      let path='/removeSupplier';
      let payload= {
        ID: id,
      };
      axios.post(path, payload)
      .then(res => {
        console.log("remove supplier status: ", res.data.status);
        if (res.data.status) {
          this.tosterOK = false;  //false: 關閉錯誤訊息畫面
          this.editedItem = Object.assign({}, this.defaultItem);
        } else {
          this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        }
      })
      .catch(err => {
        console.error(err)
        this.tosterOK = true;   //true: 顯示錯誤訊息畫面
      });
    },

    save() {
      console.log("click save button, editedIndex: ", this.editedIndex);

      this.editedItem.sup_id=this.editedItem.sup_id.trim();     //2023-05-13 add

      this.editedItem.sup_products=[];
      let temp_len=this.temp_product.length
      for (var i = 0; i < temp_len; i++) {
        let tt = this.items.findIndex((element)=>{
          return element === this.temp_product[i];
        })
        this.editedItem.sup_products.push(tt+1);
      }
      console.log("this.editedItem", this.editedItem, this.temp_product);
      //this.editedItem.sup_products=[];
      this.editedItem.sup_products = Object.assign([], this.temp_product);

      if (this.editedIndex == -1) {    //add
        this.createSupplier(this.editedItem);
        if (!this.tosterOK) {
          this.desserts.push(this.editedItem);
          this.close();
        }
      } else {    //edit
        this.updateSupplier(this.editedItem);
        if (!this.tosterOK) {
          Object.assign(this.desserts[this.editedIndex], this.editedItem)
          this.close();
        }
      }
    },

    removeChip(item) {
      this.temp_product.splice(this.temp_product.indexOf(item), 1);
      //this.chips.splice(this.chips.indexOf(item), 1);
      //this.chips = [...this.chips];
      this.temp_product = [...this.temp_product];
    },

    updateSupplier(object) {  //編輯 supplier後端table資料
      console.log("---click update_supplier data---", object);

      const path='/updateSupplier';
      let payload = Object.assign({}, object);
      axios.post(path, payload)
      .then(res => {
        console.log("update supplier data status: ", res.data.status)
        if (res.data.status) {
          this.tosterOK = false;  //false: 關閉錯誤訊息畫面
          this.editedItem = Object.assign({}, this.defaultItem)
        } else {
          this.tosterOK = true;   //true: 顯示錯誤訊息畫面
        }
      })
      .catch(err => {
        console.error(err);
        this.tosterOK = true;   //true: 顯示錯誤訊息畫面
      });

    },

    createSupplier(object) {  //新增 supplier後端table資料
      console.log("---click create_supplier data---");
      console.log("supplier object: ", object);

      const path='/createSupplier';
      let payload = Object.assign({}, object);
      axios.post(path, payload)
      .then(res => {
        console.log("save supplier data status: ", res.data.status)

        if (res.data.status) {
          this.tosterOK = false;  //false: 關閉錯誤訊息畫面
          this.editedItem = Object.assign({}, this.defaultItem)
        } else {
          this.tosterOK = true;   //true: 顯示錯誤訊息畫面
          this.snackbar_color='red accent-2';
          this.snackbar=true;
          this.snackbar_info= '錯誤! 供應商資料重複!';
          this.snackbar_icon_color= '#adadad';
        }
      })
      .catch(err => {
        console.error(err);
      });
    },

    permCloseFun () {
      this.permDialog = false
      console.log("press permission Close Button...");
      this.$router.push('/navbar');
    },
    rightCloseFun() {
      this.rightDialog = false
      console.log("press permission Close Button...");
      //this.$router.push('/navbar');
    },
  },
}
</script>

<style lang="scss" scoped>
@import url('https://fonts.googleapis.com/css?family=Noto+Sans+TC:400,500&display=swap&subset=chinese-traditional'
);

div.v-toolbar__title {
  margin: 0;
  font-family: "Noto Sans TC", "Microsoft Yahei", "微軟雅黑", sans-serif;
}

::v-deep .v-data-table-header {
  background-color: #7DA79D;
}

::v-deep .v-data-table-header th {
  font-size: 1em !important;
}

::v-deep .v-data-table-header th:nth-last-child(1) {
  font-size: 0.8em !important;
}

small.msgErr {
  font-size: 80%;
  color: red;
  top: -20px;
  position: relative;
}

::v-deep .style-1 td {
  padding-left: 8px !important;
  padding-right: 0px !important;
}

::v-deep .v-data-table > .v-data-table__wrapper > table > thead > tr > th {
  padding-left: 8px !important;
  padding-right: 0px !important;
  text-align: start !important;
}

::v-deep .v-data-table > .v-data-table__wrapper > table > thead > tr > th:nth-last-child(2) {
  text-align: center !important;
}

::v-deep .v-data-table-header th:nth-last-child(1) span {
  color: #1f4788 !important;
}

::v-deep .style-1 td:nth-last-child(2) > span {
  margin-left: 4px !important;
  margin-right: 0px !important;
  margin-top: 4px !important;
  margin-bottom: 4px !important;
  border-style: solid !important;
  border-color: coral !important;
}

::v-deep .style-1 td:nth-child(2) {
  font-size: 10px !important;
}
::v-deep .style-1 td:nth-child(4) {
  font-size: 10px !important;
}

/*
::v-deep .add_modalbox > .v-card {
    background: rgba(170, 209, 183, 0.37);
    border-radius: 16px;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(4.8px);
    -webkit-backdrop-filter: blur(4.8px);
    border: 1px solid rgba(170, 209, 183, 1);
}

::v-deep div[role=combobox].v-input__slot {
    background: rgba(170, 209, 183, 0.37) !important;
}

::v-deep div[role=listbox].v-list.v-select-list {
  background: rgba(170, 209, 183, 1) !important;
}
*/
</style>