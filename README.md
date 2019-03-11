# mpvue_canvas_drawer
> Fork自[mpvue_canvas_drawer](https://github.com/kuckboy1994/mpvue_canvas_drawer)

## how to use

``` vue
# install dependencies
npm install mpvue-canvas-drawer

# 引用注册组件
import canvasdrawer from 'mpvue-canvas-drawer'
<canvasdrawer :width="346" :height="284" ref="pic"/>

# 准备配置数据，绘制canvas
this.$refs.pic.draw(CANVAS_VIEWS)

# 将canvas导出图片到临时文件
let tempPath = await this.$refs.pic.saveToTemp()
console.log('tempPath:', tempPath)
```
完整示例代码请参考 [demo](https://github.com/xiaoyunchen/mpvue_canvas_drawer/blob/master/src/pages/index/index.vue) 

### options
#### width
> canvas宽度，单位px，默认值100

#### height 
> canvas高度，单位px，默认值100

### 方法
#### draw
> 开始绘制方法，可由父组件直接调用

> 参数：需要绘制的配置数据，配置文件参考demo



#### saveToTemp
> 保存绘制好的canvas图片到本地临时文件，返回临时文件地址
