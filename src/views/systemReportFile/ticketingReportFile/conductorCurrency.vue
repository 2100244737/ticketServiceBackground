<template>
  <div>
    <div id="div" class="clearFix formBox-top">
      <!--       头部查询部分 -->
      <el-form :model="formItem" ref="formData" :rules="rules2"  label-width="120px" :inline="true">
        <el-form-item label="线路：" class="star marBottom7">
          <el-select
            :disabled="disabledBtn.create"
            @change="changeLines(formItem.lines)"
            class="width200"
            v-model="formItem.lines">
            <el-option
              v-for="(item,index) in linesList"
              :key="index"
              :label="item.LINE_LOCATION_NAME"
              :value="item.STATION_ID.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="车站：" prop="station"  class="star marBottom7">
          <el-select
            clearable
            :disabled="disabledBtn.create"
            class="width200"
            @change="changeStation(formItem.station)"
            v-model="formItem.station">
            <el-option
              v-for="(item,index) in stationList"
              :key="index"
              :label="item.STATION_LOCATION_NAME"
              :value="item.STATION_ID.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item  label="售票员ID：" prop="operationId" class="marBottom7">
          <el-select
            clearable
            class="width200"
            v-model="formItem.operationId">
            <el-option
              v-for="(item,index) in operatorIdList"
              :key="index"
              :label="item.USERNAME + '-' + item.USERID"
              :value="item.USERID.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作时间：" prop="dateTime" class=" star marBottom7">
          <el-date-picker
            value-format="yyyyMMdd"
            v-model="formItem.dateTime"
            type="date"
          />
        </el-form-item>
        <br/>
        <el-form-item class="fr marBottom7">
          <el-button type="primary" @click="getData">查询</el-button>
          <el-button type="warning" @click="resetQuery">重置</el-button>
        </el-form-item>
      </el-form>
    </div>

    <div class="tableWp">
      <iframe
        v-show="isShow"
        :src="formIframeSrc"
        scrolling="auto" frameborder="0" id="iframepage" class="frame">
      </iframe>
    </div>
  </div>
</template>

<script>
export default {
  name: 'conductorCurrency',
  data() {
    return {
      // 查询表单
      formItem: {
        operationId: '', // 售票员id
        lines: "", // 线路
        station: "", // 车站
        dateTime:'', // 操作时间
        // interval: '', // 时间间隔
        type: '' // 设备类型
      },
      operatorIdList: [],
      disabledBtn: {
        create: true
      },
      rules2: {
        type: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
        station: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
        dateTime: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
        operationId: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
      },
      // 校验标识
      errorTip: {
        lineFlag: false, // 线路
        stationFlag: false, // 车站
        timeFlag: false, // 时间
        typeFlag: false, // 类型
      },
      isShow: false, // 嵌入网页是否显示
      // 线路集合
      linesList: [],
      // 车站集合
      stationList: [],
      statusList: [], // 设备类型
      formIframeSrc: '', // 网页路径 ,
      // 从cookie中获取
      cookieMap: {
        location_number: this.getCookie('LOCATION_NUMBER'), // 位置编号
        user_id: this.getCookie('USER_ID'), // 操作员ID
        role: this.getCookie('USER_TYPE'), // 用户角色
      }
    }
  },
  methods: {
    // 查询按钮
    getData() {
      var _t = this;
      _t.$refs.formData.validate((val) => {
        if (val) {
          _t.getHeight();
          if (1) {
            // 嵌入网页是否显示
            _t.isShow = true;
            _t.formIframeSrc = _t.$apiForms +'/tbIetp-reportForm/ReportEmitter?rpt=/票务售票员结算单.brt&params=p_station_id=' + _t.formItem.station + ';p_operator_id='+_t.formItem.operationId +';p_date=' + _t.formItem.dateTime  + ';&grabheight=true&fitwidth=true';
          }
        }
      });

    },
    // 获取高度
    getHeight () {
      var ifm= document.getElementById("iframepage");
      var div= document.getElementById("div");
      ifm.height=document.documentElement.clientHeight -  div.offsetHeight*1.7;
    },
    // 查询线路及车站
    getLinesAndStation() {
      var _t = this;
      _t.$api.post("/tsGateWay/ticketUtility/locationCascadeQuery", {}, function (res) {
          // 查到线路信息
          if (res.rtCode && res.rtCode === "14007") {
            var linesStationArr = res.rtData === null ? [] : res.rtData;
            // 处理线路/车站/操作员数据
            _t.dealWithLinesAndStation(linesStationArr);
          } else if (res.rtCode) {
            _t.alertDialogTip(_t, res.rtMessage);
          }
        }
      );
    },
    // 处理线路及车站首次加载数据问题
    dealWithLinesAndStation(linesStationArr) {
      var _t = this;
      if (_t.getCookie('LOCATION_TYPE_ID') !== '09') {
        _t.disabledBtn.create = false;
      }
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
        var USERID = ''; // 售票员id
        if (location !== '') {
          let lines = linesStationArr [0];
          linesStationArr.forEach(item => {
            if (item.LINE_LOCATION_NUMBER == location) {
              linesArr.push(item);
              item.mlcList.forEach(val => {
                val.lineList.forEach(vals => {
                  if (vals.LINE_LOCATION_NUMBER.toString() == '285212673') {
                    // 线路选中值
                    LINE_LOCATION_NUMBER = vals.STATION_ID.toString();
                    //  线路下拉框
                    linesArr = val.lineList;
                    stationArr = vals.stationList;
                  }
                });
                // LINE_LOCATION_NUMBER = item.STATION_ID.toString();
              });
            } else {
              item.mlcList.forEach(lts => {
                if (lts.LINE_LOCATION_NUMBER == location) {
                  _t.disabledBtn.create = false
                  lts.lineList.forEach(val => {
                    if (val.LINE_LOCATION_NUMBER.toString() == '285212673') {
                      // 线路选中值
                      LINE_LOCATION_NUMBER = val.STATION_ID.toString();
                      //  线路下拉框
                      linesArr = lts.lineList;
                      stationArr = val.stationList;
                    }
                  });
                } else {
                  if (lts.lineList && lts.lineList !== null) {
                    // 遍历线路
                    lts.lineList.forEach(val => {
                      val.stationList.forEach(e => {
                        if (e.LOCATION_NUMBER == location) {
                          // 线路选中值
                          LINE_LOCATION_NUMBER = val.STATION_ID.toString();
                          // 车站选中值
                          LOCATION_NUMBER = e.STATION_ID;
                          // 售票员id

                          _t.operatorIdList = e.userList;
                          e.userList.forEach(oper => {
                            if (oper.USERID === _t.getCookie("USER_ID")) {
                              USERID = oper.USERID;
                            }
                          });
                          // 线路下拉框
                          linesArr = lts.lineList;
                          // 找到了对应的车站
                          stationArr = val.stationList;
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
      //_t.getData();
    },
    // 改变线路
    changeLines(val) {
      var _t = this;
      if (val !== "") {
        _t.linesList.forEach(item => {
          // 找到对应线路
          if (item.STATION_ID.toString() === val) {
            // 存在对应的车站集合
            if (item.stationList !== undefined && item.stationList !== null) {
              // 车站下拉框赋值
              _t.stationList = item.stationList;
            } else {
              // 车站下拉框置空
              _t.stationList = [];
            }
            // 选中车站置空
            _t.formItem.station = "";
            // 操作员下拉框置空
            _t.operatorIdList = [];
            // 操作员id置空
            _t.formItem.operationId = "";
          }
        });
      } else {
        // 选中线路为空
        _t.stationList = [];
        _t.formItem.station = "";
        _t.operatorIdList = [];
        _t.formItem.operationId = "";
      }
    },

    // 改变车站
    changeStation(val) {
      var _t = this;
      if (val !== '') {
        // 遍历车站集合
        _t.stationList.forEach((item) => {
          // 找到车站
          if (item.STATION_ID.toString() === val) {
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
    },
    // 重置按钮
    resetQuery() {
      // 查询表单
      this.formItem.dateTime = '';// 时间间隔
      this.formItem.type = '';// 设备类型
    },
  },
  created() {
    this.getLinesAndStation()
  }
}
</script>

<style scoped>
  .frame {
    width: 100%;
  }
  /deep/.el-date-editor .el-range-separator {
    width: 7%;
  }
</style>
