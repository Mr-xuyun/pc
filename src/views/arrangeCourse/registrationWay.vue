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
      <a-form-model :model="form" :rules="rules" ref="ruleForm">
        <table>
          <tr>
            <td>
              <a-form-model-item label="学生"   :label-col="{ span: 3}"
                  :wrapper-col="{span:14}" prop="student">
                <a-select
                  style="width: 150px; margin-left: 7px"
                  @change="changeStudent"
                  placeholder="请选择学生"
                  v-model="form.student"
                >
                  <a-select-option v-for="item of studentList" :key="item.sysUserId" :value="JSON.stringify(item)">
                    {{ item.name }}
                  </a-select-option>
                </a-select>
              </a-form-model-item>
            </td>

            <span v-if="semester != ''" style="margin-top: 29px; display: inline-block"
              >学期段: {{ this.semester }}
            </span>
            <td>
              <a-form-model-item :label-col="labelCol" label="产品" :wrapper-col="wrapperCol" prop="prodoct">
                <a-select
                  v-model="form.prodoct"
                  style="width: 250px; margin-left: 7px"
                  @select="changeProduct"
                  @deselect="deselectProduct"
                  placeholder="请选择产品"
                  mode="multiple"
                >
                  <a-select-option v-for="item of productList" :key="item.id" :value="JSON.stringify(item)">
                    {{ item.productName }}
                  </a-select-option>
                </a-select>
              </a-form-model-item>
            </td>
          </tr>

          <tr>
            <td colspan="5" style="height: 0px">
              <a-divider />
            </td>
          </tr>
          <sidebar style="margin-left: -10px;margin-bottom:15px;"> 合同</sidebar>
          <tr>
            <td>
              <span>总价:</span>
              <span class="total"> {{ totalPrices }}</span> 元
            </td>
            <td>
              <span style="margin-left: 20px">优惠: </span>
              <span class="total"> {{ reduce==0 ? reduce : '-'+reduce }}</span> 元
            </td>
            <td colspan="3">
              <span style="margin-left: 20px">实付: </span>
              <span style="color:red;font-size: 16px; "> ¥ {{actual  }}</span>
            </td>
          </tr>
          <tr>
            <td>
              <div>
                <a-form-model-item
                  label="协议时长:"
                  :label-col="{ span: 4}"
                  :wrapper-col="{span:14}"
                  prop="contractDays"
                >
                  <a-input-number :min="1" style="margin-left: 10px" v-model="form.contractDays" />
                  <span style="margin-left: 10px">天</span>
                </a-form-model-item>
              </div>
            </td>
            <td colspan="3">
              <a-form-model-item
                label="起止日期:"
                :label-col="labelCol"
                :wrapper-col="wrapperCol"
                prop="start_stopDate"
              >
                <a-range-picker @change="startDateFun" v-model="form.start_stopDate" />
              </a-form-model-item>
            </td>
          </tr>
        </table>
      </a-form-model>
      <!-- 规划日期的校验 -->
      <a-form-model ref="dynamicValidateForm" :model="dynamicValidateForm">
        <div>
          <sidebar style="margin: 15px 0px"> 规划日期</sidebar>
          <div v-for="(item, i) of dynamicValidateForm.domains" :key="i">
            <h1 style="background-color: #1b8fff; color: #ffffff; text-align: center; margin-top: 15px;border-radius:7px;">
              {{ item.productName }}
            </h1>
            <div class="planTime">
              <div style="margin-top: 15px">
                <span style="margin-left: 10px">单价: </span>
                <span class="total"> {{ item.price }}</span> 元
              </div>
              <a-form-model-item
                v-if="item.unit == '次'"
                label="规划次数"
                :prop="'domains.' + i + '.count'"
                :rules="{
                  required: true,
                  message: '规划次数不能为空',
                  trigger: 'blur',
                }"
                :label-col="{ span: 10 }"
                :wrapper-col="{ span: 10 }"
                style="width: 400px"
                 placeholder="请输入规划次数"
              >
                <a-input-number style="margin-left: 10px" @blur="inputBlur(item)" v-model="item.count" :min="0" />
                  <span style="margin-left:8px">次</span>
              </a-form-model-item>

              <a-form-model-item
                v-else
                label="规划次数"
                :label-col="{ span: 10 }"
                :wrapper-col="{ span: 10 }"
                style="width: 400px"
                :prop="'domains.' + i + '.count'"
                :rules="{
                  required: true,
                  message: '规划次数不能为空',
                  trigger: 'blur',
                }"
                placeholder="请输入规划次数"
              >
                <a-input-number style="margin-left: 10px" @blur="inputBlur(item)" v-model="item.count" :min="0" />
                <span style="margin-left:8px">月</span>
              </a-form-model-item>
              <!-- 责任老师 -->
               <a-form-model-item v-if="item.unit=='月'"
                label="责任老师"
                :label-col="{ span: 10 }"
                :wrapper-col="{ span: 14 }"
                style="width:200px"
                :prop="'domains.' + i + '.teacherObj'"
                :rules="{
                  required: true,
                  message: '规划次数不能为空',
                  trigger: 'change',
                }"  >
                <a-select style="width: 150px" v-model="item.teacherObj"  placeholder="请选择老师"  >
                  <a-select-option v-for="it in teacherList" :key="it.sysUserId" :value="JSON.stringify({teacherId:it.sysUserId,teacher:it.name})"> {{ it.name }}
                  </a-select-option>
                </a-select>
              </a-form-model-item>
            </div>
            <div class="planTime">
              <!-- 起止日期范围： -->
              <a-form-model-item
              
                 label="起止日期范围："
                 :label-col="{ span: 7 }"
                 :wrapper-col="{ span: 8 }"
                :prop="'domains.' + i + '.dateRange'"
                :rules="{
                  required: true,
                  message: '日期范围不能为空',
                  trigger: 'change',
                }"   
              >
                 <a-range-picker v-model="item.dateRange" style="width: 220px;margin-left:10px;" /> 
              </a-form-model-item>
              <!--  -->
                   <!-- 可用时间范围： -->
                <a-form-model-item
                   label="可用时间范围："
                   style="width:300px;"
                  :label-col="{ span: 14 }"
                    :wrapper-col="{ span:10 }"
                    :prop="'domains.' + i + '.start'"
                    :rules="{
                      required: true,
                      message: '开始时间不能为空',
                      trigger: 'change',
                    }"   
                >
                  <a-time-picker placeholder="开始时间"   v-model="item.start" format="HH:mm" />
                </a-form-model-item>
                  <!-- 结束时间 -->
                  <a-form-model-item
                    style="margin-left:15px; width:150px;"
                    :label-col="{ span: 12 }"
                    :wrapper-col="{ span:18}"
                    :prop="'domains.' + i + '.end'"
                    :rules="{
                      required: true,
                      message: '结束时间不能为空',
                      trigger: 'change',
                    }"   
                >
                  <a-time-picker placeholder="结束时间" v-model="item.end" format="HH:mm" />
                </a-form-model-item>
             
             
              <!-- 星期 -->
                <a-form-model-item 
                style="margin-left:10px;"
                    label="选择规律星期："
                  :label-col="{ span: 8 }"
                    :wrapper-col="{ span:10 }"
                    :prop="'domains.' + i + '.weekRange'"
                    :rules="{
                      required: true,
                      message: '星期不能为空',
                      trigger: 'change',
                    }"   
                >
                  <a-select mode="multiple" v-model="item.weekRange" style="width: 250px;" placeholder="请选择规律星期">
                    <a-select-option v-for="(item, i) of weekList" :value="item.value" :key="i + 66">
                      {{ item.text }}
                    </a-select-option>
                  </a-select>
               </a-form-model-item>
            </div>
          </div>
        </div>
       
      </a-form-model>
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
          <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible = false">
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
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
          <a-modal :visible="protocolVisible" :footer="null" @cancel="protocolVisible = false">
            <img style="width: 100%" :src="protocolImage" />
          </a-modal>
        </div>
      </div>
    </div>

    <div style="text-align: right">
       <a-form-model-item v-if="status == '报名方式'" >
          <a-button style="margin-top: 50px" type="primary" html-type="submit" @click="checkForm('dynamicValidateForm')" > 下一步 </a-button>
       </a-form-model-item>
       <span v-else>
        <a-button style="margin-top: 50px" @click="status = '报名方式'"> 上一步</a-button>
        <a-button style="margin-top: 50px; margin-left: 20px" :loading="loading" @click="submit" type="primary">
          生成预排课表</a-button
        >
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
      contractDays: 0, //协议时长
      contractTimeRange: null, //规划时间范围日期
      weekList: [
        { text: '星期一', value: 1 },
        { text: '星期二', value: 2 },
        { text: '星期三', value: 3 },
        { text: '星期四', value: 4 },
        { text: '星期五', value: 5 },
        { text: '星期六', value: 6 },
        { text: '星期日', value: 7 },
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

      form: {
        student: undefined, //学生
        prodoct: [], //产品
        contractDays: undefined, //协议时长
        start_stopDate: null, // 起止日期
      },
      rules: {
        student: [{ required: true, message: '学生不能为空', trigger: 'change' }],
        prodoct: [{ required: true, message: '产品不能为空', trigger: 'change' }],
        contractDays: [{ required: true, message: '协议时长不能为空', trigger: 'blur' }],
        start_stopDate: [{ required: true, message: '起止日期为空', trigger: 'change' }],
      },
      dynamicValidateForm: {
        domains: [],
      },
      labelCol: { span: 4 },
      wrapperCol: { span: 14 },
    }
  },
  computed: {},
  methods: {
    // 校验表单是否为空
    checkForm(formName) {
         let status ; //规划日期 form表单的校验 
         let isCheck =  //合同和产品form的校验
        this.$refs[formName].validate(valid => { status = valid  })
        this.$refs.ruleForm.validate((valid) => { isCheck =valid })
           
        if(status==true && isCheck==true ){
                this.status = "付款截图"
        }else{
                this.$message.error("带星号的不能为空")
        }
    
      },

    //学生
    changeStudent(value) {
      let json = JSON.parse(value)
      this.student = json.name
      this.studentId = json.sysUserId
      this.semester = json.semester
      this.axios.post('pc/product/listAll', { delFlag: 0, semester: this.semester }).then((res) => {
        this.productList = res.result
      })
    },
    //添加产品
    changeProduct(value) {
      let json = JSON.parse(value)
      this.productName.push({ productName: json.productName, productId: json.id })
      this.dynamicValidateForm.domains.push({
        productName: json.productName,
        productId: json.id,
        price: json.price,
        teacher: null,
        number: 0,
        count: undefined,
        dateRange: null,
        start: null,
        end: null,
        weekRange: [],
        reducePrice: json.reducePrice,
        unit: json.unit,
        teacherObj :undefined
      })
    },
    //删除
    deselectProduct(value) {
      let json = JSON.parse(value)
      for (let i = 0; i < this.dynamicValidateForm.domains.length; i++) {
        if (json.productName == this.dynamicValidateForm.domains[i].productName) {
          this.dynamicValidateForm.domains.splice(i, 1)
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
      this.contractTimeRange = dateString.join('至')
      for (let data of this.dynamicValidateForm.domains) {
        data.dateRange = date
      }
    },
    //计算优惠
    inputBlur(item) {
      this.totalPrices = 0
       console.log(this.dynamicValidateForm.domains);
          //  let  priceTotal =  0;
      this.dynamicValidateForm.domains.forEach((item,i)=>{
             this.totalPrices +=  item.count * item.price 
               
      })
      // console.log(priceTotal);
      //    this.totalPrices = priceTotal
      // this.totalPrices += item.count * item.price
      let json = JSON.parse(item.reducePrice);
       
       // console.log(json);
      let maxSum =  this.totalPrices
        // console.log(maxSum,  this.totalPrices );
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
      }else{
           this.reduce=0
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
      if(this.payWay==''){
          this.$message.error("请选择付款的方式")
          return
      }
      if(this.payFile.length==0) {
           this.$message.error("请上传付款截图")
           return
      }
      if(this.contractFile.length==0) {
           this.$message.error("请上传协议截图")
           return
      }
      this.loading = true
      let data = {
        student: this.student,
        studentId: this.studentId,
        sum: this.totalPrices,
        reduce: this.reduce,
        actual: this.actual,
        contractDays: this.form.contractDays,
        contractTimeRange:   this.contractTimeRange,
        payWay: this.payWay,
      }
      let productName = ''
      let productId = ''
      for (let item of this.productName) {
        productName += item.productName + ','
        productId += item.productId + ','
      }
      data.productName = productName.substr(0, productName.length - 1)
      data.productId = productId.substr(0, productId.length - 1)
      console.log(this.dynamicValidateForm.domains);
      let dataList = [];
      for (let item of this.dynamicValidateForm.domains) {
        let obj = {};
        obj.weekRange = item.weekRange.join(',')
        obj.dateRange = item.dateRange[0].format('YYYY-MM-DD') + '至' + item.dateRange[1].format('YYYY-MM-DD')
        obj.timeRange = item.start.format('HH:mm') + ' - ' + item.end.format('HH:mm')
          if (item.teacherObj != undefined){
            obj.teacher = JSON.parse(item.teacherObj).teacher
            obj.teacherId = JSON.parse(item.teacherObj).teacherId
     
          }
           obj.productName = item.productName
           obj.productId=item.productId
           obj.price=item.price
           obj.number=item.number
           obj.count=item.count
           obj.unit=item.unit
        dataList.push(obj) 
      }
      data.productDesc = JSON.stringify(dataList)
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
      this.axios.post('pc/courseBill/saveBill', data).then((res) => {
        if (res.success && res.result) {
          this.$message.info(res.message)
          this.loading = false
        } else {
          this.$message.error('保存失败')
          this.loading = false
        }
      })
    },
  },
  mounted() {
    this.axios.post('pc/studentInfo/listAll', { delFlag: 0 }).then((res) => {
      this.studentList = res.result
    })
    this.axios.post('pc/teacherInfo/listAll').then((res) => {
      this.teacherList = res.result
    })
  },
}
</script>
<style scoped>
table {
  margin-left: 10px;
  margin-top: 10px;
  width: 100%;
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
  margin-top: 20px;
  display: flex;
  align-items: center;
}

::v-deep .ant-tag {
  border-radius: 20px;
  margin-left: 15px;
}
::v-deep .ant-form-item {
  margin-bottom: 0;
  margin-top: 18px;
}
::v-deep .ant-form-item-label > label {
  color: rgba(0, 0, 0, 0.65);
  font-weight: bold;
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
