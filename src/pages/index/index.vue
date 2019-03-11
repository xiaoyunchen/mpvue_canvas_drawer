<template>
  <div class="container">
    <img :src="shareImage" class="share-image" />
    <canvasdrawer :width="346" :height="284" ref="pic"/>

    <div class="btns">
      <button @click="draw">绘制</button>
      <button @click="save" class="save">保存到本地</button>
      <button @click="refresh">再试一次</button>
    </div>
  </div>
</template>

<script>
  import canvasdrawer from '@/components/canvas_drawer'
  const WIDTH = 346
  const HEIGHT = 284
  const CANVAS_VIEWS = [
    {
      type: 'rect',
      background: '#fff',
      top: 0,
      left: 0,
      width: WIDTH,
      height: HEIGHT
    },
    {
      type: 'image',
      url: 'https://yxs-web.oss-cn-beijing.aliyuncs.com/9253c0000bf93786cf9b949d6ef0e387.png',
      top: 0,
      left: 0,
      width: WIDTH,
      height: 164 - 46
    },
    {
      type: 'image',
      url: '/static/1x1_bg.png',
      top: 0,
      left: 0,
      width: WIDTH,
      height: 164 - 46
    },
    {
      type: 'text',
      content: '【第二曲线模块】第一节：创新',
      fontSize: 15,
      color: '#fff',
      textAlign: 'left',
      top: 117 - 46,
      left: 10,
      width: 270,
      bolder: true,
      breakWord: true,
      MaxLineNumber: 1
    },
    {
      type: 'text',
      content: '李善友 混沌大学创办人',
      fontSize: 12,
      color: '#fff',
      textAlign: 'left',
      top: 140 - 46,
      left: 10,
      width: 270,
      breakWord: true,
      MaxLineNumber: 1
    },
    {
      type: 'image',
      url: 'http://wx.qlogo.cn/mmhead/QnM5bMcic4Z3AB9xdZMyKhIn2EqgrlWSmQX7S2ppjkzUyfddPVZoMyg/64/0',
      top: 182 - 46,
      left: 13,
      width: 38,
      height: 38
    },
    {
      type: 'image',
      url: '/static/head_img_bg.png',
      top: 181 - 46,
      left: 12,
      width: 40,
      height: 40
    },
    {
      type: 'text',
      content: 'souvenir',
      fontSize: 14,
      lineHeight: 16,
      color: '#333333',
      textAlign: 'left',
      top: 182 - 46,
      left: 58,
      width: 200
    },
    {
      type: 'text',
      content: '正在',
      fontSize: 13,
      lineHeight: 16,
      color: '#848484',
      textAlign: 'left',
      top: 204 - 46,
      left: 58,
      width: 25
    },
    {
      type: 'text',
      content: ' 混沌大学 ',
      fontSize: 13,
      lineHeight: 16,
      color: '#333333',
      textAlign: 'left',
      top: 204 - 46,
      left: 85,
      width: 38
    },
    {
      type: 'text',
      content: '学习这堂课',
      fontSize: 13,
      lineHeight: 16,
      color: '#848484',
      textAlign: 'left',
      top: 204 - 46,
      left: 145,
      width: 80
    },
    {
      type: 'rect',
      background: '#ddd',
      top: 236 - 46,
      left: 18,
      width: 254,
      height: 1
    },
    {
      type: 'image',
      url: 'http://wx.qlogo.cn/mmhead/QnM5bMcic4Z3AB9xdZMyKhIn2EqgrlWSmQX7S2ppjkzUyfddPVZoMyg/64/0',
      top: 252 - 46,
      left: 20,
      width: 70,
      height: 70
    },
    {
      type: 'text',
      content: '下载【混沌大学】App',
      fontSize: 12,
      lineHeight: 16,
      color: '#999999',
      textAlign: 'left',
      top: 267 - 46,
      left: 98,
      width: 150,
      MaxLineNumber: 1
    },
    {
      type: 'text',
      content: '和我一起学习此课程',
      fontSize: 12,
      lineHeight: 16,
      color: '#999999',
      textAlign: 'left',
      top: 289 - 46,
      left: 98,
      width: 150,
      MaxLineNumber: 1
    }]
  export default {
    components: {
      canvasdrawer
    },
    data () {
      return {
        shareImage: ''
      }
    },
    methods: {
      async draw () {
        wx.showLoading({
          title: '绘制分享图片中',
          mask: true
        })
        await this.$refs.pic.draw(CANVAS_VIEWS)
        let tempPath = await this.$refs.pic.saveToTemp()
        console.log('tempPath:', tempPath)
        this.shareImage = tempPath
        wx.hideLoading()
      },
      async save () {
        if (!this.shareImage) {
          return wx.showToast({
            title: '请先绘制图片',
            icon: 'none',
            duration: 2000
          })
        }
        wx.saveImageToPhotosAlbum({
          filePath: this.shareImage,
          success () {
            wx.showToast({
              title: '保存图片成功',
              icon: 'success',
              duration: 2000
            })
          }
        })
      },
      refresh () {
        wx.reLaunch({url: '/pages/index/main'})
      }
    }
  }
</script>

<style scoped>
  .share-image {
    width: 290px;
    height: 284px;
    margin: 0 auto;
  }
  .share-image[src=''] {
    border: 1px dashed #d8d8d8;
  }
  .btns {
    margin-top: 40rpx;
    display: flex;
    justify-content: space-between;
    width: 100%;
  }
</style>
