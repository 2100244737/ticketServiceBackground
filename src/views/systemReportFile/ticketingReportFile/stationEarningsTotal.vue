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
        <el-form-item label="车站：" prop="station" class="marBottom7">
          <el-select
            clearable
            :disabled="disabledBtn.create"
            class="width200"
            v-model="formItem.station">
            <el-option
              v-for="(item,index) in stationList"
              :key="index"
              :label="item.STATION_LOCATION_NAME"
              :value="item.STATION_ID.toString()"/>
          </el-select>
        </el-form-item>
        <el-form-item label="操作时间：" prop="dateTime" class=" star marBottom7">
          <el-date-picker
            value-format="yyyyMMdd"
            v-model="formItem.dateTime"
            type="daterange"
            range-separator="至"
            start-placeholder="开始时间"
            end-placeholder="结束时间"
          />
        </el-form-item>
        <br/>
        <el-form-item class="fr marBottom7">
          <el-button type="primary" @click="getData">查询</el-button>
          <el-button type="warning" @click="resetQuery">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 头部查询部分 -->
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
  name: 'stationEarningsTotal',
  data() {
    return {
      // 查询表单
      formItem: {
        lines: "", // 线路
        station: "", // 车站
        dateTime:[], // 操作时间
        // interval: '', // 时间间隔
        type: '' // 设备类型
      },

      disabledBtn: {
        create: true
      },
      rules2: {
        type: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
        station: [{type:'string', required: true, message: '不能为空', trigger:'change'}],
        dateTime: [{type:'array', required: true, message: '不能为空', trigger:'change'}],
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
            _t.formIframeSrc = _t.$apiForms +'/tbIetp-reportForm/ReportEmitter?rpt=/票务车站收益汇总表.brt&params=time2=' + _t.formItem.dateTime[1] +';p_station_id='+_t.formItem.station+ ';time1=' + _t.formItem.dateTime[0] + ';&grabheight=true&fitwidth=true';
          }
        }
      });
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
          // 选中的车站
          if (LOCATION_NUMBER !== "") {
            _t.formItem.station = LOCATION_NUMBER;
          }
        }
      }
      // 线路接口调完之后调用查询接口
      //_t.getData();
    },
    // 获取高度
    getHeight () {
      var ifm= document.getElementById("iframepage");
      var div= document.getElementById("div");
      ifm.height=document.documentElement.clientHeight -  div.offsetHeight*2;
    },
    // 重置按钮
    resetQuery() {
      // 查询表单
      this.formItem.dateTime = '';// 时间间隔
      this.formItem.type = '';// 设备类型
      // this.isShow = false;
    },
  },
  created() {
    this.getLinesAndStation();
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
