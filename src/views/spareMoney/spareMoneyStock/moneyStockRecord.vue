<template>
  <Box>
    <div class="formBox-top">
      <el-form inline ref="formItem" :model="formItem" label-width="110px">
        <el-form-item label="线路：" class="marBottom7">
          <el-select
            class="width200"
            :disabled="disabledBtn.createDis"
            @change="changeLines(formItem.lines)"
            v-model="formItem.lines">
            <el-option
              v-for="(item,index) in linesList"
              :key="index"
              :label="item.LINE_LOCATION_NAME"
              :value="item.LINE_LOCATION_NUMBER.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="车站：" class="marBottom7">
          <el-select
            class="width200"
            :disabled="disabledBtn.createDis"
            clearable
            @change="changeStation(formItem.station)"
            v-model="formItem.station">
            <el-option
              v-for="(item,index) in stationList"
              :key="index"
              :label="item.STATION_LOCATION_NAME"
              :value="item.LOCATION_NUMBER.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作员ID：" class="marBottom7">
          <el-select
            clearable
            class="width200"
            filterable
            v-model="formItem.operationId">
            <el-option
              v-for="(item,index) in operatorIdList"
              :key="index"
              :label="item.USERNAME + '-' + item.USERID"
              :value="item.USERID.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="库存状态：" class="marBottom7">
          <el-select v-model="formItem.status" clearable class="width200">
            <el-option
              v-for="item in statusList"
              :key="item.VALUE"
              :label="item.NAME"
              :value="item.VALUE"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作类型：" class="marBottom7">
          <el-select
            class="width200"
            clearable
            @change="changeType(formItem.type)"
            v-model="formItem.type"
            placeholder="请选择操作类型">
            <el-option
              v-for="(item,index) in typeList"
              :key="index"
              :label="item.TYPE_NAME"
              :value="item.TYPE_VALUE"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作原因：" class="marBottom7">
          <el-select
            class="width200"
            clearable
            v-model="formItem.reason"
            placeholder="请选择变更原因">
            <el-option
              v-for="(item,index) in reasonList"
              :key="index"
              :label="item.REASON_NAME"
              :value="item.REASON_VALUE"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作时间：" class="marBottom7">
          <el-date-picker
            :default-time="['00:00:00', '23:59:59']"
            v-model="formItem.dateTime"
            type="daterange"
            range-separator="至"
            start-placeholder="开始时间"
            end-placeholder="结束时间"/>
        </el-form-item>
        <br>
        <el-form-item class="fr marBottom7">
          <el-button v-if="limitBtn.queryBtn" type="primary" @click="getData">查询</el-button>
          <el-button type="warning" @click="resetFormItem">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="tableWp">
      <el-table :data="tableData" stripe border :default-sort="{prop:'UPDATE_DT',order:'descending'}">
        <el-table-column prop="LINE_NAME" label="线路" header-align="center" align="center"/>
        <el-table-column prop="STATION_NAME" label="车站" header-align="center" align="center"/>
        <el-table-column prop="OPERATE_TYPE" label="操作类型" header-align="center" align="center"/>
        <el-table-column prop="CHANGE_REASON" label="操作原因" header-align="center" align="center"/>
        <el-table-column prop="STOCK_STATE_NAME" sortable label="库存状态" header-align="center" align="center"/>
        <el-table-column prop="OPERATOR_ID" label="操作员ID" header-align="center" align="center">
          <template slot-scope="scope">
            <span>{{(scope.row.IDENTIFIED_OPERATOR_NAME1 ? scope.row.IDENTIFIED_OPERATOR_NAME1 : '') + '-' + (scope.row.IDENTIFIED_OPERATOR_ID1 ? scope.row.IDENTIFIED_OPERATOR_ID1 : '')}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="UPDATE_DT" sortable label="操作时间" header-align="center" align="center"/>
        <el-table-column label="操作" header-align="center" align="center">
          <template slot-scope="scope">
            <el-button type="primary" v-if="limitBtn.detail" @click="getDetail(scope.row)">详情</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页-->
      <pages
        :total='options.total'
        :currentPage='options.currentPage'
        :page-size="options.pageSize"
        @handleSizeChangeSub="handleSizeChangeSub"
        @handleCurrentChangeSub="handleCurrentChange"/>
    </div>
    <!--详情蒙版-->
    <el-dialog
      v-dialogDrag
      append-to-body
      :fullscreen="isScreen"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      :before-close="resetDetail"
      :visible.sync="dialogDetailVisible">
      <div slot="title" class="dialogTitle clearFix">
        <span class="title">操作记录详情</span>
        <i class="el-icon-copy-document" title="放大/缩小" @click="isScreen = !isScreen"></i>
      </div>
      <el-form :model="formDetail" label-width="140px">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="线路：">
              <el-input v-model="formDetail.lines" disabled/>
            </el-form-item>
            <el-form-item label="操作类型：">
              <el-input v-model="formDetail.changeType" disabled/>
            </el-form-item>
            <el-form-item label="操作员：">
              <el-input v-model="formDetail.operator_id" disabled/>
            </el-form-item>
            <el-form-item label="运营日期：">
              <el-input v-model="formDetail.balanceDate" disabled/>
            </el-form-item>
            <el-form-item label="操作前金额(元)：">
              <el-input v-model="formDetail.quantityBefore" disabled/>
            </el-form-item>
            <el-form-item label="操作后金额(元)：">
              <el-input v-model="formDetail.quantityAfter" disabled/>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="车站：">
              <el-input v-model="formDetail.station" disabled/>
            </el-form-item>
            <el-form-item label="操作原因：">
              <el-input v-model="formDetail.changeReason" disabled/>
            </el-form-item>
            <el-form-item label="认证操作员：">
              <el-input v-model="formDetail.check_operator_id" disabled/>
            </el-form-item>
            <el-form-item label="操作时间：">
              <el-input v-model="formDetail.update" disabled/>
            </el-form-item>
            <el-form-item label="操作金额(元)：">
              <el-input v-model="formDetail.quantity" disabled/>
            </el-form-item>
            <el-form-item label="现金状态：">
              <el-input v-model="formDetail.MONEY_STATE" disabled/>
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="备注：">
          <el-input type="textarea" v-model="formDetail.comment_info" disabled/>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button type="primary" @click="resetDetail">关闭</el-button>
      </div>
    </el-dialog>
  </Box>
</template>
<script>
  import Box from '../../../components/Box';
  import {dateFilter} from "../../../assets/js/filters";

  export default {
    name: "moneyStockRecord",
    components: {Box},
    data() {
      return {
        isScreen: false, // 详情放大序缩小
        // 查询表单
        formItem: {
          lines: '', // 线路
          station: '', // 车站
          status: '', // 状态
          type: '', // 操作类型
          reason: '', // 操作原因
          dateTime: [], // 操作时间
          operationId: '', // 操作员id
        },
        // 按钮禁选
        disabledBtn:{
          createDis:true,
        },
        // 表格数据
        tableData: [],
        // 分页
        options: {
          total: 0, // 总条数
          currentPage: 1, // 当前页码
          pageSize: 10, // 显示条数
        },
        // 按钮权限
        limitBtn: {
          queryBtn: false, // 查询
          detail: false, // 详情
        },
        // 库存状态
        statusList: [],
        // 操作类型
        typeList: [],
        // 操作原因
        reasonList: [],
        // 线路集合
        linesList: [],
        // 车站集合
        stationList: [],
        // 操作员id集合
        operatorIdList: [],
        linesListArr:[], // 调入调出线路
        // 详情表单
        formDetail: {
          lines: '', // 线路
          station: '', // 车站
          changeType: '', // 操作类型
          changeReason: '', // 操作原因
          quantityBefore: '', // 操作前数量
          quantity: '', // 操作数量
          quantityAfter: '', // 操作后数量
          operator_id: '', // 操作员
          check_operator_id: '', // 认证操作员
          balanceDate: '', // 运营日期
          update: '', // 操作时间
          comment_info: '', // 备注
          MONEY_STATE: '' // 现金状态
        },
        // 详情蒙版
        dialogDetailVisible: false,
        // 从cookie中获取
        cookieMap: {
          location_number: this.getCookie('LOCATION_NUMBER'), // 位置编号
          user_id: this.getCookie('USER_ID'), // 操作员ID
          role: this.getCookie('USER_TYPE'), // 用户角色
        }
      }
    },
    methods: {
      // 查看详情
      getDetail(row) {
        var _t = this;
        _t.dialogDetailVisible = true;
        _t.formDetail.lines = row.LINE_NAME ? row.LINE_NAME : ''; // 线路
        _t.formDetail.station = row.STATION_NAME ? row.STATION_NAME : ''; // 车站
        _t.formDetail.changeType = row.OPERATE_TYPE ? row.OPERATE_TYPE : ''; // 操作类型
        _t.formDetail.changeReason = row.CHANGE_REASON ? row.CHANGE_REASON : ''; // 操作原因
        _t.formDetail.quantityBefore = row.AMOUNT_BEFORE_YUAN ? row.AMOUNT_BEFORE_YUAN : ''; // 操作前金额
        _t.formDetail.quantity = row.AMOUNT_YUAN ? row.AMOUNT_YUAN : ''; // 操作金额
        _t.formDetail.quantityAfter = row.AMOUNT_AFTER_YUAN ? row.AMOUNT_AFTER_YUAN : ''; // 操作后金额
        _t.formDetail.operator_id = (row.IDENTIFIED_OPERATOR_NAME1 !== undefined ? row.IDENTIFIED_OPERATOR_NAME1 : '') + '-' + (row.IDENTIFIED_OPERATOR_ID1 !== undefined ? row.IDENTIFIED_OPERATOR_ID1 : ''); // 操作员
        _t.formDetail.check_operator_id = (row.IDENTIFIED_OPERATOR_NAME2 !== undefined ? row.IDENTIFIED_OPERATOR_NAME2 : '') + '-' + (row.IDENTIFIED_OPERATOR_ID2 ? row.IDENTIFIED_OPERATOR_ID2 : ''); // 认证操作员
        _t.formDetail.balanceDate = row.BALANCE_DATE ? row.BALANCE_DATE : ''; // 运营日期
        _t.formDetail.update = row.UPDATE_DT ? row.UPDATE_DT : ''; // 操作时间
        _t.formDetail.comment_info = row.COMMENT_INFO ? row.COMMENT_INFO : ''; // 操作时间
        _t.formDetail.MONEY_STATE = row.MONEY_STATE; // 现金状态
      },
      // 重置详情表单
      resetDetail() {
        var _t = this;
        _t.dialogDetailVisible = false;
        _t.formDetail.lines = ''; // 线路
        _t.formDetail.station = ''; // 车站
        _t.formDetail.changeType = ''; // 操作类型
        _t.formDetail.changeReason = ''; // 操作原因
        _t.formDetail.quantityBefore = ''; // 操作前数量
        _t.formDetail.quantity = ''; // 操作数量
        _t.formDetail.quantityAfter = ''; // 操作后数量
        _t.formDetail.operator_id = ''; // 操作员
        _t.formDetail.check_operator_id = ''; // 认证操作员
        _t.formDetail.balanceDate = ''; // 运营日期
        _t.formDetail.update = ''; // 操作时间
      },
      // 重置查询表单
      resetFormItem() {
        var _t = this;
        _t.formItem.status = ''; // 状态
        _t.formItem.type = ''; // 操作类型
        _t.formItem.reason = ''; // 操作原因
        _t.formItem.dateTime = [];// 操作时间
        _t.getData()
      },
      // 获取操作类型及原因
      getTypeAndReason() {
        var _t = this;
        var params = {
          typeid: '02', // 01 车票 02 现金
        };
        _t.$api.post('/tsGateWay/ticketUtility/typeReasonCascadeQuery', params, function (res) {
          if (res.rtCode && res.rtCode === '14002') {
            _t.typeList = res.rtData === null ? [] : res.rtData;
          }
        });
      },
      // 改变操作类型
      changeType(val) {
        var _t = this;
        if (val !== null && val !== '') {
          _t.typeList.forEach((item) => {
            if (item.TYPE_VALUE === val) {
              // 变更原因赋值
              _t.reasonList = item.reasonList === null ? [] : item.reasonList;
            }
            _t.formItem.reason = '';
          });
        } else {
          _t.reasonList = [];
          _t.formItem.reason = '';
        }
      },
      // 改变当前页码
      handleCurrentChange(val) {
        var _t = this;
        _t.options.currentPage = val;
        _t.getData();
      },
      // 改变每页显示条数
      handleSizeChangeSub(val) {
        var _t = this;
        _t.options.pageSize = val;
        _t.getData();
      },
      // 查询按钮
      getData() {
        var _t = this;
        var location_number = ''; // 发送请求的 位置编号
        var location_number_query = ''; // 发送请求的 位置编号
        var operator_id = _t.formItem.operationId; // 操作员id
        // 线路为空 操作员id必为空
        if (_t.formItem.lines === '') {
          location_number = _t.cookieMap.location_number;
          operator_id = _t.cookieMap.user_id;
        } else {
          // 线路不为空
          if (_t.formItem.station === '') {
            // 车站为空 操作员必为空
            location_number = _t.formItem.lines;
            operator_id = _t.cookieMap.user_id;
          } else {
            // // 车站不为空
            // if (_t.formItem.operationId === '') {
            //   // 操作员为空
            //   location_number = _t.formItem.lines;
            //   location_number_query = _t.formItem.station;
            //   operator_id = _t.cookieMap.user_id;
            // } else {
            //   // 操作员不为空
            //   location_number = _t.formItem.lines;
            //   operator_id = _t.formItem.operationId;
            //   location_number_query = _t.formItem.station;
            // }
          }
        }
        // 车站为空 操作员必为空
        if (_t.formItem.station === '') {
          location_number = _t.cookieMap.location_number;
          // operator_id = _t.cookieMap.user_id;
        } else {
          // 车站有值
          location_number = _t.formItem.station;
          // // 操作员为空
          // if (_t.formItem.operationId === '') {
          //   operator_id = _t.cookieMap.user_id;
          // } else {
          //   // 操作员不为空
          //   operator_id = _t.formItem.operationId;
          // }
        }
        if (location_number !== '') {
          let str = 'location_number=' + location_number;
          let Base64 = require('js-base64').Base64;
          let keyToken = _t.$md5(Base64.encode(str));
          //  获取当天时间
          var start_dtime = _t.getCookie('TIME') + ' ' + '00:00:00';
          var end_dtime = _t.getCookie('TIME') + ' ' + '23:59:59';
          let params = {
            location_number: location_number, // 位置编号
            location_number_query: location_number_query === '' ? null : location_number_query,
            operator_id: operator_id, // 操作员id
            token: keyToken, // token
            stock_state: _t.formItem.status === '' ? null : _t.formItem.status, // 库存状态
            operate_type: _t.formItem.type === '' ? null : _t.formItem.type, // 操作类型
            change_reason: _t.formItem.reason === '' ? null : _t.formItem.reason, // 变更原因
            update_dt_begin: _t.formItem.dateTime !== null ? (_t.formItem.dateTime[0] !== undefined ? dateFilter(_t.formItem.dateTime[0].getTime()) : start_dtime) : start_dtime, // 开始时间
            update_dt_end: _t.formItem.dateTime !== null ? (_t.formItem.dateTime[1] !== undefined ? dateFilter(_t.formItem.dateTime[1].getTime()) : end_dtime) : end_dtime, // 结束时间
            pageNum: _t.options.currentPage, // 当前页
            pageSize: _t.options.pageSize, // 每页显示条数
          };
          _t.$api.post('/tsGateWay/readyMoney/stockOperateRecordQueryGateWay', params, function (res) {
            _t.$store.commit('setLoading', false);
            if (res.rtCode && res.rtCode === '11102') {
              // 请求成功
              _t.tableData = res.rtData === null ? [] : res.rtData;
              _t.options.total = res.totalCount ? res.totalCount : 0;
            } else if (res.rtCode) {
              _t.alertDialogTip(_t, res.rtMessage);
            }
          });
        }
      },
      // 获取车票库存状态字典
      getStatusData() {
        var _t = this;
        let str = 'guid=GUID000722201906191607';
        let Base64 = require('js-base64').Base64;
        let keyToken = _t.$md5(Base64.encode(str));
        var params = {
          guid: 'GUID000722201906191607',
          token: keyToken
        };
        _t.$api.post('/tsGateWay/ticketUtility/queryDict', params, function (res) {
          if (res.rtCode && res.rtCode === '14010') {
            _t.statusList = res.rtData === null ? [] : res.rtData;
          }
        });
      },
      // 查询线路及车站
      getLinesAndStation() {
        var _t = this;
        _t.$api.post('/tsGateWay/ticketUtility/locationCascadeQuery', {}, function (res) {
          // 查到线路信息
          if (res.rtCode && res.rtCode === '14007') {
            var linesStationArr = res.rtData === null ? [] : res.rtData;
            // 处理线路/车站/操作员数据
            _t.dealWithLinesAndStation(linesStationArr);
          } else if (res.rtCode) {
            _t.alertDialogTip(_t, res.rtMessage);
          }
        });
      },
      // 处理线路及车站首次加载数据问题
      dealWithLinesAndStation(linesStationArr) {
        var _t = this;
        // 判断角色
        var role = _t.getCookie("USER_TYPE");
        var location = _t.getCookie("LOCATION_NUMBER").toString();
        // 角色和location都不为空
        if (
          role !== null &&
          role !== "null" &&
          location !== null &&
          location !== "null"
        ) {
          // 车站角色
          var linesArr = new Array(); // 线路集合
          var stationArr = new Array(); // 车站集合
          var LINE_LOCATION_NUMBER = ""; // 选中的线路
          var LOCATION_NUMBER = ""; // 选中的车站
          var operationArr = new Array(); // 操作员id集合
          var USERID = ""; // 选中的操作员
          if (location !== '') {
            let lines = linesStationArr [0];
            linesStationArr.forEach(item => {
              if (item.LINE_LOCATION_NUMBER == location) {
                linesArr.push(item);
                item.mlcList.forEach(val => {
                  _t.linesListArr.push(val);
                  linesArr.push(val);
                  val.lineList.forEach(e => {
                    linesArr.push(e);
                    _t.linesListArr.push(e);
                  });
                  LINE_LOCATION_NUMBER = location;
                });
              } else {
                item.mlcList.forEach(lts => {
                  if (lts.LINE_LOCATION_NUMBER == location) {
                    linesArr.push(lts);
                    lts.lineList.forEach(vals => {
                      linesArr.push(vals);
                      _t.linesListArr.push(vals);
                      LINE_LOCATION_NUMBER = location;
                    });
                  } else {
                    if (lts.lineList && lts.lineList !== null) {
                      // 遍历线路
                      lts.lineList.forEach(valStr => {
                        valStr.stationList.forEach(e => {
                          if (e.LOCATION_NUMBER == location) {
                            // 线路选中值
                            LINE_LOCATION_NUMBER = valStr.LINE_LOCATION_NUMBER.toString();
                            // 车站选中值
                            LOCATION_NUMBER = location;
                            // 线路下拉框
                            linesArr.push(valStr);
                            // 找到了对应的车站
                            stationArr.push(e);
                            operationArr = e.userList;
                            operationArr.forEach(data => {
                              if (data.USERID === _t.getCookie("USER_ID")) {
                                USERID = data.USERID;
                              }
                            });
                          }
                        });
                      });
                    }
                  }
                })
              }
              // 线路存在车站

            });
            _t.linesList = linesArr;
            _t.stationList = stationArr;
            _t.operatorIdList = operationArr;
            // 选中的线路
            if (LINE_LOCATION_NUMBER !== "") {
              _t.formItem.lines = LINE_LOCATION_NUMBER;
            }
            // 选中的操作员
            if (USERID !== "") {
              _t.formItem.operationId = USERID;
            }
            // 选中的车站
            if (LOCATION_NUMBER !== "") {
              _t.formItem.station = LOCATION_NUMBER;
            }
          }
        }
        // 线路接口调完之后调用查询接口
        _t.getData();
      },
      // 改变线路
      changeLines(val) {
        var _t = this;
        if (val !== '') {
          _t.linesList.forEach((item) => {
            // 找到对应线路
            if (item.LINE_LOCATION_NUMBER.toString() === val) {
              // 存在对应的车站集合
              if (item.stationList !== undefined && item.stationList !== null) {
                // 车站下拉框赋值
                _t.stationList = item.stationList;
              } else {
                // 车站下拉框置空
                _t.stationList = [];
              }
              // 选中车站置空
              _t.formItem.station = '';
              // 操作员下拉框置空
              _t.operatorIdList = [];
              // 操作员id置空
              _t.formItem.operationId = '';
            }
          });
        } else {
          // 选中线路为空
          _t.stationList = [];
          _t.formItem.station = '';
          _t.operatorIdList = [];
          _t.formItem.operationId = '';
        }
      },
      // 改变车站
      changeStation(val) {
        var _t = this;
        if (val !== '') {
          // 遍历车站集合
          _t.stationList.forEach((item) => {
            // 找到车站
            if (item.LOCATION_NUMBER.toString() === val) {
              // 存在操作员集合
              if (item.userList !== undefined && item.userList !== null) {
                // 操作员集合赋值
                _t.operatorIdList = item.userList;
                // 遍历操作员集合
                _t.operatorIdList.forEach((data) => {
                  // 找到自身
                  if (data.USERID === _t.cookieMap.user_id) {
                    _t.formItem.operationId = data.USERID;
                  } else {
                    // 置为空
                    _t.formItem.operationId = '';
                  }
                });
              } else {
                // 对应车站没有操作员集合
                _t.operatorIdList = [];
                _t.formItem.operationId = '';
              }
            }
          });
        } else {
          // 车站改变值为空
          _t.operatorIdList = [];
          _t.formItem.operationId = '';
        }
      }
    },
    created() {
      var _t = this;
      if ( _t.getCookie('LOCATION_TYPE_ID') !== '09') {
        _t.disabledBtn.createDis = false;
      }
      _t.$store.commit('setLoading', true);
      var limitMenuList = JSON.parse(localStorage.getItem('limitMenuList'));
      if (limitMenuList !== null && limitMenuList.length !== 0) {
        var limitBtnArr = new Array();
        limitMenuList.forEach((item) => {
          var path = _t.$route.path;
          // 找到了对应权限的页面
          if (item.URL === path) {
            // 获取按钮级别权限
            if (item.CHILDREN !== null) {
              limitBtnArr = item.CHILDREN
            }
            // 本页面业务逻辑
            _t.getStatusData();
            _t.getLinesAndStation();
            _t.getTypeAndReason();
          }
        });
        // 判断权限按钮
        limitBtnArr.forEach((item) => {
          // 查询
          if (item.MENU_ID === "GUID00000000000000000000000000000051") {
            _t.limitBtn.queryBtn = true;
          }
          // 详情按钮
          if (item.MENU_ID === 'MENU20190917145107qfLfOSRyxTzWMU') {
            _t.limitBtn.detail = true;
          }
        });
      }
    }
  }

</script>
<style scoped>

</style>
