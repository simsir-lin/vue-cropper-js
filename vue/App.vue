<template>
  <div id="app">
    <vue-file-picker-wrapper @confirm="cutImage" @error="showMsg">
      <button>选择文件</button>
    </vue-file-picker-wrapper>
    <cropper @cancel="cancelCut" @confirm="confirmCut" @error="showMsg" ref="cropper"></cropper>
    <img v-if="image" :src="image" class="image">
  </div>
</template>

<script>
import vueFilePickerWrapper from 'vue-file-picker-wrapper'
import cropper from '../src/cropper'
export default {
  name: 'app',
  components: {
    vueFilePickerWrapper,
    cropper
  },
  data() {
    return {
      image: ''
    }
  },
  methods: {
    showMsg(res) {
      alert(res.msg)
    },
    cutImage(file) {
      this.$refs.cropper.start(file)
    },
    cancelCut() {
      this.cutFile = null
    },
    confirmCut(imageFile) {
      this.$set(this, 'image', imageFile.url)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  text-align: center;
}
.image {
  width: 300px;
  height: auto;
  margin-top: 20px;
}
</style>
