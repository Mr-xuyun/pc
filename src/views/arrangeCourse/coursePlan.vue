<template>
  <div class="cark1">
    <div style="margin-bottom: 35px;display:flex; align-items:center;">
      <b>姓名: &nbsp;</b>
      <a-select @select="selectStudent" v-model="studentName" style="width: 150px" placeholder="请选择姓名">
        <a-select-option v-for="item in studentList" :key="item.id" :value="item.studentId"> {{ item.student }}
        </a-select-option>
      </a-select>
      <b style="margin-left: 20px;">日期: &nbsp;</b>
      <a-date-picker @change="changeDate"  v-model="courseDate" placeholder="请选择日期" />
      <a-button type="primary" @click="resetBtn" style="margin-left: 20px;" > 重置</a-button>
    </div>
    <!--  表格的内容 -->
       <a-spin :spinning="spinning">
    <a-table :columns="columns" :data-source="data" bordered :pagination="false" rowKey="index">
      <!-- 操作的页面 -->
      <span slot="action" slot-scope="text, record">
        <div style="display: flex">
          {{ text }}
          <a-button type="danger" size="small" @click="update(record)"> <a-icon type="edit" /> 修改 </a-button>
          <a-divider type="vertical" />
           <a-popconfirm
             title="是否保存?"
             ok-text="是"
             cancel-text="否"
             @confirm="reserveFun(record)"
           >
             <a-button type="primary" size="small"> 保存 </a-button>
           </a-popconfirm>
        </div>
      </span>
      <!-- 出勤日期 -->
      <a slot="courseDate" slot-scope="timeSlot, record">
        <a-date-picker
          :default-value="moment(record.courseDate,'YYYY-MM-DD')"
          :disabled="record.showUpdate"
          v-model="record.courseDate"
          
          format="YYYY-MM-DD" />
      </a>

      <!-- 时间段  -->
      <a slot="timeRange" slot-scope="timeSlot, record">
        <a-time-picker
          style="width: 100px"
          :disabled="record.showUpdate"
          v-model="record.start"
          :default-value="moment(record.timeRange.split('-')[0].trim(),'HH:mm')"
          format="HH:mm"
        />
        <span style="color: #000"> -</span>
        <a-time-picker
          style="width: 100px; margin-left: 8px"
          :disabled="record.showUpdate"
          v-model="record.end"
          :default-value="moment(record.timeRange.split('-')[1].trim(),'HH:mm')"
          format="HH:mm"
        />
      </a>
      <!--  科目 -->
      <a slot="subject" slot-scope="subject, record">
        <a-select style="width: 150px" @select="record.teacher='';getTeacher(record)" :disabled="record.showUpdate"
                  v-model="record.subject">
          <a-select-option v-for="item in record.subjectList" :key="item" :value="item"> {{ item }}</a-select-option>
        </a-select>
      </a>
      <!--   老师 -->
      <a slot="teacher" slot-scope="teacher, record">
        <a-select style="width: 150px" v-model="record.teacher" :disabled="record.showUpdate">
          <a-select-option v-for="item in teacherList" :key="item.sysUserId" :value="JSON.stringify({teacherId:item.sysUserId,teacher:item.name})"> {{ item.name }}
          </a-select-option>
        </a-select>
      </a>
    </a-table>
      </a-spin>
    <!-- 分页 -->
    <a-pagination
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
       show-size-changer
        @showSizeChange="changePageSize"
      @change="reloadAll(pageNo)"
    />
  </div>
</template>
<script>
import moment from 'moment'

export default {
  data() {
    return {
      columns: [
        {
          title: '学生',
          dataIndex: 'student',
          width: '100px'
        },
        {
          title: '出勤日期',
          dataIndex: 'courseDate',
          width: '125px',
          scopedSlots: { customRender: 'courseDate' }
        },
        {
          title: '时间段',
          dataIndex: 'timeRange',
          width: '220px',
          scopedSlots: { customRender: 'timeRange' }
        },
        {
          title: '产品',
          dataIndex: 'productName',
          width: '100px'
        },
        {
          title: '科目',
          dataIndex: 'subject',
          width: '100px',
          scopedSlots: { customRender: 'subject' }
        },

        {
          title: '老师',
          dataIndex: 'teacher',
          width: '100px',
          scopedSlots: { customRender: 'teacher' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          width: '100px',
          scopedSlots: { customRender: 'action' }
        }
      ],
      data: [],
      studentId: '',
      studentName:undefined,
      courseDate: null, // 选择日期
      studentList: [],
      teacherList: [],
      pageNo: 1,
      total: 0,
      spinning :false,
      pageSize : 10
    }
  },
  computed: {
    //清空
    hasSelected() {
      return this.selectedRowKeys.length > 0
    }
  },
  methods: {
    moment,
   // 页码数量
    changePageSize(current, size){
           this.pageSize =size
          this.reloadAll(current)
    },
    //重置按钮 
    resetBtn(){
        if(this.studentName!=undefined || this.courseDate!=""){
            this.studentName =undefined ,
            this.courseDate =null;
            this.reloadAll(1)
        }
    },
    // 保存
    reserveFun(obj) {
      if (obj.subject == '' || obj.teacher == null) {
        this.$message.error('修改内容不能为空')
        return
      }
      let timeRange = obj.timeRange
      let start = timeRange.split('-')[0].trim()
      let end = timeRange.split('-')[1].trim()
      if (obj.start != undefined) {
        start = obj.start.format('HH:mm')
      }
      if (obj.end != undefined) {
        end = obj.end.format('HH:mm')
      }
      obj.timeRange = start + ' - ' + end
      let json = JSON.parse(obj.teacher);
      obj.teacher = json.teacher;
      obj.teacherId = json.teacherId;
      obj.courseDate = moment(obj.courseDate).format('YYYY-MM-DD');
      delete obj.showUpdate;
      delete obj.subjectList;
      this.axios.post('pc/coursePlan/saveOrUpdatePlan', obj).then((res) => {
        if (res.success && res.result) {
          this.reloadAll(1,10);
          this.$message.info(res.message);
        } else {
          this.$message.error('保存失败');
        }
      })
    },
    // 修改
    update(obj) {
      obj.showUpdate = false;
    },
    selectStudent(obj) {
       this.pageNo =1
      this.studentId = obj
      this.reloadAll(1)
    },
    changeDate(date) {
        this.pageNo =1
      if(date!=null){
          this.courseDate = date.format('YYYY-MM-DD')
      }
      this.reloadAll(1)
    },
    getTeacher(obj) {
      let data = {
        subject: obj.subject,
        dueDate: moment(obj.courseDate).format('YYYY-MM-DD')
      }
      this.axios.post('pc/teacherInfo/queryTeacherIsWorkToday', data).then((res) => {
        this.teacherList = res.result
      })
    },
    reloadAll(current,pageSize) {
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        studentId: this.studentId,
        showSubject:false,
        courseDate: this.courseDate
      }
      this.spinning=true;
      this.axios.post('pc/coursePlan/listByPage', data).then((res) => {
        console.log(res);
           if(res.success){
               if(res.result.records.length!=0){
                  this.pageNo = res.result.current
                  this.total = res.result.total
                    this.data = []
                let items = res.result.records
                      items.forEach((item,i)=>{
                        item.showUpdate = true
                        item.subjectList = item.subjectList.split(',')
                        item.index = i
                        this.data.push(item)
                      }) 
                 
                  
               }else{
                  this.$message.info('暂无数据');
               }
           }
           this.spinning=false
      })
    }
  },
  mounted() {
    this.axios.post('pc/courseBill/listAll').then((res) => {
      this.studentList = res.result
    })
    this.reloadAll(1)
  }
}
</script>
<style scoped>
.loading {
  position: absolute;
  top: 50%;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
}

::v-deep .ant-btn-lg {
  height: 33px;
}

::v-deep .ant-input-group .ant-input {
  height: 33px;
}

::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>

