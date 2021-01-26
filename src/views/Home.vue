<template>
  <div class="home_wrap">
       <div class="pdf_down" >
           <div class="pdf_set_left"  @click="scaleD()">放大</div>
           <div class="pdf_set_middle" @click="scaleX()">缩小</div>
       </div>

       
       <div :style="{width:pdf_div_width,margin:'0 auto'}" >
           <canvas v-for="page in pdf_pages" :id="'the_canvas'+page" :key="page"></canvas>
       </div>
  </div>
</template>

<script>
let PDFJS = require('pdfjs-dist');
PDFJS.GlobalWorkerOptions.workerSrc = require("pdfjs-dist/build/pdf.worker.entry.js");
export default {
  data(){
  	 return{
  	 	 pdf_scale:1.0,//pdf放大系数
  	 	 pdf_pages:[],
  	 	 pdf_div_width:'',
  	 	 pdf_src:null,
  	 }   
  },
  mounted(){
     this.get_pdfurl()
  },
  methods:{
     scaleD() {  //放大
         let max = 0
         if (window.screen.width > 1440) {
             max = 1.4
         }else{
           max = 1.2
         }
         if(this.pdf_scale >= max){
           return
         }
         this.pdf_scale = this.pdf_scale+0.1
         this._loadFile(this.pdf_src)
     },   
     scaleX() {  //缩小
         let min = 1.0
         if(this.pdf_scale <= min){
           return
         }
         this.pdf_scale = this.pdf_scale - 0.1
         this._loadFile(this.pdf_src)
     },
     get_pdfurl(){  //获得pdf教案
     	   //加载本地
         this.pdf_src = 'https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf'
        // this.pdf_src = 'http://kaoshiftp.book118.com//%E5%B0%8F%E5%AD%A6%E5%88%9D%E4%B8%AD/%E5%88%9D%E4%B8%AD%E6%95%B0%E5%AD%A6/%E6%B5%B7%E6%B7%80%E8%A7%A3%E9%A2%98%E9%A2%98%E5%85%B8-%E5%88%9D%E4%B8%AD%E6%95%B0%E5%AD%A6.pdf'
         this._loadFile(this.pdf_src)
     	   return


     	   //线上请求
        //  this.$axios.get('')
        //  .then((res)=>{
        //  	this.pdf_src = res.url
        //  	this._loadFile(this.pdf_src)
        //  })
     },
     _loadFile (url) {  //初始化pdf
        let loadingTask = PDFJS.getDocument(url)
        loadingTask.promise
        .then((pdf) => {
          this.pdfDoc = pdf
          this.pdf_pages = this.pdfDoc.numPages
          //debugger
          this.$nextTick(() => {
            this._renderPage(1)
          })
        })
     },
     _renderPage (num) {  //渲染pdf页
          const that = this
          this.pdfDoc.getPage(num)
          .then((page) => {
            let canvas = document.getElementById('the_canvas' + num)
            let ctx = canvas.getContext('2d')
            let dpr = window.devicePixelRatio || 1
            let bsr = ctx.webkitBackingStorePixelRatio ||
              ctx.mozBackingStorePixelRatio ||
              ctx.msBackingStorePixelRatio ||
              ctx.oBackingStorePixelRatio ||
              ctx.backingStorePixelRatio || 1
            let ratio = dpr / bsr
            let viewport = page.getViewport({ scale: this.pdf_scale })

            canvas.width = viewport.width * ratio
            canvas.height = viewport.height * ratio

            canvas.style.width = viewport.width + 'px'

            that.pdf_div_width = viewport.width + 'px'

            canvas.style.height = viewport.height + 'px'

            ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
            let renderContext = {
              canvasContext: ctx,
              viewport: viewport
            }
            page.render(renderContext)
            if (this.pdf_pages > num) {
              this._renderPage(num + 1)
            }
          })
     },  
  }
}
</script>
<style lang="scss" scoped>
.home_wrap{
   width: 100%;
   height: 100%;
   .pdf_down{
    position:fixed;
    display: flex;
    z-index: 20;
    right:26px;
    bottom:7%;
    .pdf_set_left{
         width: 30px;
         height: 40px;
         color: #408FFF;
         font-size: 11px;
         padding-top:25px;
         text-align: center;
         margin-right: 5px;
         cursor: pointer;
    }
    .pdf_set_middle{
         width: 30px;
         height: 40px;
         color: #408FFF;
         font-size: 11px;
         padding-top:25px;
         text-align: center;
         margin-right: 5px;
         cursor: pointer;
    }
  }
}	
</style>













