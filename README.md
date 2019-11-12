# vue-cropper-js
✂️ cropper.js for vue

### 使用
1. 安装
```bash
import vueCropperJs from 'vue-cropper-js-simplify'
```
2. 在html中即可使用组件
``` html
<vue-cropper-js @cancel="cancelCut" @confirm="confirmCut" ref="cropper"></vue-cropper-js>
```
3. 详细用法见`vue/App.vue`

### Props

| name     | description              | type     | default value |
| :---------------- | :----------------------- | :------  | :------------ |
| aspectRatio             | 裁剪比例                 | Number    | 1          |
| isMobile             | 是否为移动端                 | Boolean    | false      |

### events

| name     | description              | parameters     |
| :---------------- | :----------------------- | :------  |
| cancel            | 点击取消                 |     |
| confirm            | 点击确认                 |  file   |
| error            | 出现错误消息                 |  { msg }   |
