<template>
  <div class="cark1">
    <!-- 顶部菜单 -->
    <div style="margin-bottom: 16px">
      <label>学生:</label>
      <a-select style="width: 200px" @select="reloadAll(1)" v-model="studentId">
        <a-select-option v-for="item in studentList" :key="item.id" :value="item.sysUserId"> {{ item.name }}
        </a-select-option>
      </a-select>
      <label>老师:</label>
      <a-select style="width: 200px" @select="reloadAll(1)" v-model="teacherId">
        <a-select-option v-for="item in teacherList" :key="item.id" :value="item.sysUserId"> {{ item.name }}
        </a-select-option>
      </a-select>
      <label>日期</label>
      <a-range-picker
        v-model="dateRange"
        @change="reloadAll(1)"
        :allowClear="false"
        :ranges="{
              '上月': [moment().subtract(1,'month').startOf('month'), moment().subtract(1,'month').endOf('month')],
              '本月': [moment().startOf('month'), moment().endOf('month')]
              }"
        format="YYYY-MM-DD"
      >
      </a-range-picker>
    </div>
    <!--  表格的内容 -->
    <a-table
      :pagination="false"
      :columns="columns"
      :data-source="data"
      bordered
      rowKey="id"
    >
    </a-table>
    <!-- 分页 -->
    <a-pagination
      show-size-changer
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
      @change="reloadAll(pageNo)"
    />
  </div>
</template>

<script>
import moment from 'moment';
export default {
  data() {
    return {
      //表头
      columns:[
        {
          title: '学生',
          dataIndex: 'student',
        },
        {
          title: '老师',
          dataIndex: 'teacher',
        },
        {
          title: '上课日期',
          dataIndex: 'courseDate',
        },
        {
          title: '请假日期',
          dataIndex: 'restDate',
          //处理复杂数据结果
          customRender: (text, record) => {
            return moment(record.restDate).format("YYYY-MM-DD")
          }
        },
        {
          title: '产品',
          dataIndex: 'productName',
        },
        {
          title: '科目',
          dataIndex: 'subject',
        },
        {
          title: '情况说明',
          dataIndex: 'situation',
        }
      ],
      //表格数据
      data: [],
      studentList: [],
      studentId: '',
      teacherList: [],
      dateRange:null,
      teacherId: '',
      pageNo: 1,//当前页
      total: 1//总页数
    }
  },
  methods: {
    moment,
    //加载表格数据
    reloadAll(current) {
      let data = {
        pageNo: current,
        pageSize: 10,
        studentId:this.studentId,
        teacherId:this.teacherId
      }
      if (this.dateRange != null){
        data.dateRange = this.dateRange[0].format("YYYY-MM-DD")+"至"+this.dateRange[1].format("YYYY-MM-DD")
      }
      this.axios.post('pc/coursePlan/listRestByPage', data).then((res) => {
        this.data = res.result.records
        this.pageNo = res.result.current
        this.total = res.result.total
      })
    },
  },
  mounted() {
    this.axios.post('pc/studentInfo/listAll').then((res) => {
      this.teacherList = res.result
    })
    this.axios.post('pc/teacherInfo/listAll').then((res) => {
      this.teacherList = res.result
    })
    this.reloadAll(1)
  }
}
</script>

<style scoped>
::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
label {
  margin: 15px;
  font-weight: bold;
}
</style>