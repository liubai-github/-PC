<template>
  <div class="AllEquipment">
    <div v-if="EqList.length > 0" class="ListBox">
      <Row type="flex" justify="start" class="code-row-bg">
        <Col span="8"  v-for="(EQ, idx) in EqList" :key="idx">
          <Card style="width: 90%;margin:0 auto 30px auto;">
            <div style="text-align:left">
              <Row>
                <Col span="8"><img :src="EQ.device_img ? EQ.device_img : '../../../static/img/icons/eqNormalIcon.png'"></Col>
                <Col span="16">
                  <h4>{{EQ.device_name}}</h4>
                  <p>状态: {{EQ.on_off_status == 0 ? '在线' : '离线'}}</p>
                  <p>位置: {{EQ.house_name}} </p>
                  <Row class="operationIcon">
                    <Col span="24">
                      <img v-if="curHome.isCreater" class="iconImg" src="../../../static/img/icons/move-up.png" @click="moveEq(EQ)">
                      <img v-if="!curHome.isCreater" class="iconImg" src="../../../static/img/icons/move-up_gray.png" @click="moveEq(EQ)">
                      <img v-if="EQ.default_device_type == 'HAir(有线)'" class="iconImg" src="../../../static/img/icons/AnalysisBlue.png" @click="showCharts(EQ)">
                      <!-- <i-switch style="float:right;margin-top:5px;" size="small" @on-change="setConfig(EQ)"/> -->
                    </Col>
                  </Row>
                </Col>
              </Row>
              <Row class="MarginT_10 PaddingT_16 BorderT_gray">
                <Col span="12"><img class="iconImg" src="../../../static/img/icons/editor-line.png"><span @click="editEqInfo(EQ.id)">编辑</span></Col>
                <Col span="12" class="TextAlignR"><img class="iconImg" src="../../../static/img/icons/delete.png"><span @click="deleteEq(EQ)">删除</span></Col>
              </Row>
            </div>
          </Card>
        </Col>
      </Row>
    </div>
    <NoData v-if="EqList.length == 0"/>
    <!-- 移动 -->
    <Modal v-model="ifMove" width="360">
      <p slot="header" style="color:#333;text-align:left">
          <span>设备移动</span>
      </p>
      <div style="text-align:left">
          <Form label-position="left" :label-width="100">
            <FormItem label="设备名称" style="margin-bottom: 10px;">
              <p>{{curEQName}}</p>
            </FormItem>
            <FormItem label="当前房间" style="margin-bottom: 10px;">
              <p>{{curEQRoom}}</p>
            </FormItem>
            <FormItem label="移动至" style="margin-bottom: 10px;">
              <p>{{moveDestination}}</p>
            </FormItem>
            <RadioGroup v-model="moveDestination" vertical @on-change="changeDestinationRoom">
              <Radio :label="House.house_name" v-for="(House, idx) in HouseList" :key="idx">
                  <Icon type="social-apple"></Icon>
                  <span>{{House.house_name}}</span>
              </Radio>
            </RadioGroup>
          </Form>
      </div>
      <div slot="footer">
          <!-- <Button type="error" size="large" long  @click="sureMove">确认移动</Button> -->
          <Button type="primary" :loading="btLoading" @click="sureMove">
            <span v-if="!btLoading">确认移动</span>
            <span v-else>Loading...</span>
          </Button>
      </div>
    </Modal>
    <!-- 添加设备 -->
    <Modal v-model="ifAddEQ" scrollable width="360">
      <p slot="header" style="color:#333;text-align:left">
          <!-- <Icon type="ios-information-circle"></Icon> -->
          <span>添加设备</span>
      </p>
      <div style="text-align:left">
         <Row  class="MarginB_10">
          <Col span="24">
            <RadioGroup v-model="EqType">
              <Radio label="无线设备"></Radio>
              <Radio label="红外设备" disabled></Radio>
              <Radio label="单品"></Radio>
            </RadioGroup>
          </Col>
         </Row>
         <!-- 单品 -->
          <Form v-show="EqType == '单品'" :model="formSingle" ref="formSingle" :rules="ruleValidateSingle" label-position="left" :label-width="100">
            <FormItem label="选择单品类型" prop="SingleSelectName">
              <Select v-model="formSingle.SingleSelectName">
                <Option v-for="item in SingleList" :value="item.deviceDescibe" :key="item.id">{{ item.deviceDescibe }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择房间" prop="selectRoomId">
              <Select v-model="formSingle.selectRoomId">
                <Option v-for="item in HouseList" :value="item.id" :key="item.id">{{ item.house_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="单品名称" prop="SingleName">
              <Input v-model="formSingle.SingleName" placeholder="可输入自定义名称" style="" />
            </FormItem>
            <FormItem label="单品码" prop="SingleCode">
              <Input v-model="formSingle.SingleCode" placeholder="请输入主控码" style="" />
            </FormItem>
          </Form>
          <!-- 无线设备 一般三级设备 -->
          <Form v-show="EqType == '无线设备'" :model="formEQNormal" ref="formEQNormal" :rules="ruleValidateEQ" label-position="left" :label-width="100">
            <FormItem label="选择主控" prop="selectMasterId">
              <Select v-model="formEQNormal.selectMasterId" @on-change="selectChangeMaster">
                <Option v-for="item in MasterControlList" :value="item.id" :key="item.id">{{ item.main_control_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择从控" prop="selectSecondId">
              <Select v-model="formEQNormal.selectSecondId" @on-change="selectChangeSecond">
                <Option v-for="item in SecondControlList" :value="item.id" :key="item.id">{{ item.second_control_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择房间" prop="selectRoomId">
              <Select v-model="formEQNormal.selectRoomId">
                <Option v-for="item in HouseList" :value="item.id" :key="item.id">{{ item.house_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择设备类型" prop="eqType">
              <Select v-model="formEQNormal.eqType">
                <Option v-for="item in deviceTypeList" :value="item.device_type_code" :key="item.id">{{ item.device_type_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="设备名称" prop="SingleName">
              <Input v-model="formEQNormal.EQName" placeholder="可输入自定义名称" style="" />
            </FormItem>
            <FormItem label="设码" prop="SingleCode">
              <Input v-model="formEQNormal.EQCode" placeholder="请输入主控码" style="" />
            </FormItem>
          </Form>
      </div>
      <div slot="footer" class="TextAlignC">
          <Button v-if="EqType == '单品'"  type="error" size="large" :loading="btLoading" @click="handleSubmitSingle('formSingle')">
            <span v-if="!btLoading">确认添加</span>
            <span v-else>Loading...</span>
          </Button>
          <Button v-if="EqType == '无线设备'"  type="error" size="large" :loading="btLoading" @click="handleSubmitEQ('formEQNormal')">
            <span v-if="!btLoading">确认添加</span>
            <span v-else>Loading...</span>
          </Button>
          <!-- <Button v-if="EqType == '单品'" type="error" size="large" @click="handleSubmitSingle('formSingle')">确认添加</Button>
          <Button v-if="EqType == '无线设备'" type="error" size="large" @click="handleSubmitEQ('formEQNormal')">确认添加</Button> -->
      </div>
    </Modal>
    <!-- chart -->
    <Modal v-model="ifShowChart" fullscreen title="数据详情">
        <!-- <div class="MarginB_10">选择类型</div> -->
        <div class="kindBar" style="text-align:left">
          <Row>
            <Col span="4">选择类型</Col>
            <Col span="20">
              <RadioGroup v-model="curKind" @on-change="changeKind">
                <Radio v-for="(item, idx) in kind" :key="idx" :label="item"></Radio>
              </RadioGroup>
            </Col>
          </Row>
          <Row class="MarginB_10">
            <Col span="4">选择时间间隔</Col>
            <Col span="20">
              <RadioGroup v-model="time" @on-change="changeTime">
                <Radio label="6"></Radio>
                <Radio label="12"></Radio>
                <Radio label="24"></Radio>
              </RadioGroup>
            </Col>
          </Row>
        </div>
        <div v-show="hasData" id="myChart" :style="{width: '1024px', height: '300px', margin: '0 auto'}"></div>
        <div class="TextAlignC MarginT_30" v-show="!hasData && !ifSpin">
          <img style="width:100px;height:100px;margin:30px auto;" src="../../../static/img/icons/nodata.png">
          <p>暂无数据</p>
        </div>
    </Modal>
  </div>
</template>

<script>
import {mapState, mapActions} from 'vuex'
import {send} from '../../util/send'
import NoData from '../NoData.vue'
export default {
  name: 'AllEquipment',
  props: ['curHomeId', 'AddList', 'MasterControlList', 'deviceTypeList'],
  data () {
    return {
      btLoading: false,
      moveDestination: '',
      moveDestinationId: '',
      ifMove: false,
      EqList: [],
      // HouseList: [],
      newEqName: '',
      curEQName: '',
      curEQRoom: '',
      curEQId: '',
      // add
      EqType: '无线设备',
      MasterList: [],
      SingleList: [],
      formSingle: {
        SingleName: '',
        SingleCode: '',
        SingleSelectName: '',
        selectRoomId: '',
        selectMasterId: ''
      },
      ruleValidateSingle: {
        SingleSelectName: [
          { required: true, message: '请选择单品！', trigger: 'change' }
        ],
        SingleCode: [
          { required: true, message: '单品码不能为空！', trigger: 'blur' }
        ],
        selectRoomId: [
          { required: true, message: '请选择所在房间！', trigger: 'change' }
        ]
      },
      // 一般三级设备
      loading: false,
      SecondControlList: [],
      choosedMaster: {},
      choosedSecond: {},
      formEQNormal: {
        EQName: '',
        EQCode: '',
        eqType: '',
        selectRoomId: '',
        selectMasterId: '',
        selectSecondId: ''
      },
      ruleValidateEQ: {
        selectMasterId: [
          { required: true, message: '请选择主控！', trigger: 'change' }
        ],
        selectSecondId: [
          { required: true, message: '请选择从控！', trigger: 'change' }
        ],
        EQCode: [
          { required: true, message: '单品码不能为空！', trigger: 'blur' }
        ],
        selectRoomId: [
          { required: true, message: '请选择所在房间！', trigger: 'change' }
        ],
        eqType: [
          { required: true, message: '请选择设备类型！', trigger: 'change' }
        ]
      },
      // chhart
      curKind: 'PM2.5',
      curKindI: 'pm25',
      time: '6',
      kindI: ['pm25', 'temperature', 'humidity', 'pm100', 'formaldehyde', 'voc', 'co2', 'co'],
      kind: ['PM2.5', '温度', '湿度', 'PM10', '甲醛', 'VOCs', 'CO2', 'CO'],
      hasData: false
    }
  },
  computed: {
    ...mapState({
      curHome: state => state.curHome,
      ifSpin: state => state.ifSpin,
      HouseList: state => state.roomList
    }),
    ifAddEQ: {
      get: function () {
        return this.$store.state.ifAddEQ
      },
      set: function (newValue) {
        this.changeModalShow('EQ')
      }
    },
    ifShowChart: {
      get: function () {
        return this.$store.state.ifShowChart
      },
      set: function (newValue) {
        this.changeModalShow('Chart')
      }
    }
  },
  watch: {
    curHomeId: function (val) {
      this.getAllEq()
    },
    curKind: function (val) {
      this.drawLine()
    },
    time: function (val) {
      this.drawLine()
    }
  },
  components: {
    NoData
  },
  created () {
    this.getAllEq()
    this.filterSingle()
    // this.getAllRoom()
  },
  methods: {
    ...mapActions([
      'toggleSpin',
      'changeModalShow'
    ]),
    editEqInfo (EqId) {
      if (!this.curHome.isCreater) {
        this.$Message.warning('您不是管理员不能进行该操作！')
        return false
      }
      this.$Modal.confirm({
        onOk: () => {
          this.sureModify(EqId)
        },
        render: (h) => {
          return h('Input', {
            props: {
              value: this.newEqName,
              autofocus: true,
              placeholder: '请输入新的设备名称...'
            },
            on: {
              input: (val) => {
                this.newEqName = val
              }
            }
          })
        }
      })
    },
    deleteEq (EQ) {
      if (!this.curHome.isCreater) {
        this.$Message.warning('您不是管理员不能进行该操作！')
        return false
      }
      this.$Modal.confirm({
        title: '提示',
        content: '确定删除该设备?',
        onOk: () => {
          this.sureDel(EQ)
        },
        onCancel: () => {
        }
      })
    },
    // 删除设备
    sureDel (EQ) {
      this.toggleSpin(true)
      send({
        name: '/device?id=' + EQ.id + '&device_type=' + EQ.deviceType + '&second_control_id=' + EQ.second_control_id + '&home_id=' + this.curHomeId + '&register_id=' + this.$store.state.register_id,
        method: 'DELETE',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.getAllEq()
            setTimeout(() => {
              this.toggleSpin(false)
              this.$Message.success('删除成功!')
            }, 1000)
            break
          default:
            this.toggleSpin(false)
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.toggleSpin(false)
        this.$Message.error('Interface Error!')
      })
    },
    // 设备移动Moadl
    moveEq (item) {
      if (!this.curHome.isCreater) {
        this.$Message.warning('您不是管理员不能进行该操作！')
        return false
      }
      this.curEQName = item.device_name
      this.curEQRoom = item.house_name
      this.moveDestination = item.house_name
      this.curEQId = item.id
      this.ifMove = true
    },
    // 查看chart
    showCharts (EQ) {
      // 没配置过先配置
      if (EQ.device_config !== '1') {
        this.setConfig(EQ)
      } else {
        this.changeModalShow('Chart')
        this.drawLine()
      }
    },
    changeKind (val) {
      this.kind.map((item, idx) => {
        if (item === val) {
          this.curKindI = this.kindI[idx]
        }
      })
    },
    changeTime (val) {
      this.time = val
      console.log(val)
    },
    drawLine () {
      this.toggleSpin(true)
      let _THIS = this
      // 初始化echarts实例
      let myChart = this.$echarts.init(document.getElementById('myChart'))
      send({
        name: '/hair?device_id=00123456&type=' + _THIS.curKindI + '&hh=' + _THIS.time,
        method: 'GET',
        data: {
        }
      }).then(_res => {
        if (_res.data.result.hairList.length === 0) {
          _THIS.hasData = false
          this.toggleSpin(false)
          return false
        } else {
          _THIS.hasData = true
        }
        switch (_res.data.result.code) {
          case 1:
            let tempData = _res.data.result.hairList
            let option = {
              title: {
                text: ''
              },
              tooltip: {
                trigger: 'axis'
              },
              xAxis: {
                data: tempData.map(function (item) {
                  return item.faddtime || ''
                })
              },
              yAxis: {
                splitLine: {
                  show: false
                }
              },
              toolbox: {
                left: 'center',
                feature: {
                  dataZoom: {
                    yAxisIndex: 'none'
                  },
                  restore: {},
                  saveAsImage: {}
                }
              },
              dataZoom: [{
                startValue: '2018-09-01'
              }, {
                type: 'inside'
              }],
              visualMap: {
                top: 10,
                right: 10,
                pieces: _THIS.$store.state.LimitRange[_THIS.curKind],
                outOfRange: {
                  color: '#999'
                }
              },
              series: {
                name: _THIS.curKind + '数值',
                type: 'line',
                data: tempData.map(function (item) {
                  return item[_THIS.curKindI]
                }),
                markLine: {
                  silent: true
                  // data: [{
                  //   yAxis: 50
                  // }, {
                  //   yAxis: 100
                  // }, {
                  //   yAxis: 150
                  // }, {
                  //   yAxis: 200
                  // }, {
                  //   yAxis: 300
                  // }]
                }
              }
            }
            myChart.setOption(option)
            this.toggleSpin(false)
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.toggleSpin(false)
        this.$Message.error('Interface Error!')
      })
    },
    // 命令序号
    getCurCNS (MasterCode) {
      return new Promise((resolve, reject) => {
        send({
          name: '/deviceSequence?main_control_code=' + MasterCode,
          method: 'GET',
          data: {
          }
        }).then(_res => {
          switch (_res.data.code) {
            case 1:
              resolve(_res.data.sequence_number)
              break
            case 0:
              this.$Message.error(_res.data.message)
              break
            default:
              this.$Message.error(_res.data.message)
          }
        }).catch((_res) => {
          console.log(_res)
          this.$Message.error('Interface Error!')
        })
      })
    },
    setConfig (EQ) {
      this.EqConfig(EQ)
    },
    async EqConfig (eqItem) {
      var sendCurCns = await this.getCurCNS(eqItem.master_control)
      send({
        name: '/deviceConfig?main_control_id=' + eqItem.master_control + '&second_control_id=00123456' + '&sequence_number=' + sendCurCns + '&measure_point=08&control_point=00&qty=01',
        method: 'GET',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            // this.EqPlay(eqItem, sendCurCns)
            this.UpdateConfigStatus(eqItem.id, eqItem.master_control)
            break
          case 0:
            this.$Message.error(_res.data.message)
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // async EqPlay (eqItem, sendCurCns) {
    //   // var sendCurCns = await this.getCurCNS(eqItem.main_control_code)
    //   wepy.showLoading({
    //     title: '加载中'
    //   })
    //   return new Promise((resolve, reject) => {
    //     wepy.request({
    //       url: this.urlPre + '/payload?topic=' + eqItem.main_control_code + '/ConfigAck&sequence_number=' + sendCurCns,
    //       method: 'GET'
    //     }).then((res) => {
    //       console.log(res)
    //       switch (res.data.result.code) {
    //         case 1:
    //           wepy.hideLoading()
    //           // console.log(res.data.result.payload.payload)
    //           // console.log(res.data.result.payload.payload === 'sendCurCns')
    //           // if (res.data.result.payload.payload === 'sendCurCns') {
    //           //   wepy.showToast({
    //           //     title: '指令匹配!',
    //           //     icon: 'success'
    //           //   })
    //           // } else {
    //           //   this.$invoke('toast', 'show', {
    //           //     title: '指令不匹配!',
    //           //     img: '../../images/icons/attention.png'
    //           //   })
    //           // }
    //           this.UpdateConfigStatus(eqItem.id, eqItem.main_control_code)
    //           resolve(res)
    //           break
    //         case 0:
    //           wepy.hideLoading()
    //           this.$invoke('toast', 'show', {
    //             title: res.data.result.message,
    //             img: '../images/icons/attention.png'
    //           })
    //           break
    //         default:
    //           wepy.hideLoading()
    //           this.$invoke('toast', 'show', {
    //             title: '服务器繁忙！',
    //             img: '../images/icons/attention.png'
    //           })
    //       }
    //     }).catch((res) => {
    //       console.log(res)
    //       wepy.hideLoading()
    //       this.$invoke('toast', 'show', {
    //         title: '服务器繁忙！',
    //         img: '../images/icons/attention.png'
    //       })
    //     })
    //   })
    // }
    UpdateConfigStatus (EQID, MasterCode) {
      send({
        name: '/deviceConfig?id=' + EQID + '&device_config=1' + '&main_control_code=' + MasterCode,
        method: 'PUT',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.getAllEq()
            break
          case 0:
            this.$Message.error('Interface Error!')
            break
          default:
            this.$Message.error('Interface Error!')
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // 获取目标房间ID
    changeDestinationRoom (RommName) {
      this.HouseList.map((room, idx) => {
        if (room.house_name === RommName) {
          this.moveDestinationId = room.id
        }
      })
    },
    // 确认移动
    sureMove () {
      // this.toggleSpin(true)
      if (this.btLoading) {
        return false
      }
      this.btLoading = true
      send({
        name: '/deviceHouse?id=' + this.curEQId + '&house_id=' + this.moveDestinationId,
        method: 'PUT',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.getAllEq()
            setTimeout(() => {
              // this.toggleSpin(false)
              this.ifMove = false
              this.btLoading = false
              this.$Message.success('修改成功!')
            }, 1500)
            break
          default:
            // this.toggleSpin(false)
            this.btLoading = false
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        // this.toggleSpin(false)
        this.$Message.error('Interface Error!')
      })
    },
    // 更新设备名称
    sureModify (EqId) {
      console.log(EqId)
      send({
        name: '/device?device_name=' + this.newEqName + '&id=' + EqId + '&house_id=' + this.curHomeId,
        method: 'PUT',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.$Message.success('修改成功!')
            this.getAllEq()
            this.newEqName = ''
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error22!')
      })
    },
    // 所有设备
    getAllEq () {
      send({
        name: '/selecteqlist?home_id=' + this.curHomeId,
        method: 'GET',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.EqList = _res.data.deviceHomeList
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // 当前家下房间
    getAllRoom () {
      send({
        name: '/house?home_id=' + this.curHomeId,
        method: 'GET',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.HouseList = _res.data.houseList
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // 单品添加列表
    filterSingle () {
      let SingleList = this.AddList.filter(master => {
        if (master.devcieType === '2') {
          return master
        }
      })
      this.SingleList = SingleList
    },
    // 提交
    handleSubmitSingle (name) {
      this.$refs[name].validate((valid) => {
        if (valid) {
          this.addSingleProduct()
        } else {
        }
      })
    },
    handleSubmitEQ (name) {
      this.$refs[name].validate((valid) => {
        if (valid) {
          this.addEQProduct()
        } else {
        }
      })
    },
    // 添加单品
    addSingleProduct () {
      // this.toggleSpin(true)
      if (this.btLoading) {
        return false
      }
      this.btLoading = true
      send({
        name: '/device',
        method: 'POST',
        data: {
          register_id: this.$store.state.register_id,
          home_id: this.curHomeId,
          house_id: this.formSingle.selectRoomId,
          main_control_code: '',
          main_control_id: '',
          second_control_id: '',
          second_contrl_code: '',
          device_name: this.formSingle.SingleName.trim() === '' ? this.formSingle.SingleSelectName : this.formSingle.SingleName,
          device_code: this.formSingle.SingleCode,
          default_device_type: '',
          device_img: '',
          device_status: 0,
          deviceType: 2,
          device_config: 99
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            // this.toggleSpin(false)
            this.changeModalShow('EQ')
            this.$Message.success('添加成功!')
            this.getAllEq()
            this.btLoading = false
            // 清除输入
            this.EqType = '无线设备'
            this.formSingle = {
              SingleName: '',
              SingleCode: '',
              SingleSelectName: '',
              selectRoomId: ''
            }
            break
          default:
            // this.toggleSpin(false)
            this.btLoading = false
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        // this.toggleSpin(false)
        this.btLoading = false
        this.$Message.error('Interface Error!')
      })
    },
    // 切换主控
    selectChangeMaster (MasterId) {
      let temp = this.formEQNormal
      let Master = this.MasterControlList.filter(master => {
        if (master.id === MasterId) {
          return master
        }
      })
      this.choosedMaster = Master[0]
      this.getSecondControlList(Master[0].main_control_code)
      temp.selectSecondId = ''
      this.formEQNormal = temp
    },
    // 获取该主控下所有从控
    getSecondControlList (MasterCode) {
      send({
        name: '/secondControl?home_id=' + this.curHomeId + '&main_control_code=' + MasterCode,
        method: 'GET',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.SecondControlList = _res.data.secondControlList
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // 切换从控
    selectChangeSecond (SecondId) {
      let Second = this.SecondControlList.filter(second => {
        if (second.id === SecondId) {
          return second
        }
      })
      this.choosedSecond = Second[0]
    },
    // 添加设备
    addEQProduct () {
      // this.toggleSpin(true)
      if (this.btLoading) {
        return false
      }
      this.btLoading = true
      send({
        name: '/device',
        method: 'POST',
        data: {
          register_id: this.$store.state.register_id,
          home_id: this.curHomeId,
          house_id: this.formEQNormal.selectRoomId,
          main_control_code: this.choosedMaster.main_control_code,
          main_control_id: this.choosedMaster.id,
          second_control_id: this.choosedSecond.id,
          second_contrl_code: this.choosedSecond.secondControl_Code,
          device_name: this.formEQNormal.EQName.trim(),
          device_code: this.formEQNormal.EQCode,
          default_device_type: this.formEQNormal.eqType,
          device_img: '',
          device_status: 0,
          deviceType: 4,
          device_config: 99
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            // this.toggleSpin(false)
            this.changeModalShow('EQ')
            this.$Message.success('添加成功!')
            this.getAllEq()
            this.btLoading = false
            // 清除输入
            this.EqType = '无线设备'
            this.formSingle = {
              SingleName: '',
              SingleCode: '',
              SingleSelectName: '',
              selectRoomId: ''
            }
            break
          default:
            // this.toggleSpin(false)
            this.btLoading = false
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        // this.toggleSpin(false)
        this.btLoading = false
        this.$Message.error('Interface Error!')
      })
    },
    // 所有主控
    remoteMethod (query) {
      let that = this
      if (query !== '') {
        that.loading = true
        setTimeout(() => {
          that.loading = false
          that.getMasterControl()
        }, 200)
      } else {
        that.MasterControlList = []
      }
    },
    getMasterControl () {
      send({
        name: '/mainControl?home_id=' + this.curHomeId + '&register_id=' + this.$store.state.register_id,
        method: 'GET',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.MasterControlList = _res.data.homeList
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    }
  }
}
</script>
<style lang="less">
.ListBox{
  margin: 40px 0;
  img{
    width: 60px;
    height: 60px;
  }
  p, span{
    color: #333;
  }
  span{
    font-weight: bold;
    font-size: 14px;
    cursor: pointer;
  }
  .operationIcon{
    margin-top: 10px;
  }
  .iconImg{
    width: 20px;
    height: 20px;
    margin-right: 5px;
    display: inline-block;
    vertical-align: bottom;
    cursor: pointer;
  }
  .ivu-col-span-10{
    margin-bottom: 30px;
  }

  .ivu-input-large {
    border-top: 0px !important;
    order-left: 0px;
    order-right: 0px;
  }
}
</style>
