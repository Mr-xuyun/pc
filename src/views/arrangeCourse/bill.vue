<template>
  <div class="cark1">
    <div style="margin-bottom: 16px">
      <!-- 清空选中的 -->
      <a-input-search placeholder="请输入学生名字或者产品名称" style="width:300px;" enter-button="查询" size="large" @search="onSearch" />
    </div>
      <a-spin :spinning="spinning">
    <!--  表格的内容 -->
    <a-table
      :columns="columns"
      :pagination="false"
      :data-source="data"
      rowKey="id"
      bordered
    >
      <span slot="action" slot-scope="text, record">
        <div style="display: flex">
          <a-button type="primary" size="small" @click="showPic(record)">
            <a-icon type="edit" /> 查看截图
          </a-button>
        </div>
      </span>
    </a-table>
       </a-spin>
    <!-- 分页 -->
    <a-pagination
      :default-current="1"
      v-model = "pageNo"
      :total="total"
      show-quick-jumper
      @change="reloadAll(pageNo)"
    />
    <a-drawer title="截图查看" :visible="showDrawer" @close="showDrawer = false" width="500px">
      <h1> 付款截图</h1>
      <a-upload list-type="picture-card" disabled :defaultFileList="payFile"></a-upload>
      <h1> 协议截图</h1>
      <a-upload list-type="picture-card" disabled :defaultFileList="contractFile"></a-upload>
    </a-drawer>
  </div>
</template>
<script>
import JTreeSelect from '@comp/jeecg/JTreeSelect'
import JTreeDict from '@comp/jeecg/JTreeDict'
export default {
  components: { JTreeDict, JTreeSelect },
  data() {
    return {
      columns: [
        {
          title: '学生',
          dataIndex: 'student',
        },
        {
          title: '产品',
          dataIndex: 'productName',
        },
        {
          title: '总价',
          dataIndex: 'sum',
        },
        {
          title: '实付',
          dataIndex: 'actual',
        },
        {
          title: '优惠',
          dataIndex: 'reduce',
        },
        {
          title: '付款时间',
          dataIndex: 'createTime',
        },
        {
          title: '操作',
          key: 'action',
          dataIndex: 'action',
          scopedSlots: { customRender: 'action' },
        },
      ],
      imgUrl:window._CONFIG['domianURL'] + "/sys/common/static/",
      showDrawer:false,
      payFile:[],
      contractFile:[],
      data: [],
      pageNo:1,
      total:0,
      spinning:false,
    }
  },
  methods: {
    //搜索
    onSearch(value) {
      this.reloadAll(1, value);
    },
    showPic(obj){
      this.showDrawer = true;
      this.payFile = [];
      this.contractFile = [];
      let payFiles = obj.payFile.split(",");
      let uid = 0;
      for (let file of payFiles) {
        let fileData = {
          uid:++uid,
          name:file,
          status:"done",
          url:this.imgUrl+file,
          thumbUrl:this.imgUrl+file
        }
        this.payFile.push(fileData);
      }
      let contractFile = obj.contractFile.split(",");
      for (let file of contractFile) {
        let fileData = {
          uid:++uid,
          name:file,
          status:"done",
          url:this.imgUrl+file,
          thumbUrl:this.imgUrl+file
        }
        this.contractFile.push(fileData);
      }
    },
    reloadAll(current,keyWords) {
      let data = {
        pageNo: current,
        pageSize: 10,
      }
      if (keyWords != ""){
        data.keyWords = keyWords.trim();
      }
      this.spinning =true
      this.axios.post('pc/courseBill/listByPage', data).then((res) => {
        console.log(res);
          if(res.success){
                 this.data = res.result.records;
                this.pageNo = res.result.current;
                this.total = res.result.total;
          }
           this.spinning =false
      })
    }
  },
  mounted() {
    this.reloadAll(1, "")
  }
}
</script>
<style scoped>

::v-deep .ant-pagination{
  text-align: right;
  margin-top: 10px;
}
</style>

