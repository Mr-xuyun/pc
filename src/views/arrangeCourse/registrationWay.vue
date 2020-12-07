<template>
  <div class="cark1">
    <div style="padding: 8px 10px; border-bottom: 1px solid #d9d9d9">
      <span :style="{ color: status == '报名方式' ? '#1B8FFF' : '#C7C7C7', fontSize: '20px' }">报名方式 </span>
      <span :style="{ color: status == '付款截图' ? '#1B8FFF' : '#C7C7C7', fontSize: '20px' }">-> 付款截图</span>
    </div>
    <!-- 报名方式 -->
    <div v-show="status == '报名方式'" style="margin-top: 15px">
      <div style="display: flex; justify-content: space-between">
        <div style="display: flex">
          <sidebar> 产品</sidebar>
        </div>
      </div>
      <table>
        <tr>
          <td>
            <img src="../../assets/星号.png" width="10" height="10" />
            <span style="margin-left: 5px">学生: </span>
            <a-select
              style="width: 150px; margin-left: 7px"
              @change="changeStudent"
              placeholder="请选择学生"
            >
              <a-select-option v-for="item of studentList" :key="item.sysUserId" :value="JSON.stringify(item)">
                {{ item.name }}
              </a-select-option>
            </a-select>
          </td>
          <td v-if="semester!=''">
            <span style="margin-left: 5px">学期段: {{ this.semester }} </span>
          </td>
          <td>
            <img src="../../assets/星号.png" width="10" height="10" />
            <span style="margin-left: 5px">产品: </span>
            <a-select
              style="width:250px; margin-left: 7px"
              @select="changeProduct"
              @deselect="deselectProduct"
              placeholder="请选择产品"
              mode='multiple'
              
            >
              <a-select-option v-for="item of productList" :key="item.id" :value="JSON.stringify(item)">
                {{ item.productName }}
              </a-select-option>
            </a-select>
          </td>
        </tr>

        <tr>
          <td colspan="5" style="height: 20px">
            <a-divider />
          </td>
        </tr>
        <sidebar style="margin-left: -10px;"> 合同</sidebar>
        <tr>
          <td>
            <span>总价:</span>
            <span class="total"> {{ totalPrices }}</span> 元
          </td>
          <td>
            <span style="margin-left: 20px">优惠: </span>
            <span class="total"> {{ reduce }}</span> 元
          </td>
          <td colspan="3">
            <span style="margin-left: 20px">实付: </span>
            <a-input-number
              size="large"
              v-model="actual"
              style="margin-left: 10px"
              :min="0"
              :step="100"
              :formatter="value => `¥ ${value}`"
            />
          </td>
        </tr>
        <tr>
          <td><span>协议时长: </span>
            <a-input-number v-model="contractDays" :min="1" />
            天
          </td>
          <td colspan="4"><span>起止日期：</span>
            <a-range-picker @change="startDateFun" />
          </td>
        </tr>
      </table>
      <div>
        <sidebar style="margin:15px 0px;"> 规划日期</sidebar>
        <div v-for="item of dataList" :key="item.productName">
          <h1 style="background-color: #1B8FFF;color: #FFFFFF;text-align: center;margin-top: 15px">{{ item.productName }}</h1>
          <table style="width:50%;margin-bottom:20px;">
            <tr>
              <td>
                <span style="margin-left: 10px">单价: </span>
                <span class="total"> {{ item.price }}</span> 元
              </td>
              <td v-if="item.unit=='次'">
                <span style="margin-left: 10px">规划次数:</span>
                <a-input-number style="margin-left: 10px" @blur="inputBlur(item)" v-model="item.count" :min="0" />
                次
              </td>
              <td v-else>
                <span style="margin-left: 10px">规划月份: </span>
                <a-input-number style="margin-left: 10px" @blur="inputBlur(item)" v-model="item.number" :min="1" />
                月
              </td>
            </tr>
          </table>
          <div class="planTime">
            <!-- 起止日期范围： -->
            <span>起止日期范围：</span>
            <a-range-picker
              v-model="item.dateRange"
              style="width: 220px"
            />
            <!-- 可用时间范围： -->
            <span style="margin-left: 20px">可用时间范围：</span>
            <a-time-picker placeholder="开始时间" v-model="item.start" format="HH:mm" />
            -
            <a-time-picker placeholder="结束时间" v-model="item.end" format="HH:mm" />
            <!-- 星期 -->
            <span style="margin-left: 20px">选择规律星期：</span>
            <a-select mode="multiple" v-model="item.weekRange" style="width: 20%" placeholder="请选择规律星期">
              <a-select-option v-for="(item, i) of weekList" :value="item.value" :key="i + 66">
                {{ item.text }}
              </a-select-option>
            </a-select>
          </div>
            
        </div>
      </div>
    </div>
    <!--  付款方式 -->
    <div v-show="status == '付款截图'" style="margin-top: 15px">
      <sidebar> 付款方式</sidebar>
      <a-tag :color="paymentColor == '线上' ? '#1B8FFF' : '#dcdfe6'" @click="paymentColor = '线上'"> 线上</a-tag>
      <a-tag :color="paymentColor == '线下' ? '#1B8FFF' : '#dcdfe6'" @click="paymentColor = '线下'"> 线下</a-tag>
      <a-divider type="vertical" />
      <a-radio-group default-value="vx" v-model="payWay" style="margin-left: 20px">
        <a-radio value="vx">
          <img src="../../assets/微信.png" width="45" height="45" />
        </a-radio>
        <a-radio value="zfc">
          <img src="../../assets/支付宝.png" width="35" height="35" />
        </a-radio>
        <a-radio value="yl">
          <img src="../../assets/银联.png" width="33" height="33" />
        </a-radio>
        <a-radio value="xj" v-show="paymentColor == '线下'">
          <img src="../../assets/现金.png" width="31" height="31" />
        </a-radio>
      </a-radio-group>
    </div>
    <a-divider />
    <!-- 支付的截图 -->
    <div style="display: flex" v-show="status == '付款截图'">
      <div>
        <sidebar> 付款截图</sidebar>
        <div class="clearfix" style="margin: 10px 15px">
          <a-upload
            :action="actionUrl"
            list-type="picture-card"
            :file-list="payFile"
            @preview="handlePreview"
            @change="handleChange"
          >
            <div v-if="payFile.length < 8">
              <a-icon type="plus" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
          <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible=false">
            <img alt="example" style="width: 100%" :src="previewImage" />
          </a-modal>
        </div>
        <sidebar> 协议截图</sidebar>
        <div class="clearfix" style="margin: 10px 15px">
          <a-upload
            :action="actionUrl"
            list-type="picture-card"
            :file-list="contractFile"
            @preview="protocolPreview"
            @change="protocolChange"
          >
            <div v-if="contractFile.length < 8">
              <a-icon type="plus" />
              <div class="ant-upload-text">
                上传
              </div>
            </div>
          </a-upload>
          <a-modal :visible="protocolVisible" :footer="null" @cancel="protocolVisible = false">
            <img style="width: 100%" :src="protocolImage" />
          </a-modal>
        </div>
      </div>
    </div>

    <div style="text-align: right">
      <a-button v-if="status == '报名方式'" style="margin-top: 50px" type="primary" @click="status = '付款截图'">
        下一步
      </a-button
      >
      <span v-else>
        <a-button style="margin-top: 50px" @click="status = '报名方式'"> 上一步</a-button>
        <a-button style="margin-top: 50px; margin-left: 20px" :loading="loading" @click="submit"
                  type="primary"> 生成预排课表</a-button>
      </span>
    </div>
  </div>
</template>
<script>
import Sidebar from '../../components/sidebar'
import moment from 'moment'

function getBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = (error) => reject(error)
  })
}

export default {
  components: { Sidebar },
  data() {
    return {
      moment,
      dataList: [],
      status: '报名方式',
      studentList: [],
      student: '',
      studentId: '',
      semester: '',
      productList: [],
      productName: [], //产品
      totalPrices: 0, //总价
      reduce: 0, //优惠
      actual: 0, //实付
      contractDays: 0, //协议天数
      contractTimeRange: null, //规划时间范围日期
      weekList: [
        { text: '星期一', value: 1 },
        { text: '星期二', value: 2 },
        { text: '星期三', value: 3 },
        { text: '星期四', value: 4 },
        { text: '星期五', value: 5 },
        { text: '星期六', value: 6 },
        { text: '星期日', value: 7 }
      ], //选择星期
      paymentColor: '线上', // 线上或者是线下 颜色切换
      payWay: '', // 付款的方式
      // 付款截图
      previewVisible: false,
      previewImage: '',
      payFile: [],
      //协议截图
      protocolVisible: false,
      protocolImage: '',
      contractFile: [],
      loading: false,
      actionUrl: window._CONFIG['domianURL'] + '/sys/common/upload',
    }
  },
  computed:{
        
  },
  methods: {
    moment,
    //学生
    changeStudent(value) {
      let json = JSON.parse(value)
      this.student = json.name
      this.studentId = json.sysUserId
      this.semester = json.semester
      this.axios.post('pc/product/listAll', { delFlag: 0, semester: this.semester }).then((res) => {
        console.log(res);
        this.productList = res.result
      })
    },
    //添加产品
    changeProduct(value) {
      let json = JSON.parse(value)
      let data = {
        productName: json.productName,
        productId: json.id,
        price: json.price,
        number: 0,
        count: 0,
        dateRange: null,
        start: null,
        end: null,
        weekRange: [],
        reducePrice: json.reducePrice,
        unit: json.unit
      }
       console.log(data);
      this.dataList.push(data)
      this.productName.push({ productName: json.productName, productId: json.id })
    },
    //删除
    deselectProduct(value) {
      let json = JSON.parse(value)
      for (let i = 0; i < this.dataList.length; i++) {
        if (json.productName == this.dataList[i].productName) {
          this.dataList.splice(i, 1)
        }
      }
      for (let i = 0; i < this.productName.length; i++) {
        if (json.productName == this.productName[i].productName) {
          this.productName.splice(i, 1)
        }
      }
    },
    //合同 起止日期范围
    startDateFun(date, dateString) {
      console.log(date)
      this.contractTimeRange = dateString.join('至')
      for (let data of this.dataList) {
        data.dateRange = date;
      }
    },
    //计算优惠
    inputBlur(item) {
      this.reduce = 0
      this.totalPrices += item.count * item.price
      let json = JSON.parse(item.reducePrice)
      let maxSum = item.count * item.price
      let maxReduce = 1
      for (let i = 0; i < json.length; i++) {
        if (i + 1 < json.length) {
          if (json[i].sum <= maxSum && maxSum < json[i + 1].sum) {
            maxSum = json[i].sum
            maxReduce = json[i].reduce
            break
          } else if (maxSum >= json[json.length - 1].sum) {
            maxSum = json[json.length - 1].sum
            maxReduce = json[json.length - 1].reduce
          }
        } else {
          maxSum = json[i].sum
          maxReduce = json[i].reduce
        }
      }
      if (maxReduce != 1 && maxSum < this.totalPrices) {
        this.reduce += Math.floor(this.totalPrices / maxSum) * maxReduce
      }
      this.actual = this.totalPrices - this.reduce
    },
    //支付截图
    async handlePreview(file) {
      if (!file.url && !file.preview) {
        file.preview = await getBase64(file.originFileObj)
      }
      this.previewImage = file.url || file.preview
      this.previewVisible = true
    },
    handleChange({ fileList }) {
      this.payFile = fileList
    },
    // 协议截图
    async protocolPreview(file) {
      if (!file.url && !file.preview) {
        file.preview = await getBase64(file.originFileObj)
      }
      this.protocolImage = file.url || file.preview
      this.protocolVisible = true
    },
    protocolChange({ fileList }) {
      this.contractFile = fileList
    },
    submit() {
      this.loading = true
      let data = {
        student: this.student,
        studentId: this.studentId,
        sum: this.totalPrices,
        reduce: this.reduce,
        actual: this.actual,
        contractDays: this.contractDays,
        contractTimeRange: this.contractTimeRange,
        payWay: this.payWay
      }
      let productName = ''
      let productId = ''
      for (let item of this.productName) {
        productName += item.productName + ','
        productId += item.productId + ','
      }
      data.productName = productName.substr(0, productName.length - 1)
      data.productId = productId.substr(0, productId.length - 1)
      for (let item of this.dataList) {
        item.weekRange = item.weekRange.join(',')
        item.dateRange = item.dateRange[0].format('YYYY-MM-DD') + '至' + item.dateRange[1].format('YYYY-MM-DD')
        item.timeRange = item.start.format("HH:mm") + ' - ' + item.end.format('HH:mm')
        delete item.start
        delete item.end
        delete item.reducePrice
        delete item.price
      }
      data.productDesc = JSON.stringify(this.dataList)
      let contractFile = ''
      for (let file of this.contractFile) {
        contractFile += file.response.message + ','
      }
      data.contractFile = contractFile.substr(0, contractFile.length - 1)
      let payFile = ''
      for (let file of this.payFile) {
        payFile += file.response.message + ','
      }
      data.payFile = payFile.substr(0, payFile.length - 1)
      console.log(data);
      // this.axios.post('pc/courseBill/saveBill', data).then((res) => {
      //   if (res.success && res.result) {
      //     this.$message.info(res.message)
      //     this.loading = false
      //   } else {
      //     this.$message.error('保存失败')
      //     this.loading = false
      //   }
      // })
    }
  },
  mounted() {
    this.axios.post('pc/studentInfo/listAll', { delFlag: 0 }).then((res) => {
      this.studentList = res.result
    })
  }
}
</script>
<style scoped>
table {
  margin-left: 10px;
  margin-top: 10px;
  width: 100%;
}

tr {
  height: 45px;
}

.total {
  background-color: #f3f3f3;
  padding: 5px 5px;
  padding-right: 8px;
  text-align: center;
  margin-left: 26px;
}

span {
  font-weight: bold;
}

/* 规划时间 */
.planTime {
  margin-left: 20px;
  display: flex;
  align-items: center;
}

::v-deep .ant-tag {
  border-radius: 20px;
  margin-left: 15px;
}
</style>
<style>
.ant-upload-select-picture-card i {
  font-size: 32px;
  color: #999;
}

.ant-upload-select-picture-card .ant-upload-text {
  margin-top: 8px;
  color: #666;
}
</style>
