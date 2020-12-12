<template>
  <div class="cark1">
    <div style="margin-bottom: 10px; display: flex; align-items: center">
      <b>学生: &nbsp;</b>
      <a-select @select="selectStudent" v-model="studentName" style="width: 150px" placeholder="按姓名来搜索">
        <a-select-option v-for="item in studentList" :key="item.id" :value="item.studentId">
          {{ item.student }}
        </a-select-option>
      </a-select>
      <b style="margin-left: 20px">老师:&nbsp; </b>
      <a-select @select="selectTeacher" v-model="teacherName" style="width: 135px" placeholder="请选择老师">
        <a-select-option v-for="item in teacherList" :key="item.sysUserId" :value="item.sysUserId">
          {{ item.name }}
        </a-select-option>
      </a-select>
      <a-button type="primary" @click="resetBtn" style="margin-left: 20px"> 重置</a-button>
    </div>
    <!--  表格的内容 -->
    <a-spin :spinning="spinning">
      <a-table :columns="columns" rowKey="id" :pagination="false" :data-source="data" bordered>
        <!--   进度章节 -->
        <a slot="knowledgePointId" slot-scope="subject, record">
          <j-tree-select
            ref="treeSelect"
            v-decorator="['pid', validatorRules.pid]"
            dict="sys_category,name,id"
            pidField="pid"
            v-model="record.knowledgePointId"
            :disabled="record.showUpdate"
            pidValue="1320927942363590657"
            searchPlaceholder="请输入知识点"
            placeholder="请选择知识点"
            :multiple="true"
          >
          </j-tree-select>
        </a>
        <!-- 操作的页面 -->
        <span slot="action" slot-scope="text, record">
          <div style="display: flex">
            <a-button type="danger" size="small" @click="record.showUpdate = false">
              <a-icon type="edit" /> 修改
            </a-button>
            <a-button type="primary" style="margin-left: 5px" size="small" @click="save(record)"
              ><a-icon type="plus" /> 保存
            </a-button>
            <a-button type="primary" style="margin-left: 5px" size="small" @click="getFile(record)">
              <span v-if="record.file1 == '' || record.file1 == null"> <a-icon type="plus" /> 添加资料 </span>
              <span v-else> <a-icon type="search" /> 查看资料 </span>
            </a-button>
            <a-button style="margin-left: 5px" size="small" @click="getDesc(record.studentId)"
              ><a-icon type="search" /> 情况说明
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
    <a-modal
      :closable="false"
      @cancel="closeModal"
      @ok="closeModal"
      width="1500px"
      v-model="visibleUpload"
      title="添加资料"
      ok-text="确认"
      cancel-text="取消"
    >
      <a-select
        style="width: 135px"
        @change="reloadResources(1, foreignKeyIds)"
        v-model="keyName"
        placeholder="请选择类型"
      >
        <a-select-option value="讲义"> 讲义 </a-select-option>
        <a-select-option value="教案"> 教案 </a-select-option>
        <a-select-option value="作业"> 作业 </a-select-option>
      </a-select>
      <a-checkbox v-model="watchMineOnly" @change="reloadResources(1, foreignKeyIds)" style="margin-left: 30px">
        <span style="color: #999">只看我的</span>
      </a-checkbox>
      <a-table
        style="margin-top: 10px"
        :columns="columns1"
        rowKey="id"
        :pagination="false"
        :data-source="data1"
        bordered
      >
        <span slot="action1" slot-scope="text, record">
          <div style="display: flex">
            <a-button type="primary" style="margin-left: 5px" size="small" @click="updateItem(record)"
              ><a-icon type="plus" /> 确认
            </a-button>
          </div>
        </span>
      </a-table>
      <a-pagination
        :default-current="1"
        v-model="pageNo1"
        :total="total1"
        show-quick-jumper
          show-size-changer
        @showSizeChange="changePageSize"
        @change="reloadResources(pageNo1)"
      />
      <div style="display: flex">
        <span style="margin: 10px 15px">
          讲义
          <a-upload list-type="picture-card" :default-file-list="file2" disabled></a-upload>
        </span>
        <span style="margin: 10px 15px">
          教案
          <a-upload list-type="picture-card" :default-file-list="file1" disabled></a-upload>
        </span>
        <span style="margin: 10px 15px">
          作业
          <a-upload list-type="picture-card" :default-file-list="file3" disabled></a-upload>
        </span>
      </div>
    </a-modal>
    <!-- 查看情况说明弹出框 -->
    <a-drawer
      title="查看情况说明"
      placement="right"
      :closable="false"
      width="400px"
      :visible="checkedCondition"
      @close="checkedCondition = false"
    >
      <div style="margin-left: -16px">
        <sidebar>分数情况</sidebar>
      </div>
      <div v-if="studentInfo != null">
        <table style="text-align: center">
          <tr>
            <th>科目</th>
            <th>当前分数</th>
            <th>目标分数</th>
          </tr>
          <tr v-for="item of JSON.parse(studentInfo.score)" :key="item.subject">
            <td>{{ item.subject }}</td>
            <td>{{ item.now }}</td>
            <td>{{ item.expect }}</td>
          </tr>
        </table>
      </div>
      <div style="margin-left: -16px; margin-top: 20px; margin-bottom: 10px">
        <sidebar>要求和特点</sidebar>
      </div>
      <div v-if="studentInfo != null">
        <table>
          <tr>
            <td style="width: 25%"><b>学生特点:</b></td>
            <td>{{ studentInfo.characteristic }}</td>
          </tr>
          <tr>
            <td><b>家长要求:</b></td>
            <td>{{ studentInfo.parentsExpect }}</td>
          </tr>
          <tr>
            <td><b>对老师要求:</b></td>
            <td>{{ studentInfo.teacherExpect }}</td>
          </tr>
          <tr>
            <td><b>规划师要求:</b></td>
            <td>{{ studentInfo.arrangeExpect }}</td>
          </tr>
        </table>
      </div>
    </a-drawer>
  </div>
</template>
<script>
import moment from 'moment'
import { TreeSelect } from 'ant-design-vue'
import JTreeSelect from '@comp/jeecg/JTreeSelect'
import Sidebar from '../../components/sidebar'

function getBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = (error) => reject(error)
  })
}

export default {
  components: { JTreeSelect, Sidebar },
  data() {
    return {
      columns: [
        {
          title: '学生',
          width: '80px',
          dataIndex: 'student',
        },
        {
          title: '学期段',
          width: '70px',
          dataIndex: 'semester',
        },
        {
          title: '出勤日期',
          width: '120px',
          dataIndex: 'courseDate',
        },
        {
          title: '时间段',
          width: '120px',
          dataIndex: 'timeRange',
        },
        {
          title: '产品',
          width: '100px',
          dataIndex: 'productName',
        },
        {
          title: '科目',
          width: '70px',
          dataIndex: 'subject',
        },
        {
          title: '老师',
          width: '70px',
          dataIndex: 'teacher',
        },
        {
          title: '进度章节',
          dataIndex: 'knowledgePointId',
          width: '350px',
          scopedSlots: { customRender: 'knowledgePointId' },
        },
        {
          title: '操作',
          dataIndex: 'action',
          width: '350px',
          scopedSlots: { customRender: 'action' },
        },
      ],
      columns1: [
        {
          dataIndex: 'fileName',
          title: '名称',
        },
        {
          title: '文件后缀',
          dataIndex: 'fileSuffix',
        },

        {
          title: '类型',
          dataIndex: 'keyName',
        },
        {
          title: '使用次数',
          dataIndex: 'useTimes',
        },
        {
          title: '文件大小',
          dataIndex: 'fileSize',
          customRender: (text, record) => {
            let size1 = (record.fileSize / 1024).toFixed(2)
            let size2 = (record.fileSize / 1024 / 1024).toFixed(2)
            if (size1 < 1) {
              return record.fileSize + 'Byte'
            } else if (size1 >= 1 && size1 < 1024) {
              return size1 + 'KB'
            } else if (size1 >= 1024 && size2 < 1024) {
              return size2 + 'MB'
            } else if (size2 >= 1024) {
              return (size2 / 1024).toFixed(2) + 'GB'
            }
          },
        },
        {
          title: '知识点',
          dataIndex: 'foreignKeyName',
        },
        {
          title: '操作',
          key: 'action1',
          dataIndex: 'action1',
          scopedSlots: { customRender: 'action1' },
        },
      ],
      data: [],
      data1: [],
      id: '',
      studentList: [],
      studentId: '', //姓名
      studentInfo: null,
      teacherList: [],
      teacherId: '', //老师
      materialImg: false,
      topicImg: false,
      planImg: false,
      visibleUpload: false, //添加资料弹出框
      checkedCondition: false, //查看情况说明弹出框
      // 进度章节
      validatorRules: {
        code: {
          rules: [
            {
              required: true,
              message: '请输入类型编码!',
            },
            {
              validator: this.validateMyCode,
            },
          ],
        },
        pid: {},
        name: { rules: [{ required: true, message: '请输入类型名称!' }] },
      },
      //查看资料
      checkedMaterial: false, //查看资料的弹出框
      pageNo: 1,
      total: 0,
      pageNo1: 1,
      total1: 0,
      watchMineOnly: false,
      keyName: '',
      foreignKeyName: '',
      foreignKeyIds: '',
      file1: [],
      file2: [],
      file3: [],
      previewUrl: window._CONFIG['domianURL'] + '/sys/common/static/',
      spinning: false,
      studentName: undefined,
      teacherName: undefined,
      pageSize:10,
    }
  },
  methods: {
    moment,
    // 页码数量
    changePageSize(current, size) {
      this.pageSize = size
      this.reloadAll(current)
    },
    // 重置按钮
    resetBtn() {
      if (this.studentName != undefined || this.teacherName != undefined) {
        this.studentName = undefined
        this.teacherName = undefined
        this.studentId = ''
        this.teacherId = ''
        this.reloadAll(1)
      }
    },
    closeModal() {
      this.foreignKeyIds = ''
      this.visibleUpload = false
    },
    save(value) {
      if (value.knowledgePointId == null || value.knowledgePointId == '') {
        this.$message.error('知识点不能为空!')
        value.showUpdate = true
        return
      }
      this.axios.post('pc/coursePlan/saveOrUpdatePlan', value).then((res) => {
        if (res.success && res.result) {
          this.reloadAll(1)
          this.$message.info(res.message)
        } else {
          this.$message.error('保存失败')
        }
      })
    },
    updateItem(obj) {
      if (this.keyName == '') {
        this.$message.error('文件类型不能为空!')
        return
      }
      let data = {
        id: this.id,
        fileId: obj.id,
      }
      if (this.keyName == '教案') {
        data.file1 = obj.realFileName
      } else if (this.keyName == '讲义') {
        data.file2 = obj.realFileName
      } else {
        data.file3 = obj.realFileName
      }
      this.axios.post('pc/coursePlan/updatePlanAndSysFile', data).then((res) => {
        if (res.success && res.result) {
          this.$message.info('添加成功!')
        } else {
          this.$message.error('添加失败!')
        }
      })
    },
    getDesc(student) {
      this.axios.post('pc/studentInfo/findByStudentId', { studentId: student }).then((res) => {
        this.studentInfo = res.result
        this.checkedCondition = true
      })
    },
    getFile(obj) {
      this.foreignKeyIds = ''
      this.id = obj.id
      this.visibleUpload = true
      let count = 0
      this.reloadResources(1, obj.knowledgePointId)
      this.foreignKeyIds = obj.knowledgePointId
      this.file1 = []
      this.file2 = []
      this.file3 = []
      if (obj.file1 != null) {
        let ids = obj.file1.split(',')
        for (let item of ids) {
          this.file1.push({
            uid: ++count,
            name: item,
            status: 'done',
            url: this.previewUrl + item,
          })
        }
      }
      if (obj.file2 != null) {
        let ids = obj.file2.split(',')
        for (let item of ids) {
          this.file2.push({
            uid: ++count,
            name: item,
            status: 'done',
            url: this.previewUrl + item,
          })
        }
      }
      if (obj.file3 != null) {
        let ids = obj.file3.split(',')
        for (let item of ids) {
          this.file3.push({
            uid: ++count,
            name: item,
            status: 'done',
            url: this.previewUrl + item,
          })
        }
      }
    },
    selectStudent(obj) {
      this.studentId = obj
      this.pageNo = 1
      this.reloadAll(1)
    },
    selectTeacher(obj) {
       this.pageNo = 1
      this.teacherId = obj
      this.reloadAll(1)
    },
    reloadAll(current) {
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        studentId: this.studentId,
        courseDate: this.courseDate,
        showSubject: true,
        teacherId: this.teacherId,
      }
      this.spinning = true
      this.axios.post('pc/coursePlan/listByPage', data).then((res) => {
        if (res.success) {
          if (res.result.records.length != 0) {
            let items = res.result.records
            this.data = []
            for (let item of items) {
              item.showUpdate = true
              this.data.push(item)
            }
            this.pageNo = res.result.current
            this.total = res.result.total
          }else{
           
          }
        }
        this.spinning = false
      })
    },
    reloadResources(current, foreignKeyIds) {
      let data = {
        pageNo: current,
        pageSize: 5,
        watchMineOnly: this.watchMineOnly,
        foreignKeyIds: foreignKeyIds,
        keyName: this.keyName,
        foreignKeyName: this.foreignKeyName,
      }
      this.axios.post('sys/file/resourcesPage', data).then((res) => {
        this.data1 = res.result.records
        this.pageNo1 = res.result.current
        this.total1 = res.result.total
      })
    },
  },
  mounted() {
    this.axios.post('pc/courseBill/listAll').then((res) => {
      this.studentList = res.result
    })
    this.axios.post('pc/teacherInfo/listAll').then((res) => {
      this.teacherList = res.result
    })
    this.reloadAll(1)
  },
}
</script>

<style lang="less" scoped>
.filtrate {
  div {
    padding: 8px 0px;
    border-bottom: 1px solid #e5e5e5;

    &:nth-child(1) {
      border: none;
    }

    &:nth-child(1) {
      color: #999;
      font-size: 18px;
    }
  }
}

.projectSum {
  margin-bottom: 20px;

  span {
    &:nth-child(2) {
      display: inline-block;
      width: 60px;
      height: 30px;
      padding: 8px;
      background-color: #f5f5f5;
      line-height: 15px;
    }

    &:nth-child(1) {
      font-weight: bold;
    }
  }
}

.loading {
  position: absolute;
  top: 50%;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
}

table {
  width: 100%;
}

tr {
  height: 45px;
}

::v-deep .ant-btn-lg {
  height: 33px;
}

::v-deep .ant-input-group .ant-input {
  height: 33px;
}
</style>
<style scoped>
/* tile uploaded pictures */
.upload-list-inline >>> .ant-upload-list-item {
  float: left;
  width: 200px;
  margin-right: 8px;
}

.upload-list-inline >>> .ant-upload-animate-enter {
  animation-name: uploadAnimateInlineIn;
}

.upload-list-inline >>> .ant-upload-animate-leave {
  animation-name: uploadAnimateInlineOut;
}

::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>

