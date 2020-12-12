<template>
  <div class="cark1">
    <div class="seach1">
      <div>
        <strong>科目:</strong>
        <a-select
          style="width: 120px; margin-left: 10px"
          placeholder="请选择科目"
          v-model="subject"
          @change="changeSubject"
        >
          <a-select-option value="jack"> Jack </a-select-option>
        </a-select>
      </div>
      <div>
        <strong>学生:</strong>
        <a-select
          style="width: 120px; margin-left: 10px"
          v-model="student"
          placeholder="请选择学生"
          @change="changeStudent"
        >
          <a-select-option value="jack"> Jack </a-select-option>
        </a-select>
      </div>
      <div style="margin-right: 10px">
        <strong> 日期范围：</strong>
        <a-range-picker @change="onChangeDate" v-model="date" />
      </div>
      <div>
        <a-button type="primary" @click="reset"> 重置</a-button>
      </div>
    </div>
    <!-- 列表 -->
    <a-tabs default-active-key="1">
      <a-tab-pane key="1" tab="未评改的">
        <a-table :columns="columns" :data-source="data">
            <a slot="file" slot-scope="img">
                 <img :src="img"  width="50" height="50">
            </a>
            <span slot="action" >
                 <a-button type="primary" size="small" @click="$router.push('/workbench/regarding?id=1')"> 评改作业</a-button>   
            </span>
        </a-table>
      </a-tab-pane>
      <a-tab-pane key="2" tab="已评改的" force-render> Content of Tab Pane 2 </a-tab-pane>
    </a-tabs>
  </div>
</template>
<script>
import testVue from '../finance/test.vue'
const columns = [
  { title: '学生', dataIndex: 'student', key: 'student' },
  { title: '科目', dataIndex: 'subject', key: 'subject' },
  { title: '试卷', dataIndex: 'textPaper', key: 'textPaper' },
  { title: '提交时间', dataIndex: 'saveTime', key: 'saveTime' },
  { title: '文件', dataIndex: 'file', key: 'file', scopedSlots: { customRender: 'file' }},
  { title: '操作', dataIndex: '', key: 'x', scopedSlots: { customRender: 'action' } },
]

const data = [
  {
    key: 1,
    student: '王虎',
    subject: '语文',
    textPaper: '期中试卷',
    saveTime:"2020-12-30 15:30",
    file: require('../../assets/logo.png'),
  },
  {
    key: 2,
    student: '李四',
    subject: '数学',
    textPaper: '期末试卷',
    saveTime:"2020-12-30 15:30",
    file: require('../../assets/logo.png'),
  },
]


export default {
  data() {
    return {
      data,
      columns,
      subject: undefined,
      student: undefined,
      date: null,
    }
  },
  methods: {
    //   科目
    changeSubject(value) {},
    //学生
    changeStudent() {},
    // 日期范围
    onChangeDate(date, dateStr) {
      console.log(date, dateStr)
    },
    // 重置按钮
    reset() {
      this.date = null
      this.subject = undefined
      this.student = undefined
    },
  },
}
</script>
<style  scoped>
.seach1 {
  display: flex;
  margin: 15px 5px;
}
.seach1 > div {
  margin-right: 50px;
}
</style>
