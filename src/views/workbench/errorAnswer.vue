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
      <a-tab-pane key="1" tab="未解答列表">
        <a-table :columns="columns" :data-source="data">
          <a slot="file" slot-scope="img">
            <img :src="img" width="50" height="50" />
          </a>
          <span slot="action" slot-scope="text">
            <a-button type="primary" size="small" @click="answer(text)"> 解答</a-button>
          </span>
        </a-table>
      </a-tab-pane>
      <a-tab-pane key="2" tab="已解答列表" force-render>
        Content of Tab Pane 2
      </a-tab-pane>
    </a-tabs>
    <!-- 解答的模态框 -->
    <a-drawer
      placement="right"
      :closable="false"
      :visible="visible"
      @close="visible = false"
      width="500"
    >
      <div style=" ;">
        <a-card
          hoverable
          style="width: 400px; margin-right: 5px"
          v-for="(item, i) of answerList"
          :key="i"
        >
          <div style="width: 400px">
            <a-carousel autoplay>
              <div><h3>1</h3></div>
              <div><h3>1</h3></div>
            </a-carousel>
          </div>
          <div>
            <b style="font-size: 11px; padding: 0px 2px">{{ item.chapter }}</b>
          </div>
          <!-- <div>
            <sidebar>上传</sidebar>
          </div> -->
          <a-upload
            style="margin-left: "
            action="https://www.mocky.io/v2/5cc8019d300000980a055e76"
            list-type="picture-card"
            :file-list="item.fileList"
            @preview="handlePreview"
            @change="handleChange"
          >
            <div v-if="item.fileList.length < 8">
              <a-icon type="plus" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
          <a-modal
            :visible="previewVisible"
            :footer="null"
            @cancel="previewVisible = false"
          >
            <img alt="example" style="width: 100%" :src="previewImage" />
          </a-modal>
        </a-card>
      </div>
    </a-drawer>
  </div>
</template>
<script>
import testVue from "../finance/test.vue";
import sidebar from "../../components/sidebar";
function getBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result);
    reader.onerror = (error) => reject(error);
  });
}
const columns = [
  { title: "学生", dataIndex: "student", key: "student" },
  { title: "科目", dataIndex: "subject", key: "subject" },
  { title: "试卷", dataIndex: "textPaper", key: "textPaper" },
  { title: "提交时间", dataIndex: "saveTime", key: "saveTime" },
  {
    title: "文件",
    dataIndex: "file",
    key: "file",
    scopedSlots: { customRender: "file" },
  },
  { title: "操作", dataIndex: "", key: "x", scopedSlots: { customRender: "action" } },
];

const data = [
  {
    key: 1,
    student: "王虎",
    subject: "语文",
    textPaper: "方根与根式及根式的化简运算,三角函数,勾股定理,勾股定理",
    saveTime: "2020-12-30 15:30",
    file:
      "https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png,https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png",
  },
  {
    key: 2,
    student: "李四",
    subject: "数学",
    textPaper: "方根与根式及根式的化简运算",
    saveTime: "2020-12-30 15:30",
    file: require("../../assets/logo.png"),
  },
];

export default {
  components: { sidebar },
  data() {
    return {
      data,
      columns,
      subject: undefined,
      student: undefined,
      date: null,
      visible: false,
      confirmLoading: false,
      answerList: [],
      previewVisible: false,
      previewImage: "",
      fileList: [
        // {
        //   uid: "-1",
        //   name: "image.png",
        //   status: "done",
        //   url:
        //     "https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png",
        // },
      ],
    };
  },
  methods: {
    //   图片预览
    lookImg(imgUrl) {
      this.previewImage = imgUrl;
      this.previewVisible = true;
    },
    //   图片上传
    async handlePreview(file) {
      if (!file.url && !file.preview) {
        file.preview = await getBase64(file.originFileObj);
      }
      this.previewImage = file.url || file.preview;
      this.previewVisible = true;
    },
    handleChange({ fileList }) {
      console.log(fileList);
    },
    //解答按钮
    answer(obj) {
      this.answerList = [];
      this.visible = true;
      let chapterArr = obj.textPaper.split(",");
      let imgArr = obj.file.split(",");
      chapterArr.forEach((item, i) => {
        this.answerList.push({ chapter: item, img: imgArr, fileList: [] });
      });
      console.log(this.answerList);
    },
    //科目
    changeSubject(value) {},
    //学生
    changeStudent() {},
    // 日期范围
    onChangeDate(date, dateStr) {
      console.log(date, dateStr);
    },
    // 重置按钮
    reset() {
      this.date = null;
      this.subject = undefined;
      this.student = undefined;
    },
    //对话框的确认按钮
    handleOk(e) {
      this.confirmLoading = true;
      setTimeout(() => {
        this.visible = false;
        this.confirmLoading = false;
      }, 2000);
    },
  },
};
</script>
<style scoped>
.seach1 {
  display: flex;
  margin: 15px 5px;
}
.seach1 > div {
  margin-right: 50px;
}
.ant-carousel >>> .slick-slide {
  text-align: center;
  height: 160px;
  line-height: 160px;
  background: #364d79;
  overflow: hidden;
}

.ant-carousel >>> .slick-slide h3 {
  width: 50px;
  color: #fff;
}
::v-deep .ant-card-body {
  padding: 0;
}
.ant-upload-select-picture-card i {
  font-size: 32px;
  color: #999;
}

.ant-upload-select-picture-card .ant-upload-text {
  margin-top: 8px;
  color: #666;
}
::v-deep .ant-upload.ant-upload-select-picture-card {
  margin-top: 10px;
  margin-left: 5px;
  width: 80px;
  height: 80px;
}
</style>
