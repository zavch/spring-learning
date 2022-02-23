<template>
  <uploader :options="options" :file-status-text="statusText" :autoStart="true" class="uploader-example" ref="uploader"
            @file-added="fileAdded" @files-added="filesAdded" @file-complete="fileComplete" @complete="complete">
    <uploader-btn :attrs="imgAttrs">选择图片</uploader-btn>
    <p>图片列表</p>
    <uploader-files selector-type="img-selector"></uploader-files>
    <br>
    <uploader-btn :attrs="fileAttrs">选择文件</uploader-btn>
    <p>文件列表</p>
    <uploader-files selector-type="file-selector"></uploader-files>
  </uploader>
</template>

<script>
  import axios from 'axios'
  import qs from 'qs'

  export default {
    data() {
      return {
        options: {
          target: '/boot/uploader/chunk',
          testChunks: true,
          simultaneousUploads: 1,
          chunkSize: 1 * 1024 * 1024,
          // checkChunkUploadedByResponse: function (chunk, message) {
          //   let objMessage = {}
          //   try {
          //     objMessage = JSON.parse(message)
          //   } catch (e) {
          //   }
          //   // fake response
          //   // objMessage.uploaded_chunks = [2, 3, 4, 5, 6, 8, 10, 11, 12, 13, 17, 20, 21]
          //   // check the chunk is uploaded
          //   return (objMessage.uploaded_chunks || []).indexOf(chunk.offset + 1) >= 0
          // },
          allowDuplicateUploads: true
        },
        imgAttrs: {
          accept: 'image/*',
          id: 'img-selector'
        },
        fileAttrs: {
          id: 'file-selector'
        },
        statusText: {
          success: '成功了',
          error: '出错了',
          uploading: '上传中',
          paused: '暂停中',
          waiting: '等待中'
        }
      }
    },
    methods: {
      // 上传完成
      complete() {
        console.log('complete', arguments)
      },
      // 一个根文件（文件夹）成功上传完成。
      fileComplete() {
        console.log('file complete', arguments)
        const file = arguments[0].file;
        axios.post('/boot/uploader/mergeFile', qs.stringify({
          filename: file.name,
          identifier: arguments[0].uniqueIdentifier,
          totalSize: file.size,
          type: file.type
        })).then(function (response) {
          console.log(response);
        }).catch(function (error) {
          console.log(error);
        });
      },
      fileAdded(file) {
        console.log("fileAdded")
      },
      filesAdded(files, fileList) {
        console.log("filesAdded")
        let imgCounts = uploader.files.filter(f => f.selectorType === "img-selector").length;
        let fileCounts = uploader.files.filter(f => f.selectorType === "file-selector").length;
        let size = uploader.getSize();
        console.log("imgCounts:",imgCounts,",fileCounts", fileCounts, ",size:", size);
        for(let file of files) {
          size += file.size;
          if (size > 10 * 1024 * 1024) {
              files.ignored = true
              alert("总大小限制为10M");
              break;
          }
          if (file.selectorType === "img-selector") {
              imgCounts++;
          } else {
              fileCounts++;
          }
          if (imgCounts > 3) {
            files.ignored = true
            alert("图片最多3张");
            break;
          }
          if (fileCounts > 3) {
            files.ignored = true
            alert("文件最多3个");
            break;
          }
        }
      }
    },
    mounted() {
      this.$nextTick(() => {
        window.uploader = this.$refs.uploader.uploader
      })
    }
  }
</script>

<style>
  .uploader-example {
    width: 1000px;
    padding: 15px;
    margin: 40px auto 0;
    font-size: 12px;
    box-shadow: 0 0 10px rgba(62, 146, 203, 0.4);
  }

  .uploader-example .uploader-btn {
    margin-right: 4px;
  }

  .uploader-example .uploader-list {
    max-height: 440px;
    overflow: auto;
    overflow-x: hidden;
    overflow-y: auto;
  }
</style>
