<template>
<div class="cropper" :class="{'cropper-mobile': isMobile, 'cropper-pc': !isMobile}" v-show="croppable">
  <div>
    <img id="image" :src="url" :style="imageStyle">
  </div>
  <div class="btn btn-cancel" @click="cancel">取消</div>
  <div class="btn btn-confirm" @click="confirm">完成</div>
</div>
</template>

<script>
import Cropper from 'cropperjs'
import {
  Loading
} from 'element-ui'
export default {
  name: 'cropper',
  data() {
    return {
      cropper: null,
      croppable: false,
      imageStyle: '',
      url: '',
      loadingInstance: null,
      file: null
    }
  },
  props: {
    aspectRatio: {
      type: Number,
      default: 1
    },
    isMobile: {
      type: Boolean,
      default: false
    }
  },
  mounted() {
    this.cropper = new Cropper(document.getElementById('image'), {
      aspectRatio: this.aspectRatio,
      viewMode: 1,
      background: false,
      zoomable: false,
      ready: () => {
        console.log('cropper ready')
      }
    })
  },
  methods: {
    start(file) {
      if (this._getFileRawType(file) != 'image') {
        this._error('只有图片才能进行裁剪')
        return
      }
      this.file = file
      this.cropper && this.cropper.setAspectRatio(this.aspectRatio)
      this.getObjectURL(file).then((url) => {
        this.url = url
        if (this.cropper) {
          this.cropper.replace(this.url)
          this.croppable = true
        }
      })
    },
    getObjectURL(file) {
      return new Promise((resolve, reject) => {
        if (!FileReader) {
          this._error('当前浏览器版本过低无法裁剪')
          this.$emit('confirm', this.file)
          reject(new Error())
          return
        }
        this.loadingInstance = Loading.service({
          text: ''
        })
        const reader = new FileReader()
        reader.onload = (event) => {
          let image = document.createElement('img')
          image.onload = () => {
            let maxWidth = window.innerWidth
            let maxHeight = window.innerHeight
            let imageStyle = `max-width:${maxWidth}px;max-height:${maxHeight}px;`
            if (this.isMobile) {
              if (image.width === image.height) {
                imageStyle += `height:${maxWidth * 0.8}px;width:${maxWidth * 0.8}px;`
              } else {
                if (image.width > image.height) {
                  // 扁
                  imageStyle += `width:${maxWidth * 0.9}px;`
                }
                if (image.width < image.height) {
                  // 长
                  imageStyle += `height:${maxWidth}px;`
                }
              }
            } else {
              if (image.width === image.height) {
                imageStyle += `height:${maxHeight * 0.7}px;width:${maxHeight * 0.7}px;`
              } else {
                if (image.width > image.height) {
                  // 扁
                  imageStyle += `width:${maxHeight}px;`
                }
                if (image.width < image.height) {
                  // 长
                  imageStyle += `height:${maxHeight * 0.9}px;`
                }
              }
            }
            this.imageStyle = imageStyle
            this.loadingInstance.close()
            resolve(event.target.result)
          }
          image.src = event.target.result
        }
        reader.readAsDataURL(file)
      })
    },
    cancel() {
      this.croppable = false
      this.$emit('cancel')
    },
    confirm() {
      if (!this.croppable) {
        return
      }
      let destURL = this.cropper.getCroppedCanvas().toDataURL('image/jpeg')
      let file = this._blobToFile(destURL)
      this.croppable = false
      this.$emit('confirm', file)
    },
    _error(msg) {
      this.$emit('error', {
        msg
      })
    },
    _getFileRawType(file) {
      return file.type.split('/')[0]
    },
    _blobToFile(dataurl) {
      let theBlob = this._dataURLtoBlob(dataurl)
      theBlob.lastModifiedDate = new Date()
      theBlob.name = this.file.name
      theBlob.url = dataurl
      return theBlob
    },
    _dataURLtoBlob(dataurl) {
      let arr = dataurl.split(',')
      let mime = arr[0].match(/:(.*?);/)[1]
      let bstr = atob(arr[1])
      let n = bstr.length
      let u8arr = new Uint8Array(n)
      if (!u8arr) {
        this._error('裁剪失败，检测当前浏览器版本过低，请升级浏览器')
        return
      }
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new Blob([u8arr], {
        type: mime
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.cropper {
    z-index: 9990;
    position: fixed;
    box-sizing: border-box;
    left: 0;
    top: 0;
    background: rgba(0, 0, 0, 0.9);
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    .btn {
        z-index: 9991;
        position: fixed;
        &:hover {
            cursor: pointer;
        }
    }
}
.cropper-pc {
    width: 100%;
    height: 100%;
    .btn {
        font-size: 14px;
        bottom: 30px;
        font-weight: 400;
        color: #333;
        width: 82px;
        height: 36px;
        background-color: #fff;
        border-radius: 4px;
        text-align: center;
        line-height: 36px;
    }
    .btn-cancel {
        left: 50px;
    }
    .btn-confirm {
        right: 50px;
    }
}
.cropper-mobile {
    width: 100vw;
    height: 100vh;
    .btn {
        font-size: 14px;
        bottom: 15px;
        font-weight: 400;
        color: rgba(255,255,255,1);
    }
    .btn-cancel {
        left: 35px;
    }
    .btn-confirm {
        right: 35px;
    }
}
</style>
<style>
/*!
 * Cropper.js v1.5.5
 * https://fengyuanchen.github.io/cropperjs
 *
 * Copyright 2015-present Chen Fengyuan
 * Released under the MIT license
 *
 * Date: 2019-08-04T02:26:27.232Z
 */

.cropper-container {
  direction: ltr;
  font-size: 0;
  line-height: 0;
  position: relative;
  -ms-touch-action: none;
  touch-action: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.cropper-container img {
  display: block;
  height: 100%;
  image-orientation: 0deg;
  max-height: none !important;
  max-width: none !important;
  min-height: 0 !important;
  min-width: 0 !important;
  width: 100%;
}

.cropper-wrap-box,
.cropper-canvas,
.cropper-drag-box,
.cropper-crop-box,
.cropper-modal {
  bottom: 0;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
}

.cropper-wrap-box,
.cropper-canvas {
  overflow: hidden;
}

.cropper-drag-box {
  background-color: #fff;
  opacity: 0;
}

.cropper-modal {
  background-color: rgba(0, 0, 0, 0.1);
  opacity: 0.5;
}

.cropper-view-box {
  display: block;
  height: 100%;
  outline: 1px solid #39f;
  outline-color: rgba(51, 153, 255, 0.75);
  overflow: hidden;
  width: 100%;
}

.cropper-dashed {
  border: 0 dashed #eee;
  display: block;
  opacity: 0.5;
  position: absolute;
}

.cropper-dashed.dashed-h {
  border-bottom-width: 1px;
  border-top-width: 1px;
  height: calc(100% / 3);
  left: 0;
  top: calc(100% / 3);
  width: 100%;
}

.cropper-dashed.dashed-v {
  border-left-width: 1px;
  border-right-width: 1px;
  height: 100%;
  left: calc(100% / 3);
  top: 0;
  width: calc(100% / 3);
}

.cropper-center {
  display: block;
  height: 0;
  left: 50%;
  opacity: 0.75;
  position: absolute;
  top: 50%;
  width: 0;
}

.cropper-center::before,
.cropper-center::after {
  background-color: #eee;
  content: ' ';
  display: block;
  position: absolute;
}

.cropper-center::before {
  height: 1px;
  left: -3px;
  top: 0;
  width: 7px;
}

.cropper-center::after {
  height: 7px;
  left: 0;
  top: -3px;
  width: 1px;
}

.cropper-face,
.cropper-line,
.cropper-point {
  display: block;
  height: 100%;
  opacity: 0.1;
  position: absolute;
  width: 100%;
}

.cropper-face {
  background-color: #fff;
  left: 0;
  top: 0;
}

.cropper-line {
  background-color: #39f;
}

.cropper-line.line-e {
  cursor: ew-resize;
  right: -3px;
  top: 0;
  width: 5px;
}

.cropper-line.line-n {
  cursor: ns-resize;
  height: 5px;
  left: 0;
  top: -3px;
}

.cropper-line.line-w {
  cursor: ew-resize;
  left: -3px;
  top: 0;
  width: 5px;
}

.cropper-line.line-s {
  bottom: -3px;
  cursor: ns-resize;
  height: 5px;
  left: 0;
}

.cropper-point {
  background-color: #39f;
  height: 5px;
  opacity: 0.75;
  width: 5px;
}

.cropper-point.point-e {
  cursor: ew-resize;
  margin-top: -3px;
  right: -3px;
  top: 50%;
}

.cropper-point.point-n {
  cursor: ns-resize;
  left: 50%;
  margin-left: -3px;
  top: -3px;
}

.cropper-point.point-w {
  cursor: ew-resize;
  left: -3px;
  margin-top: -3px;
  top: 50%;
}

.cropper-point.point-s {
  bottom: -3px;
  cursor: s-resize;
  left: 50%;
  margin-left: -3px;
}

.cropper-point.point-ne {
  cursor: nesw-resize;
  right: -3px;
  top: -3px;
}

.cropper-point.point-nw {
  cursor: nwse-resize;
  left: -3px;
  top: -3px;
}

.cropper-point.point-sw {
  bottom: -3px;
  cursor: nesw-resize;
  left: -3px;
}

.cropper-point.point-se {
  bottom: -3px;
  cursor: nwse-resize;
  height: 20px;
  opacity: 1;
  right: -3px;
  width: 20px;
}

@media (min-width: 768px) {
  .cropper-point.point-se {
    height: 15px;
    width: 15px;
  }
}

@media (min-width: 992px) {
  .cropper-point.point-se {
    height: 10px;
    width: 10px;
  }
}

@media (min-width: 1200px) {
  .cropper-point.point-se {
    height: 5px;
    opacity: 0.75;
    width: 5px;
  }
}

.cropper-point.point-se::before {
  background-color: #39f;
  bottom: -50%;
  content: ' ';
  display: block;
  height: 200%;
  opacity: 0;
  position: absolute;
  right: -50%;
  width: 200%;
}

.cropper-invisible {
  opacity: 0;
}

/*
.cropper-bg {
  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC');
} */

.cropper-hide {
  display: block;
  height: 0;
  position: absolute;
  width: 0;
}

.cropper-hidden {
  display: none !important;
}

.cropper-move {
  cursor: move;
}

.cropper-crop {
  cursor: crosshair;
}

.cropper-disabled .cropper-drag-box,
.cropper-disabled .cropper-face,
.cropper-disabled .cropper-line,
.cropper-disabled .cropper-point {
  cursor: not-allowed;
}
</style>
