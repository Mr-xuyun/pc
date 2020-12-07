<template>
  <div class="cark1">
    <div style="margin-bottom: 16px">
      <!-- 清空选中的 -->
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="deleteBatch"> 批量删除</a-button>
      <a-button type="primary" style="margin-left: 20px" @click="DrawerFun('add')">
        <a-icon type="plus-circle" />
        新增学员信息
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
        <a slot="name" slot-scope="text">{{ text }} </a>
        <!--操作页面 -->
        <span slot="action" slot-scope="text, record">
          <div style="display: flex">
            <a-divider type="vertical" />
            <a-button type="primary" size="small" @click="update(record)"> <a-icon type="edit" /> 修改 </a-button>
            <a-divider type="vertical" />
            <a-button type="danger" size="small" @click="deleteItem(record)"> <a-icon type="delete" /> 删除 </a-button>
            <a-divider type="vertical" />
            <a-button size="small" @click="consume"> <a-icon type="dollar" />查看消费</a-button>
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
      @showSizeChange="changePageSize"
      show-size-changer
      @change="reloadAll(pageNo)"
    />
    <!-- 弹出框 -->
    <a-drawer
      :title="studentName == undefined ? '新增学员' : studentName + '详细信息'"
      :closable="false"
      :visible="showDrawer"
      @close="showDrawer = false"
      width="800"
    >
      <sidebar> 基本信息</sidebar>
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 学生: </span>
          <div style="width: 80%; margin-left: 29px" v-if="statusBtn == 'update'">
            {{ studentName }}
          </div>
          <a-select
            v-else
            style="width: 80%; margin-left: 29px"
            @change="changeStudentName"
            :disabled="showdetails"
            placeholder="请输入学生"
          >
            <a-select-option v-for="item in studentList" :key="item.value" :value="JSON.stringify(item)">
              {{ item.label }}
            </a-select-option>
          </a-select>
        </div>
      </div>
      <!-- 学期段 -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 学期段:</span>
          <a-select
            style="width: 80%; margin-left: 15px"
            :disabled="showdetails"
            v-model="classTime"
            placeholder="请选择学期段"
          >
            <a-select-option v-for="item in semesterList" :key="item" :value="item"> {{ item }}</a-select-option>
          </a-select>
        </div>
      </div>
      <!-- 类别 -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 类别:</span>
          <a-select
            style="width: 80%; margin-left: 29px"
            :disabled="showdetails"
            v-model="category"
            placeholder="请选择类别"
          >
            <a-select-option v-for="item in categoryList" :key="item" :value="item"> {{ item }}</a-select-option>
          </a-select>
        </div>
      </div>
      <!-- 来源 -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 来源:</span>
          <a-select
            style="width: 80%; margin-left: 29px"
            :disabled="showdetails"
            v-model="source"
            placeholder="请选择来源"
          >
            <a-select-option v-for="item in sourceList" :key="item" :value="item"> {{ item }}</a-select-option>
          </a-select>
        </div>
      </div>
      <!-- 学校名称 -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 学校名称</span>
          <a-input
            style="margin-left: 5px; width: 80%"
            :disabled="showdetails"
            v-model="school"
            placeholder="请输入学校名称"
          />
        </div>
      </div>
      <div>
        <sidebar> 情况说明</sidebar>
      </div>
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 科目:</span>
          <a-select
            style="width: 75%; margin-left: 45px"
            :disabled="showdetails"
            v-model="subject"
            mode="tags"
            @select="changeSubject"
            @deselect="deselectSubject"
          >
            <a-select-option v-for="a in subjectList" :key="a + 11" :value="a"> {{ a }}</a-select-option>
          </a-select>
        </div>
      </div>
      <div v-if="situationMarkList.length > 0">
        <div v-for="(mark, i) of situationMarkList" :key="i">
          <!--分数情况 -->
          <div style="margin: 15px">
            <div class="flex_center">
              <a-tag color="#1B8FFF">
                {{ mark.subject }}
              </a-tag>
              <span style="margin: 0px 10px">当前分数</span>
              <a-input
                style="width: 20%; margin-right: 10px"
                v-model="mark.now"
                :disabled="showdetails"
                type="number"
                placeholder="请输入分数"
              />
              <span style="margin: 0px 10px">目标分数</span>
              <a-input
                style="width: 20%; margin-right: 10px"
                v-model="mark.expect"
                :disabled="showdetails"
                type="number"
                placeholder="请输入分数"
              />
            </div>
          </div>
        </div>
      </div>

      <!-- 学生特点  -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 学生特点 :</span>
          <a-input :disabled="showdetails" style="width: 75%; margin-left: 15px" v-model="StudentTrait"></a-input>
        </div>
      </div>
      <!-- 家长要求:  -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 家长要求:</span>
          <a-input style="width: 75%; margin-left: 19px" v-model="patriarchRequire" :disabled="showdetails"></a-input>
        </div>
      </div>
      <!-- 对老师要求::  -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 对老师要求:</span>
          <a-input :disabled="showdetails" style="width: 75%; margin-left: 5px" v-model="teacherRequire"></a-input>
        </div>
      </div>
      <!-- 规划师要求:  -->
      <div style="margin: 15px">
        <div class="flex_center">
          <img src="../../assets/星号.png" width="10" height="10" />
          <span> 规划师要求:</span>
          <a-input :disabled="showdetails" style="width: 75%; margin-left: 5px" v-model="plannerRequire"></a-input>
        </div>
      </div>
      <div v-show="statusBtn == 'add' || statusBtn == 'update'" style="text-align: right; margin-top: 15px">
        <a-button type="primary" @click="submit"> 确定</a-button>
      </div>
    </a-drawer>
  </div>
</template>
<script>
import Login from '@views/user/Login'

const columns = [
  {
    title: '姓名',
    dataIndex: 'name',
  },
  {
    title: '学期段',
    dataIndex: 'semester',
  },
  {
    title: '学校名称',
    dataIndex: 'schoolName',
  },

  {
    title: '学生类别',
    dataIndex: 'studentType',
  },
  {
    title: '学生来源',
    dataIndex: 'studentSource',
  },
  {
    title: '科目',
    dataIndex: 'subject',
  },
  {
    title: '操作',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' },
  },
]

import Sidebar from '../../components/sidebar.vue'

export default {
  components: { Sidebar },
  data() {
    return {
      id: '',
      data: [],
      selectedRowKeys: [], // 选中的对象
      loading: false,
      columns: columns, //表头的内容
      showDrawer: false,
      name: '',
      // 基本信息
      subject: [],
      studentList: [],
      studentName: '', //学生姓名
      studentId: '',
      classTime: '', //学期段
      semesterList: [],
      source: '', //来源
      sourceList: [],
      category: '', //类型
      categoryList: [],
      school: '', //学校名称
      StudentTrait: '', //学生特点
      patriarchRequire: '', //家长要求
      teacherRequire: '', // 老师要求
      plannerRequire: '', // 规划师的要求
      subjectList: [],
      situationMarkList: [],
      showdetails: false,
      statusBtn: '', //按钮的状态
      pageNo: 1,
      total: 100,
      spinning: false, //加载的状态图标
      pageSize:10
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
    changePageSize(current, size){
           this.pageSize =size
          this.reloadAll(current)
    },
    //修改
    update(obj) {
      console.log(obj.name);
      this.statusBtn = 'update'
      this.showDrawer = true
      this.id = obj.id
      this.studentName = obj.name
      this.studentList.push(obj.name)
      this.source = obj.studentSource
      this.classTime = obj.semester
      this.school = obj.schoolName
      this.category = obj.studentType
      this.StudentTrait = obj.characteristic
      this.patriarchRequire = obj.parentsExpect
      this.teacherRequire = obj.teacherExpect
      this.plannerRequire = obj.arrangeExpect
      this.subject = obj.subject.split(',')
      this.situationMarkList = JSON.parse(obj.score)
      this.showdetails = false
      this.loadAddInfo()
    },
    deleteItem(obj) {
      let $this = this
      this.$confirm({
        title: '确认删除?',
        onOk() {
          let id = []
          id.push(obj.id)
          $this.axios.post('pc/studentInfo/deleteBatch', id).then((res) => {
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
    // 查看消费情况
    consume() {},
    // 基本信息的选择器
    changeStudentName(obj) {
       let obj1= JSON.parse(obj)
      this.studentName = obj1.label
      this.studentId = obj1.value
    },
    changeSubject(obj) {
      let data = {
        subject: obj,
        now: '',
        expect: '',
      }
      this.situationMarkList.push(data)
    },
    deselectSubject(obj) {
      for (let i = 0; i < this.situationMarkList.length; i++) {
        if (this.situationMarkList[i].subject == obj) {
          this.situationMarkList.splice(i, 1)
        }
      }
    },
    //
    //详情和新增弹窗按钮
    DrawerFun(type, obj) {
      this.loadAddInfo()
      this.statusBtn = type
      this.showDrawer = true
      this.studentName = type == 'add' ? undefined : obj.name
      this.studentId = type == 'add' ? undefined : obj.studentId
      this.source = type == 'add' ? undefined : obj.studentSource
      this.classTime = type == 'add' ? undefined : obj.semester
      this.school = type == 'add' ? undefined : obj.schoolName
      this.category = type == 'add' ? undefined : obj.studentType
      this.StudentTrait = type == 'add' ? undefined : obj.characteristic
      this.patriarchRequire = type == 'add' ? undefined : obj.parentsExpect
      this.teacherRequire = type == 'add' ? undefined : obj.teacherExpect
      this.plannerRequire = type == 'add' ? undefined : obj.arrangeExpect
      this.subject = type == 'add' ? undefined : obj.subject.split(',')
      this.situationMarkList = type == 'add' ? [] : JSON.parse(obj.score)
      // 禁用弹出框的内容
      this.showdetails = type != 'add'
    },
    // 清空按钮
    deleteBatch() {
      let $this = this
      $this.loading = true
      this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.axios.post('pc/studentInfo/deleteBatch', $this.selectedRowKeys).then((res) => {
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
    submit() {
      if (
        this.subject == undefined ||
        this.studentName == '' ||
        this.classTime == '' ||
        this.school == '' ||
        this.category == '' ||
        this.source == '' ||
        this.StudentTrait == '' ||
        this.patriarchRequire == '' ||
        this.teacherRequire == '' ||
        this.plannerRequire == '' ||
        this.situationMarkList == undefined
      ) {
        this.$message.error('以上内容不能为空')
        return
      }
      this.showDrawer = false
      let data = {
        name: this.studentName,
        sysUserId: this.studentId,
        semester: this.classTime,
        schoolName: this.school,
        studentType: this.category,
        studentSource: this.source,
        characteristic: this.StudentTrait,
        parentsExpect: this.patriarchRequire,
        teacherExpect: this.teacherRequire,
        arrangeExpect: this.plannerRequire,
        subject: this.subject.join(','),
        score: JSON.stringify(this.situationMarkList),
      }
      if (this.id != '' && this.statusBtn == 'update') {
        data.id = this.id
      }
      this.axios.post('pc/studentInfo/saveStudentInfo', data).then((res) => {
        if (res.success && res.result) {
          this.$message.info(res.message)
          this.reloadAll(1)
        } else {
          this.$message.error('保存失败')
        }
      })
    },
    reloadAll(current) {
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
      }
      this.spinning = true
      this.axios.post('pc/studentInfo/studentInfoList', data).then((res) => {
        console.log(res);
        if (res.success) {
           if (res.result.records.length != 0) {
            this.data = res.result.records
            this.total = res.result.total
           } 
        } else {
              this.data = res.result.records
        }
        this.spinning = false
      })
    },
    loadAddInfo() {
      this.axios.post('pc/studentInfo/queryStudentNotWriteInfo').then((res) => {
        console.log(res);
        let result = res.result
        this.studentList = []
        for (let item of result) {
          this.studentList.push({ label: item.realname, value: item.id })
        }
      })
      if (this.semesterList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '学期段' }).then((res) => {
          for (let category of res.result) {
            this.semesterList.push(category.title)
          }
        })
      }
      if (this.categoryList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '学生类别' }).then((res) => {
          for (let category of res.result) {
            this.categoryList.push(category.title)
          }
        })
      }
      if (this.sourceList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '学生来源' }).then((res) => {
          for (let category of res.result) {
            this.sourceList.push(category.title)
          }
        })
      }
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
    this.reloadAll(1);

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

::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>
<style>
</style>

