<template>
  <div class="cark1">
    <!-- 顶部菜单 -->
    <div class="topMenu">
      <label>学生:</label>
      <a-select style="width: 130px" @select="reloadAll(1)" v-model="studentId" placeholder="筛选学生">
        <a-select-option v-for="(item, i) of studentList" :key="i" :value="item.sysUserId">
          {{ item.name }}</a-select-option
        >
      </a-select>
      <label>科目:</label>
      <a-select style="width: 130px" @select="reloadAll(1)" v-model="subject" placeholder="筛选科目">
        <a-select-option v-for="item in subjectList" :key="item.id" :value="item.title">
          {{ item.title }}</a-select-option
        >
      </a-select>
      <label>产品:</label>
      <a-select style="width: 150px" @select="reloadAll(1)" v-model="productId" placeholder="筛选产品">
        <a-select-option v-for="item in productList" :key="item.id" :value="item.id">
          {{ item.productName }}</a-select-option
        >
      </a-select>
      <label>日期:</label>
      <a-range-picker v-model="dateRange" @change="changeDate" />
      <a-button type="primary" @click="resetBtn" style="margin-left: 15px"> 重置</a-button>
    </div>
    <a-spin :spinning="spnning">
      <!--  表格的内容 -->
      <a-table :pagination="false" :columns="columns" :data-source="data" bordered rowKey="index">
        <!--操作页面 -->
        <span slot="action" slot-scope="text, record">
          <div style="display: flex">
            <a-divider type="vertical" />
            <a-button type="danger" size="small" @click="deleteItem(record)"> <a-icon type="delete" /> 删除 </a-button>
          </div>
        </span>
      </a-table>
    </a-spin>
    <!-- 分页 -->
    <a-pagination :default-current="1" v-model="pageNo"  show-size-changer @showSizeChange="changePageSize" :total="total" show-quick-jumper @change="reloadAll(pageNo)" />
  </div>
</template>

<script>
//表头
const columns = [
  {
    title: '学生',
    dataIndex: 'student',
  },
  {
    title: '科目',
    dataIndex: 'subject',
  },
  {
    title: '产品名称',
    dataIndex: 'productName',
  },
  {
    title: '总次数',
    dataIndex: 'studyCount',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.studyCount + '次'
    },
  },
  {
    title: '产品总时长',
    dataIndex: 'studyTime',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.studyTime + '小时'
    },
  },
  {
    title: '单价',
    dataIndex: 'price',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.price + '元'
    },
  },
  {
    title: '总消费',
    dataIndex: 'totalPrice',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.totalPrice + '元'
    },
  },
]
export default {
  data() {
    return {
      columns, //表头
      data: [], //表格数据
      hasSelected: true, //是否禁用全选按钮
      loading: false, //批量删除是否显示加载状态
      dateRange: [], //日期段
      start: '', //开始日期
      end: '', //结束日期
      studentList: [], //学生列表
      studentId: undefined, //学生ID
      subjectList: [], //科目列表
      subject: undefined, //科目
      productList: [], //产品列表
      productId: undefined, //产品ID
      pageNo: 1, //当前页
      total: 100, //总页数,
      spnning: false,
      pageSize :10
    }
  },
  methods: {
     // 页码
    changePageSize(current, size){
          this.pageSize = size
          this.reloadAll(current)
    },
    //   重置按钮
    resetBtn() {
      console.log(this.studentId)
      if (this.studentId || this.subject || this.productId || this.dateRange.length != 0) {
        this.studentId = undefined
        this.subject = undefined
        this.productId = undefined
        this.dateRange = []
        this.start = ''
        this.end = ''
        this.reloadAll(1)
      }
    },
    changeDate(date, dateStr) {
      this.start = dateStr[0]
      this.end = dateStr[1]
      this.reloadAll(1)
    },
    //加载表格数据
    reloadAll(current) {
      let data = {
        pageNo: current,
        pageSize: this.pageSize,
        studentId: this.studentId,
        productId: this.productId,
        start: this.start,
        end: this.end,
        subject: this.subject,
      }
      this.spnning = true
      this.axios.post('pc/coursePlan/getStudentStudyTotalTime', data).then((res) => {
        if (res.success) {
          this.pageNo = res.result.current
          this.total = res.result.total
          res.result.records.forEach((item, i) => {
            item.index = i
          })

          this.data = res.result.records
          console.log(this.data)
        }
        this.spnning = false
      })
    },
    loadInfo() {
      this.axios.post('pc/studentInfo/listAll').then((res) => {
        this.studentList = res.result
      })
      this.axios.post('pc/product/listAll', { delFlag: 0 }).then((res) => {
        this.productList = res.result
      })
      this.axios.post('vx/coursePublish/findCategoryByName', { name: '科目' }).then((res) => {
        this.subjectList = res.result
      })
    },
  },
  mounted() {
    this.reloadAll(1)
    this.loadInfo()
  },
}
</script>

<style scoped>
::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
label {
  margin-left: 15px;
  margin-right: 10px;
  font-weight: bold;
}
.topMenu {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}
</style>