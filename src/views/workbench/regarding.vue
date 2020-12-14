<template>
  <div class="cark1">
    <!--标题 -->
    <div class="title">
      <div>数学</div>
      <b>方根与根式及根式的化简运算</b>
    </div>

    <div>
      <a-card hoverable style="width: 200px">
        <img slot="cover" alt="example" :src="imgUrl" />
        <div style="margin: 5px 5px">
          <a-button size="small" @click="imgModel">
            <a-icon type="scissor" /> 裁剪图片</a-button
          >
        </div>
      </a-card>
    </div>

    <!-- 富文本框 -->
    <div style="margin: 15px 0px">
      <j-editor @change="mytext" triggerChange></j-editor>
    </div>

    <!-- 错题和难题 -->
    <div>
      <div>
        <sidebar> 疑难题</sidebar>
        <div class="topic">
          <img src="../../assets/logo.png" />
          <img src="../../assets/logo.png" />
        </div>
      </div>
      <div>
        <sidebar> 错题</sidebar>
        <div class="topic">
          <img src="../../assets/logo.png" />
        </div>
      </div>
    </div>
    <!-- 提交 -->
    <div style="text-align: center; margin-top: 20px">
      <a-button type="primary" style="width: 200px"> 确认提交</a-button>
    </div>
    <!--   -->
    <a-modal
      title="数学"
      :width="800"
      :visible="screenshot"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="screenshot = false"
    >
      <imag-cropper :imgs="imgUrl" @onImg="cropperValue"> </imag-cropper>
      <!-- 编辑图片区域 -->
      <span
        style="margin: 10px; position: relative; display: inline"
        v-for="(imge, i) of images"
        :key="i"
      >
        <img
          @click="lookImg('look', imge)"
          width="150"
          height="150"
          style="margin-left: 10px; margin-top: 10px"
          :src="imge"
        />
        <div class="removeImg" @click="lookImg('remove', i)">
          <a-icon type="close" :style="{ color: '#fff' }" />
        </div>
        <div class="image">
          <a-tag> ＋疑难库 </a-tag>
          <a-tag color="#2db7f5" style="margin-left: 13px"> ＋错题库 </a-tag>
        </div>
      </span>
      <div class="model" v-show="model">
        <div class="model-show" @click="model = false">
          <img :src="modelSrc" @click="model = false" />
        </div>
      </div>
    </a-modal>
  </div>
</template>
<script>
import imagCropper from "../jeecg/ImagCropper.vue";
import Editor from "@tinymce/tinymce-vue";
import JEditor from "@/components/jeecg/JEditor";

import sidebar from "../../components/sidebar.vue";

export default {
  components: { imagCropper, Editor, sidebar, JEditor },
  data() {
    return {
      //截图对话框
      screenshot: false,
      confirmLoading: false,
      imgUrl: "http://cdn.xyxiao.cn/Landscape_2.jpg",
      images: [],
      modelSrc: "",
      model: false,
      msgContent: "",
      disabled: false,
    };
  },
  methods: {
    mytext(value) {
      this.msgContent = value;
    },
    //图片的预览和删除
    lookImg(type, data) {
      console.log(this.msgContent);
      if (type == "look") {
        this.model = true;
        this.modelSrc = data;
      } else {
        this.images.splice(data, 1);
      }
    },

    //
    cropperValue(value) {
      this.images.push(value);
    },
    imgModel() {
      this.screenshot = true;
    },
    //
    save() {
      this.screenshot = true;
    },
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
<style lang="less" scoped>
.topic {
  margin: 10px 0px;
  display: flex;
  img {
    width: 100px;
    height: 100px;
    margin-left: 15px;
  }
}

.title {
  display: flex;
  align-items: center;
  margin: 15px 0;
}
.title > b {
  margin-left: 15px;
}
.title > div {
  width: 120px;
  text-align: center;
  height: 28px;
  border-radius: 50px;
  line-height: 28px;
  background-color: #c8e4ff;
  color: #1b8fff;
  font-weight: bold;
}
.image {
  position: absolute;
  display: flex;
  justify-content: space-around;
  bottom: -72px;
  left: 10px;
}
.removeImg {
  position: absolute;
  top: -58px;
  right: 3px;
}
.model {
  position: fixed;
  z-index: 10;
  width: 100vw;
  height: 100vh;
  overflow: auto;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.8);
}

.model-show {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  text-align: center;
}

.model img {
  display: block;
  margin: auto;
  width: auto;
  user-select: none;
  background-position: 0px 0px, 10px 10px;
  background-size: 20px 20px;
  background-image: linear-gradient(
      45deg,
      #eee 25%,
      transparent 25%,
      transparent 75%,
      #eee 75%,
      #eee 100%
    ),
    linear-gradient(45deg, #eee 25%, white 25%, white 75%, #eee 75%, #eee 100%);
}
::v-deep .ant-card-body {
  padding: 0px;
}
</style>
