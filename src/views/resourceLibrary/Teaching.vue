<template>
  <div class="cark1">
    <!--  -->
    <div style="margin: 20px 0px">
      <!-- 只看我的上传 -->
      <a-checkbox v-model="watchMineOnly" @change="reloadAll(1)" style="margin-left: 30px">
        <span style="color: #999">只看我的</span>
      </a-checkbox>
      <!-- 上传弹出框  -->
      <a-button type="primary" @click="visibleUpload = true" style="margin-left: 30px"> 新增上传</a-button>
      <!-- 上传弹出框  -->
      <a-drawer
        placement="right"
        :closable="false"
        width="400px"
        :visible="visibleUpload"
        @close="visibleUpload = false"
      >
        <div style="margin-left: -15px; margin-bottom: 15px">
          <sidebar>知识点类型</sidebar>
          <j-tree-select
            style="width: 100%; margin-top: 15px; margin-left: 15px"
            ref="treeSelect"
            v-decorator="['pid', validatorRules.pid]"
            dict="sys_category,name,id"
            pidField="pid"
            v-model="foreignKey"
            pidValue="1320927942363590657"
            searchPlaceholder="请输入知识点"
            placeholder="请选择知识点"
          >
          </j-tree-select>
        </div>
        <div style="margin-left: -15px; margin-bottom: 15px">
          <sidebar>上传类型</sidebar>
          <a-select style="width: 100%; margin-top: 15px; margin-left: 15px" placeholder="请选择上传类型" v-model="keyName">
            <a-select-option v-for="item in keyNameList" :key="item" :value="item"> {{ item }}</a-select-option>
          </a-select>
        </div>
        <div style="margin-left: -15px; margin-bottom: 15px">
          <sidebar>上传</sidebar>
        </div>
        <a-upload
          :remove="removeFile"
          :before-upload="beforeUpload"
          list-type="picture-card"
          :file-list="fileList"
          :headers="token"
        >
          <div v-if="fileList.length < 8">
            <a-icon type="plus" />
            <div class="ant-upload-text">上传</div>
          </div>
        </a-upload>
        <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible = false">
          <img alt="example" style="width: 100%" :src="previewImage" />
        </a-modal>
        <div style="text-align: center; margin-right: 5px; margin-top: 130px">
          <a-button type="primary" @click="submit" style="width: 100%"> 确认</a-button>
        </div>
      </a-drawer>
    </div>
 <a-spin :spinning="spinning">
    <!-- 表单的内容 -->
    <a-table
      :columns="columns"
      :data-source="data"
      bordered
      rowKey="id"
      :pagination="false"
    >
      <!-- 操作 -->
      <a slot="action" slot-scope="text,record">
        <a href="#" @click="preview(record)">{{ text }} 预览</a>
        <a :href="previewUrl+record.realFileName" style="margin-right: 8px"> 下载</a>
        <a href="#" @click="deleteItem(record)">删除</a>
        <a href="#" style="margin-right: 8px"> 分享</a>
        <!-- 查看的模态框 -->
        <a-modal :title="checkedName" width="60%" :visible="checked" :footer="null" @cancel="checked=false">
          <div style="text-align:center;"><img style="width: 100%" :src="previewUrl+record.realFileName"></div>
        </a-modal>
      </a>
      <a slot="foreignKey" slot-scope="subject, record">
        <j-tree-select
          ref="treeSelect"
          v-decorator="['pid', validatorRules.pid]"
          dict="sys_category,name,id"
          pidField="pid"
          v-model="record.foreignKey"
          :disabled="true"
          pidValue="1320927942363590657"
          searchPlaceholder="请输入知识点"
          placeholder="请选择知识点"
        >
        </j-tree-select>
      </a>
    </a-table>
        </a-spin>
    <!-- 分页 -->
      <div style="text-align:right;">
         <a-pagination
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
      @change="reloadAll(pageNo)"
    />
      </div>
  </div>
</template>

<script>
import JTreeSelect from '@comp/jeecg/JTreeSelect'
import Sidebar from '../../components/sidebar'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import CaseInfo from '@comp/jeecg/caseInfo'

function getBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = (error) => reject(error)
  })
}

export default {
  components: { CaseInfo, JTreeSelect, Sidebar },
  data() {
    return {
      columns: [
        {
          dataIndex: 'fileName',
          title: '名称'
        },
        {
          title: '文件后缀',
          dataIndex: 'fileSuffix'
        },

        {
          title: '类型',
          dataIndex: 'keyName'
        },
        {
          title: '文件大小',
          dataIndex: 'fileSize',
          customRender:(text, record)=>{
            let size1 = (record.fileSize/1024).toFixed(2);
            let size2 = (record.fileSize/1024/1024).toFixed(2);
            if(size1<1){
              return record.fileSize+"Byte";
            }
            else if (size1>=1 && size1<1024){
              return size1+"KB";
            }else if (size1>=1024 && size2<1024){
              return size2+"MB";
            }else if (size2>=1024){
              return (size2/1024).toFixed(2)+"GB";
            }
          }
        },
        {
          title: '使用次数',
          dataIndex: 'useTimes'
        },
        {
          title: '上传人',
          dataIndex: 'createBy'
        },
        {
          title: '知识点',
          dataIndex: 'foreignKey',
          scopedSlots: { customRender: 'foreignKey' }
        },
        {
          title: '操作',
          key: 'action',
          dataIndex: 'action',
          scopedSlots: { customRender: 'action' }
        }
      ],
      data: [],
      // 筛选章节
      validatorRules: {
        code: {
          rules: [
            {
              required: true,
              message: '请输入类型编码!'
            },
            {
              validator: this.validateMyCode
            }
          ]
        },
        pid: {},
        name: { rules: [{ required: true, message: '请输入类型名称!' }] }
      },
      //  删除 loading
      removeLoading: false,
      loading: false,

      //   上传
      visibleUpload: false,
      //上传文件
      previewVisible: false,
      previewImage: '',
      fileList: [],
      checked: false, //查看
      checkedName: '',
      watchMineOnly: false,//只看我的
      pageNo: 1,
      total: 0,
      foreignKey: '',//知识点ID
      keyNameList: [],
      keyName: '',//上传类型
      token: { 'X-Access-Token': this.$ls.get(ACCESS_TOKEN) },
      uploading: false,
      previewUrl: window._CONFIG['domianURL'] + '/sys/common/static/',
      spinning:false
    }
  },
  methods: {
    beforeUpload(file) {
       
      this.fileList = [...this.fileList, file]
       console.log(this.fileList);
      return false
    },
    removeFile(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    preview(res) {
      this.checked = true
      this.checkedName = res.fileName
    },
    //删除
    deleteItem(res) {
      let $this = this
      this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.axios.post('sys/file/delete', res).then((res) => {
            if (res.success && res.result) {
              $this.$message.info(res.message)
              $this.reloadAll(1)
            } else {
              $this.$message.error(res.message)
            }
          })
        }
      })
    },
    submit() {
      if (this.foreignKey == '' || this.keyName == '' || this.fileList.length == 0) {
        this.$message.error('以上内容不能为空')
        return
      }
      this.uploading = true
      const formData = new FormData()
      this.fileList.forEach(file => {
        formData.append('file', file)
      })
      formData.append('keyName', this.keyName)
      formData.append('foreignKey', this.foreignKey)
      this.axios.post('sys/file/uploadResources', formData).then((res) => {
        if (res.success) {
          this.uploading = false
          this.$message.info(res.message + '上传成功')
          this.reloadAll(1)
          this.visibleUpload = false;
        } else {
          this.uploading = false
          this.$message.error('上传失败')
          this.visibleUpload = false;
        }
      })
    },
    reloadAll(current) {
      this.data = []
      let data = {
        pageNo: current,
        pageSize: 10,
        watchMineOnly: this.watchMineOnly,
        foreignKey: this.foreignKey,
        keyName: this.keyName
      }
      this.spinning=true
      console.log(data);
      this.axios.post('sys/file/listByPage', data).then((res) => {
        console.log(res);
         if(res.success){
                  for (let item of res.result.records) {
              this.data.push(item)
            }
            this.pageNo = res.result.current
            this.total = res.result.total
         }
          this.spinning=false;
      })
    }
  },
  mounted() {
    this.axios.post('vx/coursePublish/findCategoryByName', { name: '上传类型' }).then((res) => {
      for (let category of res.result) {
        this.keyNameList.push(category.title)
      }
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
</style>