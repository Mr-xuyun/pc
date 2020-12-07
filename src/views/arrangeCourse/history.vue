<template>
  <div class="cark1">
    <div style="margin-bottom: 20px">
      <b style="font-size: 16px">查询日期:</b>
      <a-range-picker @change="onChange" style="margin-left: 8px" />
    </div>
    <a-spin :spinning="spinning">
    <a-tabs default-active-key="1">
      <a-tab-pane key="1" tab="已确认" force-render>
        <a-table :columns="columns" :data-source="data" bordered rowKey="id" :pagination="false">
          <a slot="state">
            <span class="yuan" :style="{ backgroundColor: 'green' }"></span>
            <span style="color: green; margin-left:8px; }">已确认</span>
          </a>
        </a-table>
      </a-tab-pane>
      <a-tab-pane key="2" tab="未确认">
        <a-tab-pane key="1" tab="已确认" force-render>
          <a-table :columns="columns" :data-source="data" bordered rowKey="id">
            <a slot="state">
              <span class="yuan" :style="{ backgroundColor: 'red' }"></span>
              <span style="color: red; margin-left:8px; }">未确认</span>
            </a>
          </a-table>
        </a-tab-pane>
      </a-tab-pane>
    </a-tabs>
    </a-spin >
      <div style="text-align:right; margin-top:8px;">
          <a-pagination
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
        show-size-changer
        @showSizeChange="changePageSize"
      @change="reloadAll"
    />
      </div>
   
  </div>
</template>
<script>
import moment from 'moment'
import   jeditor from  "../../components/jeecg/JEditor"
export default {
    components:{jeditor},
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
        },
        {
          title: '状态',
          key: 'state',
          width: '200px',
          dataIndex: 'state',
          scopedSlots: { customRender: 'state' },
        },
      ],
      data: [],
      startTime: null,
      endTime: null,
      pageSize: 10,
      pageNo: 1,
      total:1,
      spinning :false,
      pageNo :1
    }
  },
  methods: {
     // 页码数量
    changePageSize(current, size) {
      this.pageSize = size
      this.reloadAll(current)
    },
    // 查询日期
    onChange(e, dateArr) {
       this.pageNo =1
       let data = {
        pageNo: this.page,
        pageSize: this.pageSize,
        status: 1,
        start:dateArr[0],
        end:dateArr[1]
      }
        this.spinning= true
      this.axios.post('pc/coursePlan/listByPage', data).then(res=>{
             if(res.success){
                   this.data = res.result.records ;
             }
          this.spinning =false
      })
    },
    reloadAll(page,pageSize) {
      let data = {
        pageNo:  this.pageNo,
        pageSize: this.pageSize,
        status: 1,
      }
        this.spinning= true
      this.axios.post('pc/coursePlan/listByPage', data).then((res) => {
         console.log(res);
        if (res.success) {
          for (let item of res.result.records) {
            item.createTime = moment(item.createTime).format('YYYY/MM/DD HH:mm')
          }
          this.data = res.result.records
          this.total = res.result.total
         }
           this.spinning= false
      })
    },
  },

  created() {
    this.reloadAll(1,10)
  },
}
</script>
<style scoped>
div {
}
</style>