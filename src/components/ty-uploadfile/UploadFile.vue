<template>
  <div class="ty-upload-file">
    <mt-cell v-if="showUpload" :title="attachmentLabel" is-link>
      <VueFileUpload label="上传附件" :url="serverUrl" :autoUpload="autoUpload" ref="vueFileUploader" :events="cbEvents"></VueFileUpload>
    </mt-cell>
    <mt-cell v-else :title="attachmentLabel" class="fileupload-display"></mt-cell>
    <div class="file-list">
      <TyFileList :files="fileList" :isShowDel="showUpload" v-on:del="delHandler"></TyFileList>
    </div>
  </div>
</template>
<script>
import VueFileUpload from 'vue-file-upload'
const TyFileList = () => import('@/components/common/TyFileList')

export default {
  name: 'UploadFile',
  props: {
    showUpload: {
      type: Boolean,
      default: true
    },
    files: {
      type: Array,
      default: []
    }
  },
  components: {
    VueFileUpload,
    TyFileList
  },
  watch: {
    files(newValue) {
      this.fileList = newValue
    }
  },
  data() {
    return {
      title: '附件',
      fileList: this.files,
      autoUpload: false,
      serverUrl: this.$fileServer + 'file/upload',
      cbEvents: {
        onCompleteUpload: (file, response, status, header) => {
          if (response.code === 0) {
            this.fileList[this.fileList.length - 1].name = file.file.name
            this.fileList[this.fileList.length - 1].id = response.data
            this.fileList[this.fileList.length - 1].isUpload = false
          } else {
            TyCommon.info(response.message)
          }
        },
        onAddFileSuccess: (file) => {
          //todo 可以在这里提前获取文件信息
          if (file.file.size > 5120000) {
            TyCommon.info('上传文件大小5M以下')
            file.cancel()
          } else {
            let size = (file.size / 1000).toFixed(2)
            this.fileList.push({ isUpload: true, process: 0, name: file.file.name, size: size + 'K', id: '' })
            file.upload()
          }
        },
        onErrorUpload: (file, response, status, headers) => {
          if (response.code !== 0) {
            this.fileList.pop()
            TyCommon.info(response.message)
          }
        },
        onProgressUpload: (file, progress) => {
          console.log(progress)
          this.fileList[this.fileList.length - 1].process = progress
        }
      }
    }
  },
  computed: {
    attachmentLabel: function() {
      return this.title + '(' + this.fileList.length + ')'
    }
  },
  mounted() { },
  methods: {
    delHandler(index) {
      if (this.fileList[index].isUpload) {
        this.fileList.splice(index, 1)
        return
      }
      this.$http.post('file/delete?fileIds=' + this.fileList[index].id, {}).then((res) => {
        if (res.data.code === 0) {
          this.fileList.splice(index, 1)
        } else {
          TyCommon.info(res.data.message)
        }
      }).catch((error) => {
        TyCommon.info(error.message)
      })
      this.fileList.splice(index, 1)
    }
  }
}
</script>
<style lang="less">
.ty-upload-file {
  .fileupload-button {
    color: #999 !important;
    background-color: white !important;
    border-color: white !important;
    font-size: 15px !important;
    padding: 0px !important;
  }
  .fileupload-display {
    .mint-cell-text {
      color: #999;
    }
  }
}
</style>


