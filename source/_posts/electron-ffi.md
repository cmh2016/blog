---
title: electron && ffi
date: 2019-03-14 18:18:10
tags: [vue,electron,node,ffi]
---

>技术栈
```js
electron // 使用 JavaScript, HTML 和 CSS 构建跨平台的桌面应用
electron-vue // 基于 vue (基本上是它听起来的样子) 来构造 electron 应用程序的样板代码。
electron-store // 本地简单数据存储
electron-log // 日志模块
electron-updater // 更新模块
electron-builder // 打包模块
electron-rebuild //下载 headers、编译原生模来重建 Electron 模块 
ffi // 使用node加载和调用动态库
iconv // 转码
node-machine-id // 获取计算机唯一标识
vue-router // 路由
axios // http
element-ui //UI库
```


#### 1.fatal error LNK1127

删除用户目录下.node-gyp  重新安装 npm install node-gyp -g

#### 2.项目开发环境？

    node32位最新版 electron32位（npm i会自动下载32位的） dll文件32位 win10操作系统 .NET Framework 
    安装node-gyp 的前提条件如下：
    A.
    1.python(v2.7 ，3.x不支持);
    2.visual C++ Build Tools,或者 （vs2015以上（包含15))
    3..net framework 4.5.1
    B.如果是干净的环境可以用下面命令一键安装
    npm install --global --production windows-build-tools
    报Python错误的需要 npm iconfig set python python.exe安装位置

#### 3.窗口最大化最小化 

isMaximized()有问题 需要自己声明flag
```js
let isMaxWin = false
ipcMain.on('min', e => mainWindow.minimize())
ipcMain.on('max', e => {
  console.log('isMaxWin:' + isMaxWin)
  if (isMaxWin) {
    isMaxWin = false
    mainWindow.unmaximize()
  } else {
    mainWindow.maximize()
    isMaxWin = true
  }
})
```

#### 4.node-ffi调用dll常见序错误

    1.Win32 error 126错误就是：
        通常是传入的DLL路径错误，找不到Dll文件，推荐使用绝对路径。
        如果是在x64的node/electron下引用32位的DLL，也会报这个错，反之亦然。要确保DLL要求的CPU架构和你的运行环境相同。
        DLL还有引用其他DLL文件，但是找不到引用的DLL文件，可能是VC依赖库或者多个DLL之间存在依赖关系。
        Dynamic Linking Error: Win32 error 127：DLL中没有找到对应名称的函数，需要检查头文件定义的函数名是否与DLL调用时写的函数名是否相同。

    2.详细错误码 https://www.jianshu.com/p/79bbb96e0065
#### 5.宽高分辨率适应

``` js
let devInnerHeight = 1080.0 // 开发时的InnerHeight
let devDevicePixelRatio = 1.0// 开发时的devicepixelratio
let devScaleFactor = 1.3 // 开发时的ScaleFactor
let scaleFactor = require('electron').screen.getPrimaryDisplay().scaleFactor
let zoomFactor = (window.innerHeight / devInnerHeight) * (window.devicePixelRatio / devDevicePixelRatio) * (devScaleFactor / scaleFactor)
require('electron').webFrame.setZoomFactor(zoomFactor)
```
#### 6.c与javascript类型对应
>https://www.npmjs.com/package/node-ffi#types

#### 7.node-ffi 调用 dll，打包时asar为true时报错问题

因为asar模式下为只读文件，导致ffi报错，issue上也有提问，没有解答（https://github.com/node-ffi/node-ffi/issues?utf8=%E2%9C%93&q=asar）
各种文档查看后，发现"asarUnpack"，所以才有下曲线救国的方案
```json
    "asarUnpack":[
      "./dist/electron", // dll文件等
      "node_modules/ffi" // ffi不压缩
    ],
```
```js
if (process.env.NODE_ENV !== 'development') {
  __static = __static.replace('app.asar', 'app.asar.unpacked')
}
```

#### 8.应用自动下载更新和手动检查更新
>使用electron-updater结合electron-builder实现；细节注意渲染进程和主进程通信时手动检查时多次触发。可以根据下载进度判断。
```js
//main.js
import {autoUpdater} from 'electron-updater'

// 主进程监听渲染进程传来手动检查更新的信息
ipcMain.on('update', (e, arg) => {
  console.log('update')
  autoUpdater.checkForUpdates()
})

let checkForUpdates = () => {
  // 配置安装包远端服务器
  autoUpdater.setFeedURL('http://*********:8888/lastApp')

  // 下面是自动更新的整个生命周期所发生的事件
  autoUpdater.on('error', function (message) {
    sendUpdateMessage('error', message)
  })
  autoUpdater.on('checking-for-update', function (message) {
    sendUpdateMessage('checking-for-update', message)
  })
  autoUpdater.on('update-available', function (message) {
    sendUpdateMessage('update-available', message)
  })
  autoUpdater.on('update-not-available', function (message) {
    sendUpdateMessage('update-not-available', message)
  })

  // 更新下载进度事件
  autoUpdater.on('download-progress', function (progressObj) {
    mainWindow.webContents.send('downloadProgress', progressObj)
  })
  // 更新下载完成事件
  autoUpdater.on('update-downloaded', function (event, releaseNotes, releaseName, releaseDate, updateUrl, quitAndUpdate) {
    sendUpdateMessage('isUpdateNow')
    ipcMain.on('updateNow', (e, arg) => {
      autoUpdater.quitAndInstall()
    })
  })

  // 执行自动更新检查
  autoUpdater.checkForUpdates()
}

// 主进程主动发送消息给渲染进程函数
function sendUpdateMessage (message, data) {
  console.log({ message, data })
  mainWindow.webContents.send('message', { message, data })
}

//app启动时自动更新

mainWindow.on('ready-to-show', () => {
    console.log('mainWindow opened')
    mainWindow.show()
    mainWindow.focus()
    // 启动时自动更新
    checkForUpdates()
})
```
>渲染进程监听更新事件UI层做出反馈
```js


const { ipcRenderer } = require('electron')

mounted () {
    this.$router.push({name: 'main'})
    // 更新进度
    ipcRenderer.on('message', (event, {message, data}) => {
      console.log(message + ' <br>data:' + JSON.stringify(data) + '<hr>')
      switch (message) {
        case 'update-available':
          this.newVer = data.version
          this.$notify({
            title: '发现新版本',
            dangerouslyUseHTMLString: true,
            duration: 0,
            message: '当前软件版本号：' + this.info.version + '<br>新版本版本号：' + data.version + '<br>正在下载最新版本，请稍等！',
            type: 'success'
          })
          this.show = true
          break
        case 'update-not-available':
          this.show = false
          this.$notify({
            title: '暂无新版本',
            dangerouslyUseHTMLString: true,
            duration: 0,
            message: '当前软件版本号：' + this.info.version + '<br>当前版本为最新版本，暂无新版本可更新！',
            type: 'success'
          })
          break
        case 'error':
          this.show = false
          this.$notify({
            title: '检查更新错误',
            dangerouslyUseHTMLString: true,
            duration: 0,
            message: '请联系xxx，或者手动到http://xxx.com下载' ,
            type: 'error'
          })
          break
        case 'isUpdateNow':
          this.show = false
          this.$confirm('新版本下载完成, 是否继续安装?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            closeOnClickModal: false,
            closeOnPressEscape: false,
            type: 'success'
          }).then(() => {
            this.$message({
              type: 'success',
              message: '正在安装，请稍后!'
            })
            ipcRenderer.send('updateNow')
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消安装,将在您下次使用时自动安装新版本！'
            })
          })
          break
      }
    })

    ipcRenderer.on('downloadProgress', (event, progressObj) => {
      console.log(progressObj)
      this.percent = progressObj.percent.toFixed(2) || 0
    })
  }


   methods: {
    //手动更新方法
    updater () {
      const loading = this.$loading({
        lock: true,
        text: '获取最新版本信息中...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })
      // 正在自动更新时，不发送更新指令，避免多次下载导致下载进度多条记录
      if (this.percent === 0) {
        ipcRenderer.send('update')
      } else {
        this.$notify({
          title: '发现新版本',
          dangerouslyUseHTMLString: true,
          duration: 0,
          message: '当前软件版本号：' + this.info.version + '<br>正在下载最新版本，请稍等！',
          type: 'success'
        })
        this.show = true
      }

      setTimeout(() => {
        loading.close()
      }, 2000)
    },
   ..................
  }
```

#### 9.程序秒开优化小技巧

因为electron是基于 Chromium 和 Node.js，所以页面加载的白屏问题特别是使用vue开发的单页应用。这里可以在实例化BrowserWindow时，把`show: false`然后在`ready-to-show`里面调用show方法和focus方法。

#### 10.axios封装
>主要实现以下功能

    .post 和 get 封装 

    .报错统一处理

    .post提交序列化json数据 后台支持json可忽略

    .可配置的loading 是否显示和显示的文字

    .多个请求时合并一个loading

    .统一headers提交

    .开发和产品环境下api地址的切换

>代码如下

```js
// request.js
import axios from 'axios'
import { Message } from 'element-ui'
import qs from 'qs'
import {getToken} from './cookie'
import {
  showFullScreenLoading,
  tryHideFullScreenLoading
} from './axiosLoading'
const log = require('electron-log')
const Store = require('electron-store')
const store = new Store()
log.transports.console.format = '{h}:{i}:{s} {text}'
log.transports.console.level = 'silly'
log.transports.file.maxSize = 5 * 1024 * 1024
// import store from '../store'
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8'
// 创建axios实例
const service = axios.create({
  // baseURL: process.env.BASE_API,  api的base_url
  timeout: 15000 // 请求超时时间
})

// request拦截器
service.interceptors.request.use(config => {
  if (store.get('userToken')) {
    config.headers['x-appMachineId'] = store.get('appMachineId')// 电脑的唯一识别码 除非重装系统 使用node-machine-id
    config.headers['x-userId'] = store.get('userToken').userId// 让每个请求携带用户id
    config.headers['x-appArea'] = store.get('appArea')// 软件所在区域
    config.headers['x-appDepartment'] = store.get('appDepartment')// 软件所在部门
  }
  // 是否显示loading动画
  if (config.showLoading) {
    // loading动画时下方的文字
    showFullScreenLoading(config.loadingText)
  }
  // POST 请求序列化json数据提交
  if (config.method === 'post') {
    config.data = qs.stringify(config.data)
  }
  // log文件记录
  log.info(getToken().nickName + ' 请求了 API request->' + JSON.stringify(config))
  return config
}, error => {
  // Do something with request error
  console.log(error) // for debug
  log.error(getToken().nickName + ' 请求了 API request error->' + JSON.stringify(error))
  Promise.reject(error)
})

// respone拦截器
service.interceptors.response.use(
  response => {
    log.info('API response ok->' + JSON.stringify(response))
    /**
  * code为非000是抛错 可结合自己业务进行修改
  */
    if (response.config.showLoading) {
      tryHideFullScreenLoading()
    }
    let ret = response.data
    if (ret.code !== '000') {
      // 非000错误下是否需要根据返回值做其他业务判断
      if (response.config.useError) {
        return response.data
      } else {
        Message({
          message: ret.msg,
          type: 'error',
          duration: 5 * 1000
        })
      }
    } else {
      return response.data
    }
  },
  error => {
    tryHideFullScreenLoading()
    console.log('err' + error)// for debug
    log.error('API response error->' + JSON.stringify(error))
    let err = error
    if (err && err.response) {
      switch (err.response.status) {
        case 400:
          err.message = '错误请求'
          break
        case 401:
          err.message = '未授权，请重新登录'
          break
        case 403:
          err.message = '拒绝访问'
          break
        case 404:
          err.message = '请求错误,未找到该资源'
          break
        case 405:
          err.message = '请求方法未允许'
          break
        case 408:
          err.message = '请求超时'
          break
        case 500:
          err.message = '服务器端出错'
          break
        case 501:
          err.message = '网络未实现'
          break
        case 502:
          err.message = '网络错误'
          break
        case 503:
          err.message = '服务不可用'
          break
        case 504:
          err.message = '网络超时'
          break
        case 505:
          err.message = 'http版本不支持该请求'
          break
        default:
          err.message = `连接错误${err.response.status}`
      }
    } else {
      err.message = '连接到服务器失败'
    }
    Message({
      message: error.message,
      type: 'error',
      duration: 5 * 1000
    })
    return Promise.reject(error)
  }
)

export default service

// axiosLoading.js
import { Loading } from 'element-ui'
import _ from 'lodash'

let needLoadingRequestCount = 0
let loading

function startLoading (text) {
  console.log('startLoading =============')
  loading = Loading.service({
    lock: true,
    text: text || 'loading...',
    background: 'rgba(0, 0, 0, 0.7)'
  })
}

function endLoading () {
  console.log('endLoading==========')
  loading.close()
}

const tryCloseLoading = () => {
  if (needLoadingRequestCount === 0) {
    endLoading()
  }
}

export function showFullScreenLoading (text) {
  if (needLoadingRequestCount === 0) {
    startLoading(text)
  }
  needLoadingRequestCount++
}

export function tryHideFullScreenLoading () {
  if (needLoadingRequestCount <= 0) return
  needLoadingRequestCount--
  if (needLoadingRequestCount === 0) {
    _.debounce(tryCloseLoading, 300)()
  }
}
//环境切换 
// webpack.renderer.config.js plugins添加
const config = require('../config/index.js')
new webpack.DefinePlugin({
      'process.env': process.env.NODE_ENV === 'production' ? config.build.env : config.dev.env
    })

//创建 config 文件下 和 dev.env.js index.js prod.env.js
//index.js
module.exports = {
  build: {
    env: require('./prod.env')
  },
  dev: {
    env: require('./dev.env')
  }
}

// dev.env.js
module.exports = {
  NODE_ENV: '"development"',
  BASE_API: '"https://www.easy-mock.com/xxxx/test"', // 系统接口
  BASE_API_JKPT: '"http://xxxx:8080/ims/API"' // 接口平台
}

//prod.env.js
module.exports = {
  NODE_ENV: '"production"',
  BASE_API: '"https://www.easy-mock.com/mock/xx/test"', // 系统接口
  BASE_API_JKPT: '"https://easy-mock.com/mock/xx/vue-admin"' // 接口平台
}

// 使用示例
import request from '@/tools/request'

export function getBaseData () {
  console.log(process.env)

  return request({
    url: process.env.BASE_API + '/baseData',// 根据环境变量自动切换
    method: 'get',
    showLoading: true,
    loadingText: '同步基础数据中',
    useError: true
  })
}
```

#### 11.调用dll中文乱码问题
```js
let Iconv = require('iconv').Iconv
// 读取卡信息
ds.iReadCardBas = function () {
  let info = Buffer.alloc(500)
  let ret = dsDDL.xxx(1, info) // dll xxx方法名
  let iconv = new Iconv('GBK', 'UTF-8') // 转码
  let retstring = iconv.convert(info).toString()
  if (ret === 0) {
    return retstring
  } else {
    dialog.showErrorBox('错误提示', retstring)
  }
}
```

#### 12.封装拍照模块
>使用mediaDevices 因为运行环境为谷歌浏览器所以不考虑兼容问题 [参考](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices)。已封装为cam组件，父组件调用this.$refs.camera.snapshot()方法拍照
```js
<template>
  <div class="cam"> 
     <el-alert
    title="拍摄要求：1.正脸拍照 2.白色背景 3.请勿佩戴帽子或眼镜 4.请参照绿色边框辅助线拍摄"
    type="info"
    show-icon>
  </el-alert>
      <img :src="imageUrl" alt="">
      <video ref="video" class="_vc-video vc-video" :style="videoStyle" :width="width" :height="height" webkit-playsinline></video>
      <canvas ref="canvas" :width="width" :height="height" v-show="false"></canvas>
  </div>
</template>

<script>
export default {
  name: 'Cam',
  data () {
    return {
      imageUrl: 'static/outline.png'
    }
  },
  props: {
    autoplay: {
      type: Boolean,
      default: true
    },
    width: {
      type: Number,
      default: 640
    },
    height: {
      type: Number,
      default: 480
    },
    fit: {
      type: String,
      default: 'cover',
      validator: function (value) {
        return ['cover', 'contain'].indexOf(value) !== -1
      }
    }
  },
  computed: {
    video () {
      return this.$refs.video
    },
    videoStyle () {
      return {
        'object-fit': this.fit
      }
    }
  },
  methods: {

    calculateVideoPreviewSize () {
      let video = this.$refs.video
      var videoPreviewSize = {
        width: video.videoWidth,
        height: video.videoHeight
      }

      let previewRatio = this.width / this.height
      let videoRatio = video.videoWidth / video.videoHeight

      // Set video preview size with width as a reference
      let widthFirst = () => {
        videoPreviewSize.width = this.width
        videoPreviewSize.height = video.videoHeight * this.width / video.videoWidth
      }

      // Set video preview size with height as a reference
      let heightFirst = () => {
        videoPreviewSize.height = this.height
        videoPreviewSize.width = video.videoWidth * this.height / video.videoHeight
      }

      if (this.fit === 'contain') {
        if ((previewRatio < 1 && videoRatio < 1) || (previewRatio <= 1 && videoRatio >= 1)) {
          widthFirst()
        } else {
          heightFirst()
        }
      } else if (this.fit === 'cover') {
        if ((previewRatio < 1 && videoRatio < 1) || (previewRatio <= 1 && videoRatio >= 1)) {
          heightFirst()
        } else {
          widthFirst()
        }
      }

      return videoPreviewSize
    },
    snapshot () {
      var video = this.$refs.video
      var canvas = this.$refs.canvas
      var context = canvas.getContext('2d')

      var videoPreviewSize = this.calculateVideoPreviewSize()

      // Draw the image
      let topOffset = (this.height - videoPreviewSize.height) / 2
      let leftOffset = (this.width - videoPreviewSize.width) / 2
      context.drawImage(video, leftOffset, topOffset, this.width - leftOffset * 2, this.height - topOffset * 2)

      // Return a base 64 snapshot
      return canvas.toDataURL('image/jpeg', 0.8)
    }
  },
  mounted () {
    var video = this.$refs.video
    let _this = this
    function updateDeviceList () {
      navigator.mediaDevices.enumerateDevices()
        .then(function (devices) {
          let haveok = false
          devices.forEach(function (device) {
            if (device.kind === 'videoinput') {
              haveok = true
              _this.$notify({
                title: '摄像头连接成功',
                message: device.label,
                type: 'success'
              })

              navigator.mediaDevices.getUserMedia({
                video: true
              })
                .then(stream => {
                  video.srcObject = stream

                  if (_this.autoplay) {
                    video.play()
                  }
                })
                .catch(error => {
                  _this.$notify.error({
                    title: '拍照模块错误',
                    message: error
                  })
                })
            }
          })
          if (haveok === false) {
            _this.$notify({
              title: '错误',
              message: '无连接摄像头，请查看摄像头连接是否正常可用',
              type: 'error'
            })
          }
        })
    }
    // Get permission to use the camera
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      // 插入或移除设备时监听做出提示
      navigator.mediaDevices.ondevicechange = function (event) {
        updateDeviceList()
      }
      navigator.mediaDevices.getUserMedia({
        video: true
      })
        .then(stream => {
          video.srcObject = stream

          if (_this.autoplay) {
            video.play()
          }
        })
        .catch(error => {
          _this.$notify.error({
            title: '拍照模块错误',
            message: error
          })
        })
    }
  }
}
</script>

<style lang="less" scoped>
.cam{
    position: relative;
    width: 100%;
    height: 500px;
    video,img{
        position: absolute;
        left: 100px;
        top: 43px;
    }
    img{
        z-index: 5;
    }
}
</style>

```

#### 13.封装图像实时处理
>封装的glfx.js，Caman JS也是个不错的选择。需要注意的是，项目如果开启eslint，js库会报错，开始在js文件头部添加/* eslint-disable */忽略库文件的校验。封装的常用方法，亮度、对比度、色相、饱和度、自然饱和度、降噪点、锐化、噪点等

```js
<template>
  <div class="front-process-container" v-loading="loading">
        <el-row :gutter="20">
            <el-col :span="12"><div class="grid-content">
                <p>拍摄照片</p>
                <img id="original" style="width:100%" crossorigin="anonymous" :src="imgUrl" ref="originalImg">
                </div></el-col>
            <el-col :span="12"><div class="grid-content">
                 <p>优化后照片</p>
                <img style="width:100%" crossorigin="anonymous" :src="newSrc">
                </div></el-col>
        </el-row>
        <el-row :gutter="20">
            <el-col :span="12">
            <div class="slider-wrap">
                <label class="slider-label">亮度</label>
                <el-slider v-model="valueOfBrightness" class="zj-slider" :max="100" :min="-100" @change="drawByParams"></el-slider>
            </div>
            </el-col>
            <el-col :span="12">
             <div class="slider-wrap">
                <label class="slider-label">对比度</label>
                <el-slider v-model="valueOfContrast" class="zj-slider" :max="100" :min="-100" @change="drawByParams"></el-slider>
            </div>
            </el-col>
        </el-row> 
        <el-row :gutter="20">
            <el-col :span="12">
             <div class="slider-wrap">
                <label class="slider-label">色调</label>
                <el-slider v-model="valueOfHue" class="zj-slider" :max="100" :min="-100" @change="drawByParams"></el-slider>
            </div>
            </el-col>
          <el-col :span="12">
            <div class="slider-wrap">
                <label class="slider-label">饱和度</label>
                <el-slider v-model="valueOfSaturation" class="zj-slider" :max="100" :min="-100" @change="drawByParams"></el-slider>
            </div>
            </el-col>
        </el-row>
       <el-row :gutter="20">
            <el-col :span="12">
             <div class="slider-wrap">
                <label class="slider-label">加噪</label>
                <el-slider v-model="valueOfNoise" class="zj-slider" :max="100" :min="0" @change="drawByParams"></el-slider>
            </div>
            </el-col>
          <el-col :span="12">
            <div class="slider-wrap">
                <label class="slider-label">深褐</label>
                <el-slider v-model="valueOfSepia" class="zj-slider" :max="100" :min="0" @change="drawByParams"></el-slider>
            </div>
            </el-col>
        </el-row>
        <el-row :gutter="20">
            <el-col :span="12">
              <div class="slider-wrap">
                <label class="slider-label">锐化半径</label>
                <el-slider v-model="usmRadius" class="zj-slider" :max="200" :min="0" @change="drawByParams"></el-slider>
            </div>
            </el-col>
          <el-col :span="12">
              <div class="slider-wrap">
                <label class="slider-label">锐化强度</label>
                <el-slider v-model="usmStrength" class="zj-slider" :max="5" :min="0" :step="0.01" @change="drawByParams"></el-slider>
            </div>
            </el-col>
        </el-row>
    
        <el-button class="op-btn" @click="drawByParams('denoise')" :class="{active: isDenoise}">降噪</el-button>
        <el-button class="op-btn" @click="resetImg">恢复原图</el-button>
  </div>
</template>

<script>
import * as _ from 'underscore'
import {fx} from '@/components/camera/glfx.js'
export default {
  data () {
    return {
      valueOfBrightness: 0,
      _canvas: null,
      _texture: null,
      imgElement: null,
      _draw: null,
      newSrc: '',
      OKIMG: '',
      originalSrc: '',
      valueOfContrast: 0,
      isCurves: false,
      isDenoise: false,
      valueOfHue: 0,
      valueOfSaturation: 0,
      valueOfNoise: 0,
      valueOfSepia: 0,
      usmRadius: 0,
      usmStrength: 0,
      loading: true,
      opBtn: []
    }
  },
  props: {
    imgUrl: {
      required: true
    }
  },
  methods: {
    drawByParams (operation) {
      this.loading = true
      this.resetProperty()
      let opIndex = _.indexOf(this.opBtn, operation)
      if (opIndex !== -1) {
        this.opBtn.splice(opIndex, 1)
      } else {
        this.opBtn.push(operation)
      }
      this._draw
        .brightnessContrast(this.valueOfBrightness / 100, this.valueOfContrast / 100)
        .hueSaturation(this.valueOfHue / 100, this.valueOfSaturation / 100)
        .noise(this.valueOfNoise / 100)
        .sepia(this.valueOfSepia / 100)
        .unsharpMask(this.usmRadius, this.usmStrength)
      if (_.indexOf(this.opBtn, 'curves') !== -1) {
        this.isCurves = true
        this._draw.curves([[0, 1], [1, 0]], [[0, 1], [1, 0]], [[0, 1], [1, 0]])
      } else {
        this.isCurves = false
      }
      if (_.indexOf(this.opBtn, 'denoise') !== -1) {
        this.isDenoise = true
        this._draw.denoise(20)
      } else {
        this.isDenoise = false
      }
      this._draw.update()

      console.log(this._draw)
      console.log(this._canvas)

      this.newSrc = this._canvas.toDataURL('image/jpeg', 0.8)
      this.setImg()
      this.loading = false
    },
    // 对照片进行358X441像素处理
    setImg () {
      let imageisok = new Image()
      imageisok.src = this.newSrc
      imageisok.onload = () => {
        let c = document.createElement('canvas')
        c.width = 358
        c.height = 441
        var context = c.getContext('2d')
        context.drawImage(imageisok, 0, 0, 358, 441)
        this.OKIMG = c.toDataURL('image/jpeg')
      }
    },
    returnPhoto () {
      return this.OKIMG
    },
    getWebGLElements () {
      console.log('getWebGLElements')
      if (!this._canvas) {
        this._canvas = fx.canvas()
      }
      console.log(this._canvas)
      if (!this.imgElement) {
        this.imgElement = this.$refs.originalImg
        this.originalSrc = this.imgElement.src
      }

      if (!this._texture) {
        this._texture = this._canvas.texture(this.imgElement)
      }

      if (!this._draw) {
        this._draw = this._canvas.draw(this._texture)
      }
      this.loading = false
    },
    resetProperty () {
      console.log('resetProperty:', this.imgElement)
      console.log(this._texture)
      this._texture.loadContentsOf(this.imgElement)
      this._draw = this._canvas.draw(this._texture)
    },
    resetImg () {
      this.valueOfBrightness = 0
      this.valueOfContrast = 0
      this.valueOfHue = 0
      this.valueOfSaturation = 0
      this.valueOfNoise = 0
      this.valueOfSepia = 0
      this.usmRadius = 0
      this.usmStrength = 0
      this.isCurves = false
      this.isDenoise = false
      this.opBtn = []
      this.resetProperty()
      this._draw.update()
      this.newSrc = this._canvas.toDataURL('image/jpeg', 0.8)
    }
  },
  mounted () {
    this.loading = false
    var img = new Image()
    img.setAttribute('crossOrigin', 'anonymous')
    img.onload = () => {
      this.$nextTick(() => {
        this.getWebGLElements()
      })
    }

    img.src = this.imgUrl
    this.newSrc = this.imgUrl
    this.setImg()
  },
  watch: {
    imgUrl: function (val) {
      if (val) {
        var img = new Image()
        img.setAttribute('crossOrigin', 'anonymous')
        img.onload = () => {
          this.$nextTick(() => {
            this.getWebGLElements()
          })
        }
        img.src = val
        this.newSrc = val
        this.setImg()
      }
    }
  }
}
</script>
<style lang="less" scoped>

</style>
```