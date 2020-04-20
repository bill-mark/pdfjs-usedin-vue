####开篇说明:
  pdfjs指的是:[http://mozilla.github.io/pdf.js/](http://mozilla.github.io/pdf.js/)
或者[https://github.com/mozilla/pdf.js](https://github.com/mozilla/pdf.js)


写这篇博客的原因
   PDF在vue中的使用已经有很多文章了,最后发现不能很好的满足项目需要
项目要求在线加载PDF,并且滚动显示,能放大缩小等.

####踩坑记录
第一个坑
 pdf-vue [https://github.com/FranckFreiburger/vue-pdf](https://github.com/FranckFreiburger/vue-pdf)
这个插件比较简单,npm install --save pdf-vue既可以使用
流程走到最后发现,这个插件不能滚动翻页,只能手动触发上页和下页...
还有该插件除了上下页只有一个旋转功能,太简单了.
万一项目复杂了,还得换

第二个坑
pdfjs有好多使用博客,比如这个18年的文章[https://www.jianshu.com/p/b48af7917656](https://www.jianshu.com/p/b48af7917656)

但是照着直接写,会报错.因为pdfjs的npm版本在18年后发生了很大变化,callback改成了promise,老代码直接不兼容.这算pdfjs更新的问题 不能向后兼容.

#正文
本教程使用的pdfjs版本是"pdfjs-dist": "^2.3.200",
如果读者安装教程走一遍发现报错,可以把版本改成和作者一致,再install

本文参考的是 pdfjs官网npm版本的使用示例,结合vue写的
官网npm版本地址[https://github.com/mozilla/pdf.js/blob/master/examples/node/getinfo.js](https://github.com/mozilla/pdf.js/blob/master/examples/node/getinfo.js)

第一步 安装
项目目录下
npm install pdfjs-dist --save

第二步  获得PDF地址
大概逻辑:先声明一些变量,然后获得PDF线上地址,最后调用下一步的初始化函数.
函数get_pdfurl()默认加载的是PDFjs给的在线PDF文档例子地址,外网比较慢,要等五分钟左右,可以注释掉,直接加载自己项目的pdf地址

```
<div class="pdf_down" >
           <div class="pdf_set_left"  @click="scaleD()">放大</div>
           <div class="pdf_set_middle" @click="scaleX()">缩小</div>
       </div>

       
       <div :style="{width:pdf_div_width,margin:'0 auto'}" >
           <canvas v-for="page in pdf_pages" :id="'the-canvas'+page" :key="page"></canvas>
       </div>
```

```
data(){
  	 return{
  	 	 pdf_scale:1.0,//pdf放大系数
  	 	 pdf_pages:[],
  	 	 pdf_div_width:'',
  	 	 pdf_src:null,
  	 }   
  },
```

```
get_pdfurl(){  //获得pdf
     	//例子:加载pdf线上示例
         this.pdf_src = 'https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf'
         this._loadFile(this.pdf_src)
     	   return

     	   //线上请求
         this.$axios.get('')
         .then((res)=>{
         	this.pdf_src = res.url
         	this._loadFile(this.pdf_src)
         })
     }
```


第三步:初始化
```
let PDFJS = require('pdfjs-dist');
```

```
_loadFile (url) {  //初始化pdf
        let loadingTask = PDFJS.getDocument(url)
        loadingTask.promise
        .then((pdf) => {
          this.pdfDoc = pdf
          this.pdf_pages = this.pdfDoc.numPages
          this.$nextTick(() => {
            this._renderPage(1)
          })
        })
     },
```
1.先引入pdfjs,作者和官网例子保持一致用的require没有用import.
2.引入之后 2.3版本返回的是个promise,不是以前的回调函数
3.该promise成功的话会返回一个pdfjs的实例, this.pdfDoc = pdf把该实例保存下,后面有用
4.滚动翻页也要知道总页数,所以 this.pdf_pages = this.pdfDoc.numPages 保持总页数
5.因为pdfjs其实是把PDF文件渲染成canvas,所以vue加载完dom后,要用nexttick调用渲染pdf函数

第四步,渲染
```
_renderPage (num) {  //渲染pdf页
          const that = this
          this.pdfDoc.getPage(num)
          .then((page) => {
            let canvas = document.getElementById('the-canvas' + num)
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
```
这段代码大概意思是:通过id锚点把pdf渲染成canvas,
1.let viewport = page.getViewport({ scale: this.pdf_scale })
  sacle后面跟的是pdf的缩放比例.默认1.0
 大于1.0表示放大,比如1.1
小于1.0表示缩小,比如0.9
所以pdf的放大缩小就是通过改变scale绑定的变量值的大小后再重新调用渲染pdf的函数来实现
2.that.pdf_div_width = viewport.width + 'px'
这句话是拿到pdf文件的宽度,不同的PDF文件在1.0的缩放下宽度是不确定的,如果想让渲染的PDF居中,需要canvas元素外面嵌套层div,该div的宽度就是pdf_div_width


#总结
读者直接把博客中的代码复制到自己的项目中,修改下获取PDF地址函数就可以直接用了,
完整的dome在这 [https://github.com/bill-mark/pdfjs-usedin-vue](https://github.com/bill-mark/pdfjs-usedin-vue)

先npm install 再npm run serve就能看的效果  因为用的PDF示例文档是pdfjs官网提供的,非常慢要等一会才能显示