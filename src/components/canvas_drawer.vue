<template>
  <canvas canvas-id="canvas_drawer" :style="{width: width + 'px', height: height + 'px'}" class="canvas_drawer"></canvas>
</template>

<script>
const CACHE_NAME = 'canvasdrawer_pic_cache'
export default {
  name: '',
  props: {
    // 绘制图片宽度，单位px
    width: {
      type: Number,
      default: 100
    },
    // 绘制图片高度，单位px
    height: {
      type: Number,
      default: 100
    }
  },
  data () {
    return {
      canvasId: 'canvas_drawer',
      tempFileList: [], // 临时图片列表
      views: [],
      ctx: null, // canvas上下文
      cache: {} // 图片缓存
    }
  },
  methods: {
    /***
     * 开始绘制方法，可由父组件直接调用
     * @param config 绘制配置项
     */
    async draw (views) {
      wx.removeStorageSync(CACHE_NAME)
      this.cache = wx.getStorageSync(CACHE_NAME) || {}
      this.ctx = wx.createCanvasContext(this.canvasId, this)
      this.views = views
      this.ctx.clearActions()
      this.ctx.save()
      await this._getImagesInfo(views)
      this._startPainting()
    },
    /**
     * 保存绘制好的canvas图片到本地临时文件，返回临时文件地址
     * */
    async saveToTemp () {
      let _this = this
      return new Promise(function (resolve) {
        _this.ctx.draw(false, async () => {
          wx.setStorageSync(CACHE_NAME, _this.cache)
          const system = wx.getSystemInfoSync().system
          if (/ios/i.test(system)) {
            let tempFilePath = await _this._saveImageToTemp()
            resolve(tempFilePath)
          } else {
            setTimeout(async function () {
              let tempFilePath = await _this._saveImageToTemp()
              resolve(tempFilePath)
            }, 800)
          }
        })
      })
    },
    // 以下下划线开始的为私有方法，请勿随意调用
    async _getImagesInfo (views) {
      const imageList = []
      for (let i = 0; i < views.length; i++) {
        if (views[i].type === 'image') {
          imageList.push(this._getImageInfo(views[i].url))
        }
      }

      const loadTask = []
      for (let i = 0; i < Math.ceil(imageList.length / 8); i++) {
        loadTask.push(new Promise((resolve, reject) => {
          Promise.all(imageList.splice(i * 8, 8)).then(res => {
            resolve(res)
          }).catch(res => {
            reject(res)
          })
        }))
      }
      let res = await Promise.all(loadTask)
      let tempFileList = []
      for (let i = 0; i < res.length; i++) {
        tempFileList = tempFileList.concat(res[i])
      }
      this.tempFileList = tempFileList
    },
    _startPainting () {
      const tempFileList = this.tempFileList
      const views = this.views
      for (let i = 0, imageIndex = 0; i < views.length; i++) {
        if (views[i].type === 'image') {
          this._drawImage({
            ...views[i],
            url: tempFileList[imageIndex]
          })
          imageIndex++
        } else if (views[i].type === 'text') {
          if (!this.ctx.measureText) {
            wx.showModal({
              title: '提示',
              content: '当前微信版本过低，无法使用 measureText 功能，请升级到最新微信版本后重试。'
            })
            return
          } else {
            this._drawText(views[i])
          }
        } else if (views[i].type === 'rect') {
          this._drawRect(views[i])
        }
      }
    },
    _drawImage (params) {
      this.ctx.save()
      const { url, top = 0, left = 0, width = 0, height = 0, deg = 0 } = params
      if (deg !== 0) {
        this.ctx.translate(left + width / 2, top + height / 2)
        this.ctx.rotate(deg * Math.PI / 180)
        this.ctx.drawImage(url, -width / 2, -height / 2, width, height)
      } else {
        this.ctx.drawImage(url, left, top, width, height)
      }
      this.ctx.restore()
    },
    _drawText (params) {
      this.ctx.save()
      const {
        MaxLineNumber = 2,
        breakWord = false,
        color = 'black',
        content = '',
        fontSize = 16,
        top = 0,
        left = 0,
        lineHeight = 20,
        textAlign = 'left',
        width,
        bolder = false,
        textDecoration = 'none'
      } = params

      this.ctx.beginPath()
      this.ctx.setTextBaseline('top')
      this.ctx.setTextAlign(textAlign)
      this.ctx.setFillStyle(color)
      this.ctx.setFontSize(fontSize)

      if (!breakWord) {
        this.ctx.fillText(content, left, top)
        this._drawTextLine(left, top, textDecoration, color, fontSize, content)
      } else {
        let fillText = ''
        let fillTop = top
        let lineNum = 1
        for (let i = 0; i < content.length; i++) {
          fillText += [content[i]]
          if (this.ctx.measureText(fillText).width > width) {
            if (lineNum === MaxLineNumber) {
              if (i !== content.length) {
                fillText = fillText.substring(0, fillText.length - 1) + '...'
                this.ctx.fillText(fillText, left, fillTop)
                this._drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText)
                fillText = ''
                break
              }
            }
            this.ctx.fillText(fillText, left, fillTop)
            this._drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText)
            fillText = ''
            fillTop += lineHeight
            lineNum++
          }
        }
        this.ctx.fillText(fillText, left, fillTop)
        this._drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText)
      }

      this.ctx.restore()

      if (bolder) {
        this._drawText({
          ...params,
          left: left + 0.3,
          top: top + 0.3,
          bolder: false,
          textDecoration: 'none'
        })
      }
    },
    _drawTextLine (left, top, textDecoration, color, fontSize, content) {
      if (textDecoration === 'underline') {
        this._drawRect({
          background: color,
          top: top + fontSize * 1.2,
          left: left - 1,
          width: this.ctx.measureText(content).width + 3,
          height: 1
        })
      } else if (textDecoration === 'line-through') {
        this._drawRect({
          background: color,
          top: top + fontSize * 0.6,
          left: left - 1,
          width: this.ctx.measureText(content).width + 3,
          height: 1
        })
      }
    },
    _drawRect (params) {
      this.ctx.save()
      const { background, top = 0, left = 0, width = 0, height = 0 } = params
      this.ctx.setFillStyle(background)
      this.ctx.fillRect(left, top, width, height)
      this.ctx.restore()
    },
    // 下载单个图片到本地
    _getImageInfo (url) {
      return new Promise((resolve, reject) => {
        if (this.cache[url]) {
          resolve(this.cache[url])
        } else {
          const objExp = new RegExp(/^http(s)?:\/\/([\w-]+\.)+[\w-]+(\/[\w- .?%&=]*)?/)
          if (objExp.test(url)) {
            wx.getImageInfo({
              src: url,
              complete: res => {
                if (res.errMsg === 'getImageInfo:ok') {
                  this.cache[url] = res.path
                  resolve(res.path)
                } else {
                  reject(new Error('getImageInfo fail'))
                }
              }
            })
          } else {
            this.cache[url] = url
            resolve(url)
          }
        }
      })
    },

    /**
     * 保存图片到临时文件夹
     */
    _saveImageToTemp () {
      let _this = this
      return new Promise(function (resolve, reject) {
        wx.canvasToTempFilePath({
          x: 0,
          y: 0,
          width: _this.width,
          height: _this.height,
          canvasId: _this.canvasId,
          complete: res => {
            if (res.errMsg === 'canvasToTempFilePath:ok') {
              _this.tempFileList = []
              resolve(res.tempFilePath)
            } else {
              reject(res.errMsg)
            }
          }
        }, _this)
      })
    }
  }
}
</script>

<style>
  .canvas_drawer {
    position: fixed;
    top: 2000rpx;
    z-index: -100;
  }
</style>
