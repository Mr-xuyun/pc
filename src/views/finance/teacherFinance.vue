<template>
  <div class="cark1">
    <!-- 顶部菜单 -->
    <div style="margin-bottom: 16px">
      <label>老师:</label>
      <a-select style="width: 200px" @select="reloadAll(1)" v-model="teacherId" placeholder="请选择老师">
        <a-select-option v-for="item in teacherList" :key="item.id" :value="item.sysUserId"> {{ item.name }}</a-select-option>
      </a-select>
      <label >产品:</label>
      <a-select style="width: 200px" @select="reloadAll(1)" v-model="productId" placeholder="请选择产品">
        <a-select-option v-for="item in productList" :key="item.id" :value="item.id"> {{ item.productName }}</a-select-option>
      </a-select>
      <label >日期:</label>
      <a-range-picker v-model="dateRange" @change="changeDate" />
    </div>
    <!--  表格的内容 -->
    <a-spin :spinning="spinning">
    <a-table
      :pagination="false"
      :columns="columns"
      :data-source="data"
      bordered
      rowKey="id"
    >
    </a-table>
    </a-spin >
    <!-- 分页 -->
    <a-pagination
      show-size-changer
      :default-current="1"
      v-model="pageNo"
      :total="total"
      show-quick-jumper
      @showSizeChange="changePageSize"
      @change="reloadAll(pageNo)"
    />
  </div>
</template>

<script>
const columns = [
  {
    title: '老师',
    dataIndex: 'teacher'
  },
  {
    title: '产品名称',
    dataIndex: 'productName'
  },
  {
    title: '产品总次数',
    dataIndex: 'teachCount',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.teachCount+"次";
    }
  },
  {
    title: '产品总时长',
    dataIndex: 'teachTime',
    //处理复杂数据结果
    customRender: (text, record) => {
      return record.teachTime+"小时";
    }
  }
]
export default {
  data() {
    return {
      columns,//表头
      data: [],//表格数据
      hasSelected:true,//是否禁用全选按钮
      loading:false,//批量删除是否显示加载状态
      dateRange:[],//日期段
      start:"",//开始日期
      end:"",//结束日期
      teacherList:[],//老师列表
      teacherId:"",//老师ID
      productList:[],//产品列表
      productId:"",//产品ID
      pageNo:1,//当前页
      total:100,//总页数
      spinning:false,
      pageSize : 10 // 页码的数量
    }
  },
  methods:{
    // 页码
    changePageSize(current, size){
          this.pageSize = size
          this.reloadAll(current)
    },
    changeDate(date,dateStr){
      this.start = dateStr[0];
      this.end = dateStr[1];
      this.reloadAll(1);
    },

    reloadAll(current ,pageSize){
      let data = {
        pageNo: current,
        pageSize: this.pageSize,
        teacherId:this.teacherId,
        productId:this.productId,
        start:this.start,
        end:this.end,
      }
        this.spinning=true
      this.axios.post('pc/coursePlan/getTeacherTeachTotalTime', data).then((res) => {
           if(res.success){
           this.data = [];
            let result = res.result.records;
            for (let i = 0; i < result.length; i++) {
              result[i].id = i;
              this.data.push(result[i]);
            }
            this.pageNo = res.result.current;
            this.total = res.result.total; 
            this.spinning =false;
           }
      })
    },
    loadInfo(){
      this.axios.post('pc/teacherInfo/listAll').then((res) => {
        this.teacherList = res.result
      })
      this.axios.post('pc/product/listAll',{delFlag:0}).then((res) => {
        this.productList = res.result
      })
    },
  },
  mounted() {
    this.reloadAll(1);
    this.loadInfo();
  }
}
</script>

<style scoped>
::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
label{
  margin-left: 15px;
  margin-right: 10px;
  font-weight: bold;
}
</style>