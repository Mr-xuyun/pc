<template>
  <div class="cark1">
    <!-- 顶部菜单 -->
    <div style="margin-bottom: 16px;justify-content: space-between;display: flex;">
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="sendBatch"> 批量通知</a-button>
      <a-button @click="$router.push('/arrangeCourse/history')"> <a-icon type="history" />历史记录</a-button>
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
    </a-table>
    <!-- 分页 -->
    <a-pagination
      show-size-changer
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
      @change="reloadAll"
    />
  </div>
</template>

<script>
import moment from 'moment';
export default {
  data() {
    return {
      //表头
      columns: [
        {
          title: '姓名',
          dataIndex: 'student',
          width: '120px',
        },
        {
          title: '上课时间',
          width: '150px',
          dataIndex: 'timeRange',
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
        },
        {
          title: '老师',
          width: '120px',
          dataIndex: 'teacher',
        },
        {
          title: '发布时间',
          width: '180px',
          dataIndex: 'createTime',
        }
      ],
      data: [],//表格数据
      ids: [],//被选中的对象
      loading:false,
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
    setIds(ids) {
      this.ids = ids
    },
    //批量通知
    sendBatch() {
      let this_ = this;
      this_.$confirm({
        title: '确认发布全部课程?',
        onOk() {
          this_.axios.post('pc/coursePlan/publishBatch',this_.ids).then((res) => {
            if (res.success && res.result) {
              this_.$message.info(res.message);
              this_.reloadAll();
            }else{
              this_.$message.error(res.message);
            }
          })
        }
      })
    },
    //加载表格数据
    reloadAll() {
      let data = {
        pageNo:  this.pageNo,
        pageSize: 10,
        status: 1,
      }
      this.axios.post('pc/coursePlan/listByPage', data).then((res) => {
        if (res.success) {
          for (let item of res.result.records) {
            item.createTime = moment(item.createTime).format('YYYY-MM-DD HH:mm')
          }
          this.data = res.result.records
          this.total = res.result.total
        }
      })
    },
  },
  mounted() {
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