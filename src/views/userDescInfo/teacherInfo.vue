<template>
  <div class="cark1">
    <div style="margin-bottom: 16px">
      <!-- 清空选中的 -->
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="deleteBatch"> 批量删除</a-button>
      <a-button type="primary" style="margin-left: 20px" @click="DrawerFun('add')">
        <a-icon type="plus-circle" />
        新增老师信息
      </a-button>
    </div>
    <!--  表格的内容 -->
    <a-spin :spinning="spinning">
      <a-table
        :pagination="false"
        :columns="columns"
        :data-source="data"
        bordered
        rowKey="id"
        :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
      >
        <!--操作页面 -->
        <span slot="action" slot-scope="text, record">
          <div style="display: flex">
            <a-button type="primary" size="small" @click="DrawerFun('update', record)">
              <a-icon type="edit" /> 修改
            </a-button>
            <a-divider type="vertical" />
            <a-button type="danger" size="small" @click="deleteItem(record.id)">
              删除
              <a-icon type="delete" />
            </a-button>
          </div>
        </span>
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
    <!-- 弹出框 -->
    <a-drawer
      :title="teacherName == undefined ? '新增老师' : '修改'"
      :closable="false"
      :visible="showDrawer"
      @close="showDrawer = false"
      width="450px"
    >
      <sidebar> 基本信息</sidebar>
      <div style="margin: 15px">
        <table class="add">
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">姓名: </span>
            </td>
            <td>
              <div style="width: 230px; margin-left: 15px" v-if="statusBtn == 'update'">
                {{ teacherName }}
              </div>
              <a-select
                v-else
                @change="changeTeacher"
                style="width: 230px; margin-left: 15px"
                :disabled="showdetails"
                placeholder="请选择老师"
              >
                <a-select-option v-for="item in teacherList" :key="item.value" :value="JSON.stringify(item)">
                  {{ item.label }}
                </a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">类别 :</span>
            </td>
            <td>
              <a-select
                style="width: 230px; margin-left: 15px"
                @change="changeCategory"
                v-model="category"
                placeholder="请选择类别"
              >
                <a-select-option v-for="item in categoryList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">出勤星期 :</span>
            </td>
            <td>
              <a-select style="width: 230px; margin-left: 15px" v-model="week" mode="tags" placeholder="请选择星期">
                <a-select-option v-for="item in weekList" :key="item.value" :value="item.value">
                  {{ item.text }}
                </a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">科目 :</span>
            </td>
            <td>
              <a-select
                style="width: 230px; margin-left: 15px"
                @change="changeSubject"
                v-model="subject"
                mode="tags"
                placeholder="请选择科目"
              >
                <a-select-option v-for="item in subjectList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">教师资格证:</span>
            </td>
            <td>
              <a-switch checked-children="有" un-checked-children="无" default-checked v-model="qualified" />
            </td>
          </tr>
          <tr v-show="qualified">
            <td>
              <span style="margin-left: 5px">资格证编号:</span>
            </td>
            <td>
              <a-input
                style="margin-left: 15px; width: 230px"
                v-model="qualifiedNumber"
                placeholder="请输入资格证编号"
              />
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">毕业院校:</span>
            </td>
            <td>
              <a-input style="margin-left: 15px; width: 230px" v-model="school" placeholder="请输入毕业院校" />
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">学历:</span>
            </td>
            <td>
              <a-select style="width: 230px; margin-left: 15px" v-model="education" placeholder="请选择学历">
                <a-select-option value="统招本科"> 统招本科 </a-select-option>
                <a-select-option value="专科"> 专科 </a-select-option>
                <a-select-option value="自考本科"> 自考本科 </a-select-option>
                <a-select-option value="研究生"> 研究生 </a-select-option>
                <a-select-option value="博士"> 博士 </a-select-option>
                <a-select-option value="其他"> 其他 </a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">有效时间:</span>
            </td>
            <td style="text-align: right">
              <a-button type="primary" @click="plus"> <a-icon type="plus" /></a-button>
            </td>
          </tr>
          <tr v-for="(item, index) in timeList" :key="index">
            <td>
              <div v-if="item.index != 0">
                <a-button type="danger" @click="minus(index)">
                  <a-icon type="minus" />
                </a-button>
              </div>
            </td>
            <td>
              <a-time-picker v-model="item.start" style="margin-left: 15px; width: 110px" format="HH:mm" />
              -
              <a-time-picker v-model="item.end" style="margin-left: 5px; width: 110px" format="HH:mm" />
            </td>
          </tr>
        </table>
      </div>
      <a-button type="primary" @click="submit" style="width: 98%; margin-top: 50px">确定</a-button>
    </a-drawer>
  </div>
</template>
<script>
const columns = [
  {
    title: '老师',
    dataIndex: 'name',
  },
  {
    title: '科目',
    dataIndex: 'subject',
  },
  {
    title: '有效时间',
    dataIndex: 'timeRange',
  },
  {
    title: '类别',
    dataIndex: 'type',
  },
  {
    title: '有效星期',
    dataIndex: 'week',
  },
  {
    title: '毕业学校',
    dataIndex: 'school',
  },
  {
    title: '操作',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' },
  },
]
import moment from 'moment'
import Sidebar from '../../components/sidebar.vue'
export default {
  components: { Sidebar },
  data() {
    return {
      data: [],
      week: [],
      weekList: [
        { text: '星期一', value: '1' },
        { text: '星期二', value: '2' },
        { text: '星期三', value: '3' },
        { text: '星期四', value: '4' },
        { text: '星期五', value: '5' },
        { text: '星期六', value: '6' },
        { text: '星期日', value: '7' },
      ],
      id: '',
      selectedRowKeys: [], // 选中的对象
      loading: false,
      columns, //表头的内容
      showDrawer: false,
      name: '',
      // 基本信息
      teacherList: [],
      teacherName: '', //老师姓名
      teacherId: '',
      category: '', //类型
      categoryList: ['全职', '兼职'],
      school: '', //毕业院校
      qualified: false, //是否取得教师资格证
      qualifiedNumber: '', //资格证编号
      education: '',
      subjectList: [],
      subject: [], //科目
      start: '',
      end: '',
      time: '', //有效时间
      showdetails: false,
      statusBtn: '', //按钮的状态
      //  删除 loading
      removeLoading: false,
      pageNo: 1,
      total: 100,
      timeIndex: 0,
      timeList: [
        {
          index: 0,
          start: '',
          end: '',
        },
      ],
      spinning: false,
      pageSize: 10,
    }
  },

  computed: {
    //清空
    hasSelected() {
      return this.selectedRowKeys.length > 0
    },
  },
  methods: {
    // 页码数量
    changePageSize(current, size) {
      this.pageSize = size
      this.reloadAll(current)
    },
    //删除
    deleteItem(obj) {
      let $this = this
      this.$confirm({
        title: '确认删除?',
        onOk() {
          let id = []
          id.push(obj)
          $this.axios.post('pc/teacherInfo/deleteBatch', id).then((res) => {
            if (res.success && res.result) {
              $this.$message.info(res.message)
              $this.reloadAll(1)
            } else {
              $this.$message.error('删除失败')
            }
          })
        },
      })
    },
    moment,
    //类别
    changeCategory(value) {
      this.category = value
    },
    //科目
    changeSubject(value) {
      this.subject = value
    },
    plus() {
      this.timeList.push({
        index: ++this.timeIndex,
        start: '',
        end: '',
      })
    },
    minus(index) {
      --this.timeIndex
      this.timeList.splice(index, 1)
    },
    //修改和新增弹窗按钮
    DrawerFun(type, obj) {
      this.statusBtn = type
      this.loadAddInfo()
      this.showDrawer = true
      this.teacherName = type == 'add' ? undefined : obj.name
      this.teacherId = type == 'add' ? undefined : obj.teacherId
      this.timeList = []
      if (type != 'add') {
        let timeList = obj.timeRange.split(',')
        this.timeIndex = timeList.length
        for (let i = 0; i < this.timeIndex; i++) {
          let obj = {
            index: i,
            start: moment(timeList[i].split('-')[0], 'HH:mm'),
            end: moment(timeList[i].split('-')[1], 'HH:mm'),
          }
          this.timeList.push(obj)
        }
      } else {
        this.timeIndex = 0
        this.timeList.push({
          index: 0,
          start: '',
          end: '',
        })
      }

      this.school = type == 'add' ? undefined : obj.school
      this.category = type == 'add' ? undefined : obj.type
      this.week = type == 'add' ? undefined : obj.week.split(',')
      this.subject = type == 'add' ? undefined : obj.subject.split(',')
      this.id = type == 'add' ? undefined : obj.id
      this.qualified = type == 'add' ? undefined : obj.qualified
      this.education = type == 'add' ? undefined : obj.education
      this.qualifiedNumber = type == 'add' ? undefined : obj.qualifiedNumber

      // 禁用弹出框的内容
      this.showdetails = type != 'add'
    },
    // 批量删除
    deleteBatch() {
      let $this = this
      $this.loading = true
      this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.axios.post('pc/teacherInfo/deleteBatch', $this.selectedRowKeys).then((res) => {
            if (res.success && res.result) {
              $this.$message.info(res.message)
              $this.reloadAll(1)
            } else {
              $this.$message.error('删除失败')
            }
            $this.loading = false
          })
        },
        onCancel() {
          $this.loading = false
        },
      })
    },
    //  全选的对象
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    changeTeacher(obj) {
      let res =  JSON.parse(obj)
      this.teacherName = res.label
      this.teacherId = res.value
    },
    submit() {
      if (
        this.teacherName == '' ||
        this.school == '' ||
        this.category == '' ||
        this.subject == undefined ||
        (this.category == '兼职' && this.week == '')
      ) {
        this.$message.error('以上内容不能为空')
        return
      }
      let data = {
        name: this.teacherName,
        sysUserId: this.teacherId,
        school: this.school,
        subject: this.subject.join(','),
        type: this.category,
        qualified: this.qualified,
        education: this.education,
        qualifiedNumber: this.qualifiedNumber,
      }
      data.week = this.week.join(',')
      if (this.id != '' && this.statusBtn == 'update') {
        data.id = this.id
      }
      let timeRange = ''
      for (let time of this.timeList) {
        timeRange += time.start.format('HH:mm') + ' - ' + time.end.format('HH:mm') + ','
      }
      data.timeRange = timeRange.substr(0, timeRange.length - 1)
      this.axios.post('pc/teacherInfo/saveTeacherInfo', data).then((res) => {
        if (res.success && res.result) {
          this.$message.info(res.message)
          this.showDrawer = false
          this.reloadAll(1)
        } else {
          this.$message.error('保存失败')
          this.showDrawer = false
        }
      })
    },
    reloadAll(current) {
      this.spinning = true
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
      }
      this.axios.post('pc/teacherInfo/teacherInfoList', data).then((res) => {
        if (res.success) {
          if (res.result.records.length != 0) {
            this.data = res.result.records
            this.total = res.result.total
          } 
        }
        this.spinning = false
      })
    },
    loadAddInfo() {
      this.axios.post('pc/teacherInfo/queryTeacherNotWriteInfo').then((res) => {
        let result = res.result
        this.teacherList = []
        for (let item of result) {
          this.teacherList.push({ label: item.realname, value: item.id })
        }
      })
      if (this.subjectList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '科目' }).then((res) => {
          for (let category of res.result) {
            this.subjectList.push(category.title)
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
.cark1 {
  background-color: #fff;
  border-radius: 10px;
  padding: 10px 10px;
}

.flex_center {
  display: flex;
  align-items: center;
  margin-top: 10px;
}

::v-deep .ant-table-thead > tr > th {
  color: rgba(0, 0, 0, 0.57);
  font-weight: 550;
}

.add tr {
  height: 60px;
}
</style>
<style>
</style>

