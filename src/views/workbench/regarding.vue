<template>
  <div>
    <!--截图的 对话框 -->
      <img :src="imgUrl" width="150" height="150" @click="imgModel">
    <a-modal
      title="数学"
      :width="800"
      :visible="screenshot"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="screenshot = false"
    >
       
         <imag-cropper :imgs="imgUrl" @onImg="cropperValue"> </imag-cropper>

          <!-- <div style="display:flex;"    >
               <div v-for="(item,i) of images" :key="i"  >
                  <a-card hoverable style="width: 150px; margin-left:10px; margin-top:15px; "   >
                  <img
                    slot="cover"
                    alt="example"
                    width="150" height="150"
                    src="http://cdn.xyxiao.cn/Landscape_2.jpg"  
                  />
                       <div style="display: flex;  justify-content:space-evenly; ">
                         	<a-button  size="small" >＋疑难库</a-button>
                       <a-button  type="primary" size="small" style="margin-left:4px;"  >＋错题库</a-button>
                       </div>
                </a-card>
               </div>
            </div> -->
                   <span  style="margin:10px; position: relative;  display: inline; "  v-for="(imge,i) of images" :key="i">
                       <img 
                                width="150" height="150"
                                style="margin-left:10px;margin-top:10px;"
                                :src="imge"  
                      />  
                     <span  class="image">
                          	<a-button  size="small" >＋疑难库</a-button>
                           <a-button  type="primary" size="small" style="margin-left:4px;"  >＋错题库</a-button>
                     </span>
                   </span>
          
        
    </a-modal>
      <a-button @click="screenshot=true"> wedasd</a-button>
  </div>
</template>
<script>
 import imagCropper from "../jeecg/ImagCropper.vue";
export default {
    components:{imagCropper},
  data() {
    return {
         //截图对话框
      screenshot: false,
      confirmLoading: false,
      imgUrl:"http://cdn.xyxiao.cn/Landscape_2.jpg",
      images:[]
       
    }
  },
  methods:{
    cropperValue(value){
        this.images.push(value)
    },
    imgModel(){
         this.screenshot =true 
    },
    //   
   save(){
        this.screenshot =true
    },
     handleOk(e) {
      this.confirmLoading = true
      setTimeout(() => {
        this.visible = false
        this.confirmLoading = false
      }, 2000)
    },
  }
}
</script>
<style scoped>
.image{
   bottom: 0px;
   left: 50px;
   display: inline;

}
</style>

