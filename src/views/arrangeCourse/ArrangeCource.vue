<template>
  <div class="cark1">
    <!--  表格的内容 -->
    <div style="display: flex; justify-content: space-between; margin: 15px 0px">
      <a-button type="primary" @click="loadInfo">突发排课</a-button>
      <a-drawer
        title="突发排课"
        placement="right"
        width="400"
        :closable="false"
        :visible="visible"
        @close="visible = false"
      >
        <div>
          <span style="vertical-align: middle; padding-right: 8px; color: #999">是否是新生</span>
          <a-switch
            checked-children="是"
            un-checked-children="否"
            @click="getStudent"
            default-checked
            v-model="newBirth"
          />
        </div>
        <table style="margin-top: 20px; width: 100%">
          <tr>
            <td>
              <b>学生:</b>
            </td>
            <td>
              <a-select style="width: 280px" v-if="!newBirth" v-model="student" placeholder="请选择学生">
                <a-select-option v-for="item of studentList" :key="item.sysUserId" :value="item">
                  {{ item.name }}
                </a-select-option>
              </a-select>
              <a-input v-else style="width: 280px" v-model="student" placeholder="请选输入学生姓名"></a-input>
            </td>
          </tr>
          <tr>
            <td>
              <b>产品: </b>
            </td>
            <td>
              <a-select style="width: 280px" v-model="product" placeholder="请选择产品">
                <a-select-option v-for="item in productList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <b>科目:</b>
            </td>
            <td>
              <a-select style="width: 280px" v-model="subject" placeholder="请选择科目" @select="getTeacher">
                <a-select-option v-for="item in subjectList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <b>老师:</b>
            </td>
            <td>
              <a-select style="width: 280px" v-model="teacher" placeholder="请选择老师">
                <a-select-option v-for="item in teacherList" :key="item.id" :value="item">
                  {{ item.name }}</a-select-option
                >
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <b>上课时间</b>
            </td>
            <td>
              <a-time-picker style="width: 127px; margin-left: 8px" v-model="startTime" format="HH:mm" />
              <span style="color: #000"> -</span>
              <a-time-picker style="width: 127px; margin-left: 8px" v-model="endTime" format="HH:mm" />
            </td>
          </tr>
        </table>
        <!-- 发布突发排课 -->
        <a-button type="primary" @click="submit" style="width: 98%; margin-top: 50px">发布</a-button>
      </a-drawer>
      <a-button @click="$router.push('/arrangeCourse/history')"> <a-icon type="history" />历史记录</a-button>
    </div>
    <a-spin :spinning="spinning">
      <a-table :columns="columns" :data-source="data" rowKey="id" bordered :pagination="false">
        <!-- 上课的时间 -->
        <a slot="classTime" slot-scope="classTime, record">
          <span style="color: #999"> {{ record.courseDate }}</span>
          <a-time-picker
            style="width: 80px; margin-left: 8px"
            :disabled="record.disabled"
            :default-value="moment(`${record.timeRange.split('-')[0]}`, 'HH:mm')"
            v-model="record.start"
            format="HH:mm"
          />
          <span style="color: #000"> -</span>
          <a-time-picker
            style="width: 80px; margin-left: 8px"
            :disabled="record.disabled"
            :default-value="moment(`${record.timeRange.split('-')[1]}`, 'HH:mm')"
            v-model="record.end"
            format="HH:mm"
          />
        </a>
        <!-- 操作按钮 -->
        <a slot="action" slot-scope="action, record">
          <div style="display: flex">
            <a-popconfirm title="确定发布吗?" ok-text="是" cancel-text="否" @confirm="issueOk(record)">
              <a-button type="primary" class="btn"><a-icon type="export" />发布</a-button>
            </a-popconfirm>
            <a-button @click="record.disabled = false" class="btn"> <a-icon type="edit" />修改</a-button>
            <a-button
              @click="save(record)"
              style="margin-left: 5px; color: #fff; background-color: #52c41a; height: 27px"
            >
              <a-icon type="file" />保存
            </a-button>
            <!--<a-popconfirm title="确定删除吗?" ok-text="是" cancel-text="否" @confirm="remove(record)">
            <a-button type="danger" class="btn"> <a-icon type="close" />删除</a-button>
          </a-popconfirm>-->
          </div>
        </a>
      </a-table>
    </a-spin>
    <a-pagination
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
      show-size-changer
      @showSizeChange="changePageSize"
      @change="reloadAll(pageNo)"
    />
    <a-modal v-model="conflictShow" :footer="null" :closable="false">
      <div class="title" v-show="clash.length == 1">
        <span style="margin-left: 5px"> ※ 当前时间段与“ {{ clash[0].student }} ”已排课冲突</span>
      </div>
      <a-divider />
      <a-button type="primary" style="margin-top: 10px"><a-icon type="history" />查询老师占用时间</a-button>
      <table style="width: 100%; margin-left: 10px; margin-top: 10px">
        <tr style="border-bottom: 1px" v-for="(item, index) of clash" :key="index">
          <td>{{ item.student }}</td>
          <td>{{ item.courseDate }} {{ item.timeRange }}</td>
          <td style="color: #1b8fff">{{ item.productName }}</td>
          <td>
            <a-tag color="#1B8FFF"> {{ item.subject }}</a-tag>
          </td>
        </tr>
      </table>
      <div style="text-align: right; margin-top: 50px" @click="conflictShow = false">
        <a-button> 知道了!</a-button>
      </div>
    </a-modal>
  </div>
</template>
<script>
import moment from 'moment'
import LogoVue from '../../components/tools/Logo.vue'
export default {
  data() {
    return {
      columns: [
        {
          title: '姓名',
          dataIndex: 'student',
          width: '120px',
        },
        {
          title: '上课时间',
          width: '350px',
          scopedSlots: { customRender: 'classTime' },
          dataIndex: 'courseDate',
        },
        {
          title: '产品',
          width: '120px',
          dataIndex: 'productName',
        },
        {
          title: '科目',
          width: '120px',
          dataIndex: 'subject',
          key: 'subject',
        },
        {
          title: '老师',
          width: '120px',
          dataIndex: 'teacher',
        },
        // {
        //   title: '教室',
        //    width:"150px",
        //   dataIndex: 'classroom',
        //   scopedSlots: { customRender: 'classroom' },
        // },
        {
          title: '操作',
          key: 'action',
          dataIndex: 'action',
          scopedSlots: { customRender: 'action' },
        },
      ],
      data: [],
      // 突发排课
      visible: false, //突发弹出框
      newBirth: false, //滑块是否是新生
      student: undefined, //学生
      subject: undefined, //科目
      product: undefined, //产品
      teacher: undefined, //老师
      startTime: undefined, //开始时间
      endTime: undefined, //结束时间
      conflictShow: false, //冲突的按钮
      pageNo: 1,
      total: 0,
      clash: [
        {
          student: '',
        },
      ], //对象
      subjectList: [],
      studentList: [],
      teacherList: [],
      productList: [],
      spinning: false,
      pageSize: 10,
    }
  },
  methods: {
    moment,
    // 删除
    /*remove(res) {
      this.data.forEach((item, i) => {
        if (item.name == res.name) {
          this.$message.success('删除成功')
          this.data.splice(i, 1)
        }
      })
    },*/
    // 页码数量
    changePageSize(current, size) {
      this.pageSize = size
      this.reloadAll(current)
    },
    submit() {
      let data = {
        productName: this.product,
        subject: this.subject,
        timeRange: this.startTime.format('HH:mm') + ' - ' + this.endTime.format('HH:mm'), //时间段
        courseDate: moment(new Date()).format('YYYY-MM-DD'),
        teacher: this.teacher.name,
        teacherId: this.teacher.sysUserId,
      }
      if (this.newBirth) {
        data.student = this.student
      } else {
        data.student = this.student.name
        data.studentId = this.student.sysUserId
      }
      this.axios.post('pc/coursePlan/publish', data).then((res) => {
        if (res.success) {
          this.$message.info(res.message)
          this.visible = false
        } else {
          this.$message.error(res.message)
          if (res.result != null) {
            this.conflictShow = true
            this.clash.push(res.result)
          }
        }
      })
    },
    // 保存
    save(obj) {
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
      delete obj.disabled
      this.axios.post('pc/coursePlan/saveOrUpdatePlan', obj).then((res) => {
        if (res.success && res.result) {
          this.reloadAll(1)
          this.$message.info(res.message)
        } else {
          this.$message.error('保存失败')
        }
      })
    },
    // 发布
    issueOk(res) {
      this.clash = []
      this.axios.post('pc/coursePlan/publish', res).then((res) => {
        if (res.success) {
          this.$message.info(res.message)
          this.reloadAll(1)
        } else {
          this.$message.error(res.message)
          if (res.result != null) {
            this.conflictShow = true
            this.clash.push(res.result)
          }
        }
      })
    },
    reloadAll(current) {
      let data = {
        pageNo: current,
        pageSize: 10,
        isNotNull: true,
      }
      this.spinning = true
      this.axios.post('pc/coursePlan/listByPage', data).then((res) => {
        if (res.success) {
            if(res.result.records.length){
          let items = res.result.records
              this.data = []
              for (let item of items) {
                item.disabled = true
                this.data.push(item)
              }
              this.pageNo = res.result.current
              this.total = res.result.total
            }else{
                 this.$message.info("暂无数据")
            }
        }
         this.spinning = false
      })
    },
    getTeacher(subject) {
      let data = {
        subject: subject,
        dueDate: moment(new Date()).format('YYYY-MM-DD'),
      }
      this.axios.post('pc/teacherInfo/queryTeacherIsWorkToday', data).then((res) => {
        this.teacherList = res.result
      })
    },
    getStudent() {
      this.student = ''
      if (!this.newBirth && this.studentList.length == 0) {
        this.axios.post('pc/studentInfo/listAll').then((res) => {
          this.studentList = res.result
        })
      }
    },
    loadInfo() {
      this.visible = true
      this.getStudent()
      if (this.subjectList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '科目' }).then((res) => {
          for (let category of res.result) {
            this.subjectList.push(category.title)
          }
        })
      }
      if (this.productList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '产品' }).then((res) => {
          for (let category of res.result) {
            this.productList.push(category.title)
          }
        })
      }
    },
  },
  mounted() {
    this.reloadAll(1)
  },
}
</script>
<style scoped>
.title {
  background-color: #ff4d4f;
  color: #fff;
  height: 43px;
  font-size: 16px;
  line-height: 43px;
  padding-left: 5px;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  padding-bottom: 10px;
  border-bottom: 1px solid #999;
}
.btn {
  height: 27px;
  margin-left: 5px;
}
tr {
  height: 50px;
}
::v-deep .ant-modal-title {
}
::v-deep .ant-modal-header {
}
::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>