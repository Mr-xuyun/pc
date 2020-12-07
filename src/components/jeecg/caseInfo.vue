<template>
  <keep-alive>
    <div style="position:relative;">
      <div class="button-box" v-drag draggable="false">
        <div class="btn-bg-img" @dblclick="openBox"></div>
        <div class="font-box">
          <a-upload
            list-type="picture-card"
          >
            <div>
              <a-icon type="plus" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
          <a-progress :strokeWidth="20" :percent="percent"></a-progress>
        </div>
      </div>
    </div>
  </keep-alive>
</template>

<script>
export default {
  props: {
    size: {
      type: Number,
      default:1024*1024*1024,
      required:false,
    }
},
  data () {
    return {
      isOpen: false,
      isMove: false,
      sum:this.size,
      fileSize: this.size,
      percent:50
    }
  },
  computed:{

  },
  created() {

  },
  methods: {
    openBox () {
      console.log('双击')
    },
    mousedowm (e) { // 鼠标按下时的鼠标所在的X，Y坐标
      this.mouseDownX = e.pageX
      this.mouseDownY = e.pageY
      // 初始位置的X，Y 坐标
      // this.initX = obj.offsetLeft;
      // this.initY = obj.offsetTop;
      console.log('e', e)
      // 表示鼠标已按下
      this.flag = true
    },
    mousemove (e) {
      if (this.flag) {
        console.log('e :', e)
      }
    }
  },
  directives: {
    drag (el) {
      let oDiv = el // 当前元素
      // let self = this // 上下文
      // 禁止选择网页上的文字
      document.onselectstart = function () {
        return false
      }
      oDiv.onmousedown = function (e) {
        // 鼠标按下，计算当前元素距离可视区的距离
        let disX = e.clientX - oDiv.offsetLeft
        let disY = e.clientY - oDiv.offsetTop
        document.onmousemove = function (e) {
          // 通过事件委托，计算移动的距离
          let l = e.clientX - disX
          let t = e.clientY - disY
          // 移动当前元素
          oDiv.style.left = l + 'px'
          oDiv.style.top = t + 'px'
        }
        document.onmouseup = function (e) {
          document.onmousemove = null
          document.onmouseup = null
        }
        // return false不加的话可能导致黏连，就是拖到一个地方时div粘在鼠标上不下来，相当于onmouseup失效
        return false
      }
    }
  }
}
</script>

<style scoped>
.button-box {
/*  width: 80px;*/
  border-radius: 50%;
  position: fixed;
/*  bottom: 80px;
  right: 50px;
  padding-left: 15px;
  padding-top: 8px;*/
  cursor: pointer;
  opacity: 0.7;
  z-index: 888;
}
.btn-bg-img {
  width: 80px;
  height: 80px;
  background-size: cover;
}
.button-box:hover {
  color: white;
  opacity: 1;
}

.font-box {
  width: 120px;
  height: 40px;
  display: flex;
}
</style>
