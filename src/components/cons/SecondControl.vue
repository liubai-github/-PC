<template>
  <div class="AllEquipment">
    <div class="ListBox" v-if="SecondControlList.length > 0">
      <Row type="flex" justify="start" class="code-row-bg">
        <Col span="8"  v-for="(SecondControl, idx) in SecondControlList" :key="idx">
          <Card style="width: 90%;margin:0 auto 30px auto;">
            <div style="text-align:left">
              <Row>
                <Col span="8"><img src="../../../static/img/icons/SecondControlNoraml.png"></Col>
                <Col span="16">
                  <h4>{{SecondControl.second_control_name}}</h4>
                  <Row class="MarginT_20">
                    <Col span="12"><img class="iconImg" src="../../../static/img/icons/editor-line.png"><span @click="editSecondControl(SecondControl.id)">编辑</span></Col>
                    <Col span="12" class="TextAlignR"><img class="iconImg" src="../../../static/img/icons/delete.png"><span @click="deleteSecondControl(SecondControl)">删除</span></Col>
                  </Row>
                </Col>
              </Row>
            </div>
          </Card>
        </Col>
      </Row>
    </div>
    <NoData v-if="SecondControlList.length == 0"/>
    <!-- 添加主控 -->
    <Modal v-model="ifAddSecond" width="360">
      <p slot="header" style="color:#333;text-align:left">
          <!-- <Icon type="ios-information-circle"></Icon> -->
          <span>添加主控</span>
      </p>
      <div style="text-align:left">
          <Form :model="formSecond" ref="formSecond" :rules="ruleValidate" label-position="left" :label-width="100">
            <FormItem label="选择主控" prop="selectMasterId">
              <Select v-model="formSecond.selectMasterId" @on-change="selectChangeMaster">
                <Option v-for="item in MasterControlList" :value="item.id" :key="item.id">{{ item.main_control_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择从控" prop="selectSecondId">
              <Select v-model="formSecond.selectSecondId" @on-change="selectChangeSecond">
                <Option v-for="item in addSecondControlList" :value="item.id" :key="item.id">{{ item.deviceDescibe }}</Option>
              </Select>
            </FormItem>
            <FormItem label="选择房间" prop="selectRoomId" v-if="choosedSecond.length > 0 && choosedSecond[0].devcieType == '1'">
              <Select v-model="formSecond.selectRoomId">
                <Option v-for="item in roomList" :value="item.id" :key="item.id">{{ item.house_name }}</Option>
              </Select>
            </FormItem>
            <FormItem label="从控名称" prop="secondName">
              <Input v-model="formSecond.secondName" placeholder="可输入自定义名称" style="" />
            </FormItem>
            <FormItem label="从控码" prop="secondCode">
              <Input v-model="formSecond.secondCode" placeholder="请输入从控码" style="" />
            </FormItem>
          </Form>
      </div>
      <div slot="footer" class="TextAlignC">
        <Button type="error" size="large" :loading="btLoading" @click="handleSubmit('formSecond')">
          <span v-if="!btLoading">确认添加</span>
          <span v-else>Loading...</span>
        </Button>
        <!-- <Button type="error" size="large" @click="handleSubmit('formSecond')">确认添加</Button> -->
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
  props: ['curHomeId', 'AddList', 'MasterControlList'],
  data () {
    const validateRoom = (rule, value, callback) => {
      if (this.choosedSecond[0].devcieType === '1') {
        if (value === '') {
          callback(new Error('请选择所属房间!'))
        } else {
          callback()
        }
      } else {
        callback(new Error('over'))
      }
    }
    return {
      btLoading: false,
      SecondControlList: [],
      newSecondControlName: '',
      addSecondList: [],
      choosedMaster: {},
      choosedSecond: {},
      addSecondControlList: [],
      formSecond: {
        secondCode: '',
        secondName: '',
        selectMasterId: '',
        selectSecondId: '',
        selectRoomId: ''
      },
      ruleValidate: {
        selectMasterId: [
          { required: true, message: '请选择主控！', trigger: 'change' }
        ],
        selectSecondId: [
          { required: true, message: '请选择从控！', trigger: 'change' }
        ],
        secondCode: [
          { required: true, message: '从控码不能为空！', trigger: 'blur' }
        ],
        selectRoomId: [
          { validator: validateRoom, trigger: 'change' }
        ]
      }
    }
  },
  computed: {
    ...mapState({
      curHome: state => state.curHome,
      roomList: state => state.roomList
    }),
    ifAddSecond: {
      get: function () {
        return this.$store.state.ifAddSecond
      },
      set: function (newValue) {
        this.changeModalShow('Second')
      }
    }
  },
  watch: {
    curHomeId: function (val) {
      this.getSecondControl()
    }
  },
  components: {
    NoData
  },
  created () {
    this.getSecondControl()
    this.filterSecondControl()
  },
  methods: {
    ...mapActions([
      'toggleSpin',
      'changeModalShow'
    ]),
    editSecondControl (MasterControlId) {
      if (!this.curHome.isCreater) {
        this.$Message.warning('您不是管理员不能进行该操作！')
        return false
      }
      this.$Modal.confirm({
        onOk: () => {
          if (this.newSecondControlName.trim() === '') {
            this.$Message.error('从控名称不能为空!')
            return false
          }
          this.sureModify(MasterControlId)
        },
        render: (h) => {
          return h('Input', {
            props: {
              value: this.newSecondControlName,
              autofocus: true,
              placeholder: '请输入新的从控名称...'
            },
            on: {
              input: (val) => {
                this.newSecondControlName = val
              }
            }
          })
        }
      })
    },
    // 更新从控名称
    sureModify (SecondControlId) {
      send({
        name: '/secondControl?second_control_name=' + this.newSecondControlName + '&id=' + SecondControlId,
        method: 'PUT',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.$Message.success('修改成功!')
            this.getSecondControl()
            break
          default:
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    deleteSecondControl (SecondControl) {
      if (!this.curHome.isCreater) {
        this.$Message.warning('您不是管理员不能进行该操作！')
        return false
      }
      this.$Modal.confirm({
        title: '提示',
        content: '该操作会将' + SecondControl.second_control_name + '从控下面的设备全部删除，确定删除该从控?',
        onOk: () => {
          this.sureDel(SecondControl)
        },
        onCancel: () => {
        }
      })
    },
    // 删除从控
    sureDel (SecondControl) {
      this.toggleSpin(true)
      send({
        name: '/secondControl?id=' + SecondControl.id + '&home_id=' + this.curHomeId + '&register_id=' + this.$store.state.register_id,
        method: 'DELETE',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            this.getSecondControl()
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
    // 所有从控
    getSecondControl () {
      send({
        name: '/secondControl?home_id=' + this.curHomeId,
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
    // 从控添加列表
    filterSecondControl () {
      let SecondList = this.AddList.filter(second => {
        if (second.devcieType === '1' || second.devcieType === '3') {
          return second
        }
      })
      this.addSecondControlList = SecondList
      console.log(SecondList)
    },
    // 切换主控
    selectChangeMaster (MasterId) {
      let Master = this.MasterControlList.filter(master => {
        if (master.id === MasterId) {
          return master
        }
      })
      this.choosedMaster = Master
    },
    // 切换从控
    selectChangeSecond (SecondId) {
      let Second = this.addSecondControlList.filter(second => {
        if (second.id === SecondId) {
          return second
        }
      })
      this.choosedSecond = Second
      console.log(Second)
    },
    // 提交
    handleSubmit (name) {
      this.$refs[name].validate((valid) => {
        if (valid) {
          this.addSecond()
        } else {
          this.$Message.error('Fail!')
        }
      })
    },
    // 添加主控
    addSecond () {
      if (this.btLoading) {
        return false
      }
      // this.toggleSpin(true)
      this.btLoading = true
      send({
        name: '/secondControl?home_id=' + this.curHomeId + '&main_control_code=' + this.choosedMaster[0].main_control_code + '&main_control_id=' + this.choosedMaster[0].id + '&second_control_name=' + (this.formSecond.secondName.trim() !== '' ? this.formSecond.secondName.trim() : this.choosedSecond[0].deviceDescibe) + '&second_contrl_code=' + this.formSecond.secondCode + '&second_control_category=' + this.choosedSecond[0].deviceTypeName + '&deviceType=' + this.choosedSecond[0].devcieType,
        method: 'POST',
        data: {
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
            // this.toggleSpin(false)
            this.changeModalShow('Second')
            this.$Message.success('添加成功！')
            this.btLoading = false
            // 1-无下挂自动添加下级设备
            if (this.choosedSecond[0].devcieType === '1') {
              this.addAutoEq(_res.data.id, this.formSecond.secondCode)
            }
            this.getSecondControl()
            break
          default:
            // this.toggleSpin(false)
            this.btLoading = false
            this.$Message.error(_res.data.message)
        }
      }).catch((_res) => {
        // this.toggleSpin(false)
        this.btLoading = false
        console.log(_res)
        this.$Message.error('Interface Error!')
      })
    },
    // 自动添加下级设备
    addAutoEq (SecondId, secondControlCode) {
      send({
        name: '/device',
        method: 'POST',
        data: {
          register_id: this.$store.state.register_id,
          home_id: this.curHomeId,
          house_id: this.formSecond.selectRoomId,
          main_control_code: this.choosedMaster[0].main_control_code,
          main_control_id: this.choosedMaster[0].id,
          second_control_id: SecondId,
          second_contrl_code: secondControlCode,
          device_name: (this.formSecond.secondName.trim() !== '' ? this.formSecond.secondName.trim() : this.choosedSecond[0].deviceDescibe),
          device_code: secondControlCode,
          default_device_type: this.choosedSecond[0].deviceTypeName,
          device_img: '',
          device_status: 0,
          deviceType: this.choosedSecond[0].devcieType,
          device_config: 99
        }
      }).then(_res => {
        switch (_res.data.code) {
          case 1:
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
