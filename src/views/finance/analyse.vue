<template>
  <div class="cark1">
    <!-- 查询的控件 -->
    <a-space :size="50">
        <!-- 老师 -->
      <div>
        <strong>老师：</strong>
        <a-select default-value="lucy" style="width: 120px" v-model="teacher" placeholder="请选择老师">
          <a-select-option value="jack"> Jack </a-select-option>
        </a-select>
      </div>
      <!-- 产品 -->
      <div>
        <strong>产品：</strong>
        <a-select
          default-value="lucy"
          style="width: 120px"
          v-model="prodoct"
          placeholder="请选择产品
        "
        >
          <a-select-option value="jack"> Jack </a-select-option>
        </a-select>
      </div>
      <!-- 日期范围： -->
      <div>
        <a-space>
          <strong>日期范围：</strong>
          <a-range-picker @change="onChangeDate" v-model="rangeDate" />
          <a-button type="primary"> 查询</a-button>
          <a-button @click="reset">重置</a-button>
        </a-space>
      </div>
    </a-space>
    <a-tabs default-active-key="1">
      <a-tab-pane key="1" tab="老师"> 
           <a-table :columns="columns" :data-source="data" >
               </a-table>
      </a-tab-pane>
      <a-tab-pane key="2" tab="学生" force-render> asdsad222 </a-tab-pane>
    </a-tabs>
  </div>
</template>
<script>
const columns = [
  {
    title: '老师',
    dataIndex: 'teacher',
  },
  {
    title: '学生',
    dataIndex: 'student',
  },
  {
    title: '规划总数',
    dataIndex: 'projectSum',
     defaultSortOrder: 'descend',
     sorter: (a, b) => a.projectSum - b.projectSum,
  },
  {
    title: '完成总数',
    dataIndex: 'finishSum',
    defaultSortOrder: 'descend',
     sorter: (a, b) => a.finishSum - b.finishSum,
  },
 
  {
    title: '差额',
    dataIndex: 'priceSpread',
    defaultSortOrder: 'descend',
    sorter: (a, b) => a.priceSpread - b.priceSpread,
  },
   {
    title: '完成率',
    dataIndex: 'finishRate',
    defaultSortOrder: 'descend',
    sorter: (a, b) => a.finishRate - b.finishRate,
  },
  {
    title: 'Action',
    key: 'action',
    scopedSlots: { customRender: 'action' },
  },
];
export default {
  data() {
    return {
      columns,
      data:[
        {
             key:"1",
            teacher:"王虎",
            student:"李四",
            prodoct:"一对一",
            projectSum:100,
            finishSum:50,
            priceSpread:500,
            finishRate:100
        },
           {
            key:"2",
            teacher:"许芹",
            student:"张三",
            prodoct:"学习管理",
            projectSum:220,
            finishSum:100,
            priceSpread:300,
            finishRate:95
        },

      ] , 
      teacher: undefined,
      prodoct: undefined,
      rangeDate:null
    }
  },
  methods: {
    //   日期查询
    onChangeDate(date) {
      console.log(date);
    },
    // 重置按钮
    reset(){
         this.teacher= "";
         this.prodoct="";
         this.rangeDate=null;
    }
  },
}
</script>
<style scoped>
</style>

