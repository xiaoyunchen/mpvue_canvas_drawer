## mpvue-canvas-drawer
>【mpvue promise版】在微信小程序中使用canvas绘制分享海报

----

### demo
```vue
<template>
  <canvasdrawer :width="346" :height="284" ref="pic"/>
</template>

  <script>
    import canvasdrawer from 'mpvue-canvas-drawer'
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
          }
        }
      }
  </script>
  
```
