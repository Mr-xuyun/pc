<template>
  <div class="cark1">
    <!-- 顶部菜单 -->
    <div style="margin-bottom: 16px">
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="deleteBatch"> 批量删除</a-button>
      <label>测试:</label>
      <a-select style="width: 200px" @select="reloadAll(1)" v-model="teacherId">
        <a-select-option v-for="item in teacherList" :key="item.id" :value="item.sysUserId"> {{ item.name }}
        </a-select-option>
      </a-select>
    </div>
    <!--  表格的内容 -->
    <a-table
      :pagination="false"
      :columns="columns"
      :data-source="data"
      bordered
      rowKey="id"
      :row-selection="{ selectedRowKeys: ids, onChange: setIds }"
    >
      <!--操作页面 -->
      <span slot="action" slot-scope="text, record">
        <div style="display:flex;">
          <a-divider type="vertical" />
          <a-button type="primary" size="small" @click="showDrawer = true"> <a-icon type="search" /> 查看 </a-button>
          <a-divider type="vertical" />
          <a-button type="primary" size="small" @click="showModal = true"> <a-icon type="edit" /> 修改 </a-button>
          <a-divider type="vertical" />
          <a-button type="primary" size="small" @click="deleteItem(record)"> <a-icon type="delete" /> 删除 </a-button>
        </div>
      </span>

      <!-- 行编辑 -->
      <a slot="schoolName" slot-scope="schoolName, record">
        <a-input  v-model="record.schoolName" />
      </a>

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
    <!-- 抽屉 -->
    <a-drawer title="测试标题" :visible="showDrawer" @close="showDrawer = false" width="800px" :closable="false" >
      <a-form-model :model="formData" :label-col="labelCol" :wrapper-col="wrapperCol" :rules="formRules" ref="form">
        <a-form-model-item  prop="input" label="输入框">
          <a-input v-model="formData.input" placeholder="请输入" />
        </a-form-model-item>
        <a-form-model-item prop="textarea" label="文本域">
          <a-textarea v-model="formData.textarea" placeholder="请输入" :rows="4" />
        </a-form-model-item>
        <a-form-model-item label="日期框">
          <a-date-picker
            :defaultValue="moment(new Date(),'YYYY-MM-DD')"
            v-model="formData.date"
            format="YYYY-MM-DD"
            type="date"
            style="width: 100%;"
          />
        </a-form-model-item>
        <a-form-model-item label="范围日期">
          <a-range-picker
            v-model="formData.dateRange"
            :ranges="{
              '上月': [moment().subtract(1,'month').startOf('month'), moment().subtract(1,'month').endOf('month')],
              '本月': [moment().startOf('month'), moment().endOf('month')]
              }"
            format="YYYY-MM-DD"
          >
          </a-range-picker>

        </a-form-model-item>
        <a-form-model-item label="Checkbox">
          <a-checkbox value="A" :checked="true">
            A
          </a-checkbox>
        </a-form-model-item>
        <a-form-model-item label="数字框">
          <a-input-number v-model="formData.number" :min="1" :max="100000" :default-value="3"  />
        </a-form-model-item>
        <a-form-model-item prop="select" label="选择框">
          <a-select v-model="formData.select" style="width: 100%" placeholder="请选择">
            <a-select-option value="0" >A</a-select-option>
            <a-select-option value="1" >B</a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item prop="tags" label="多选">
          <a-select v-model="formData.tags" style="width: 100%" mode="tags" placeholder="请选择">
            <a-select-option value="0" >A</a-select-option>
            <a-select-option value="1" >B</a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item label="是否框">
          <a-switch v-model="formData.switch" checked-children="是" un-checked-children="否"  />
        </a-form-model-item>
        <a-form-model-item label="时间框">
          <a-time-picker v-model="formData.time" format="HH:mm" :defaultValue="moment(new Date())" />
        </a-form-model-item>
        <a-form-model-item label="文件上传">
          <a-upload v-model="formData.fileList" >
            <a-button> <a-icon type="upload" /> 点击上传 </a-button>
          </a-upload>
        </a-form-model-item>
        <a-form-model-item :wrapper-col="{ span: 18, offset: 3 }">
          <a-button @click="submit" type="primary" block>确认</a-button>
          <a-button @click="reset" type="danger" block>重置</a-button>
          <a-button @click="showDrawer = false" block>取消</a-button>
        </a-form-model-item>
      </a-form-model>
    </a-drawer>
    <!-- 中间弹出框 -->
    <a-modal
      :closable="true"
      @cancel="showModal = false"
      @ok="showModal = false"
      width="800px"
      v-model="showModal"
      title="测试标题"
      ok-text="确认"
      cancel-text="取消">
    </a-modal>

  </div>
</template>

<script>
//表头
const columns = [
  {
    title: '学生姓名',
    dataIndex: 'student',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.student
    }
  },
  {
    title: '学校名称',
    dataIndex: 'schoolName',
    //表格行可编辑
    scopedSlots: { customRender: 'schoolName' },
  },
  {
    title: '操作',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' }
  }
]
import moment from 'moment';
export default {
  data() {
    return {
      columns,//表头
      data: [
        {
          id: 0,
          student: '张三',
          schoolName: '清华大学'
        },
        {
          id: 1,
          student: '李四',
          schoolName: '北京大学'
        }
      ],//表格数据
      ids: [],//被选中的ID
      loading: false,//批量删除是否显示加载状态
      showDrawer: false,//是否弹出抽屉
      showModal: false,//是否弹出modal框
      teacherList: [],
      teacherId: '',
      productList: [],
      subjectList: [],
      //表单数据
      formData: {
        input: '',
        textarea: "",
        date: null,
        dateRange: [],
        number: 0,
        select: null,
        tags: [],
        switch:false,
        time: null,
        fileList: [],
      },
      //表单验证
      formRules:{
        input:[
          { required: true, message: '不能为空', trigger: 'blur' },
          { pattern: /^(13[0-9]|14[01456879]|15[0-3,5-9]|16[2567]|17[0-8]|18[0-9]|19[0-3,5-9])\d{8}$/, message: '手机号输入错误', trigger: 'blur' }
          ],
        textarea:[
          {required: true, message: '不能为空', trigger: 'blur'},
          {max: 10, message: '长度最长不能超过10', trigger: 'blur'}
        ],
        select: [
          {required: true, message: '不能为空', trigger: 'change'},
        ],
        tags: [
          {type: 'array',required: true, message: '最少选择一项', trigger: 'change'},
        ]
      },
      labelCol: { span: 4 },
      wrapperCol: { span: 18 },
      pageNo: 1,//当前页
      total: 100//总页数
    }
  },
  computed: {
    //是否禁用全选按钮
    hasSelected() {
      return this.ids.length > 0
    }
  },
  methods: {
    moment,
    setIds(ids) {
      this.ids = ids
    },
    //保存
    submit(){
      this.$refs.form.validate(valid => {
        if (valid) {
          this.$message.info('校验通过');
          console.log(this.formData)
        } else {
          this.$message.error('校验失败');
          return false;
        }
      });
    },
    //重置
    reset(){
      this.$refs.form.resetFields();
    },
    //删除
    deleteItem(obj) {
      let $this = this
      $this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.$message.info('删除成功')
        },
        onCancel() {
          $this.$message.info('取消删除')
        }
      })
    },
    //批量删除
    deleteBatch() {
      console.log(this.ids)
      let $this = this
      $this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.$message.info('删除成功')
        },
        onCancel() {
          $this.$message.info('取消删除')
        }
      })
    },
    //加载表格数据
    reloadAll(current) {
      let data = {
        pageNo: current,
        pageSize: 10
      }
      this.axios.post('', data).then((res) => {
        this.data = res.result.records
        this.pageNo = res.result.current
        this.total = res.result.total
      })
    },
    loadInfo() {
      if (this.teacherList.length != 0) {
        this.axios.post('pc/teacherInfo/listAll').then((res) => {
          this.teacherList = res.result
        })
      }
      if (this.productList.length != 0) {
        this.axios.post('pc/product/listAll', { delFlag: 0 }).then((res) => {
          this.productList = res.result
        })
      }
      if (this.subjectList.length != 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '科目' }).then((res) => {
          this.subjectList = res.result
        })
      }
    }
  },
  mounted() {
    //this.reloadAll(1)
    this.loadInfo()
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