<template>
  <div class="upload-container">
    <el-button
      :style="{ background: color, borderColor: color }"
      icon="el-icon-upload"
      size="mini"
      type="primary"
      @click="dialogVisible = true"
    >
      上传图片
    </el-button>
    <el-dialog
      :visible.sync="dialogVisible"
      append-to-body
      custom-class="uploadImg"
      center
    >
      <el-upload
        :multiple="true"
        :file-list="fileList"
        :show-file-list="true"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
        :before-upload="beforeUpload"
        class="editor-slide-upload"
        :action="url"
        :headers="headers"
        list-type="picture-card"
      >
        <el-button size="small" type="primary">
          点击上传图片
        </el-button>
      </el-upload>
      <div class="btn">
        <el-button @click="dialogVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="handleSubmit">
          提交
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  // import { getToken } from 'api/qiniu'
  import { getToken } from "@/utils/auth";
  export default {
    name: "EditorSlideUpload",
    props: {
      color: {
        type: String,
        default: "#1890ff"
      }
    },
    data() {
      return {
        dialogVisible: false,
        listObj: {},
        fileList: [],
        headers: {
          serveResource: "web_school",
          token: getToken(),
          smail: "*_~"
        },
        url:
          process.env.NODE_ENV !== "production"
            ? "/dev-api/echo-art/upload/uploadImages"
            : "/echo-art/upload/uploadImages"
      };
    },
    methods: {
      checkAllSuccess() {
        return Object.keys(this.listObj).every(
          item => this.listObj[item].hasSuccess
        );
      },
      handleSubmit() {
        const arr = Object.keys(this.listObj).map(v => this.listObj[v]);
        if (!this.checkAllSuccess()) {
          this.$message.warning(
            "请等待全部图片上传成功"
            //"Please wait for all images to be uploaded successfully. If there is a network problem, please refresh the page and upload again!"
          );
          return;
        }
        this.$emit("successCBK", arr);
        this.listObj = {};
        this.fileList = [];
        this.dialogVisible = false;
      },
      handleSuccess(response, file) {
        const uid = file.uid;
        const objKeyArr = Object.keys(this.listObj);
        for (let i = 0, len = objKeyArr.length; i < len; i++) {
          if (this.listObj[objKeyArr[i]].uid === uid) {
            this.listObj[objKeyArr[i]].url = response.data.url;
            this.listObj[objKeyArr[i]].hasSuccess = true;
            return;
          }
        }
      },
      handleRemove(file) {
        const uid = file.uid;
        const objKeyArr = Object.keys(this.listObj);
        for (let i = 0, len = objKeyArr.length; i < len; i++) {
          if (this.listObj[objKeyArr[i]].uid === uid) {
            delete this.listObj[objKeyArr[i]];
            return;
          }
        }
      },
      beforeUpload(file) {
        const _self = this;
        const _URL = window.URL || window.webkitURL;
        const fileName = file.uid;
        this.listObj[fileName] = {};
        return new Promise((resolve, reject) => {
          const img = new Image();
          img.src = _URL.createObjectURL(file);
          img.onload = function() {
            _self.listObj[fileName] = {
              hasSuccess: false,
              uid: file.uid,
              width: this.width,
              height: this.height
            };
          };
          resolve(true);
        });
      }
    }
  };
</script>

<style lang="scss" scoped>
  .editor-slide-upload {
    margin-bottom: 20px;
    /deep/ .el-upload--picture-card {
      width: 100%;
    }
  }
</style>
<style lang="scss">
  .el-dialog__wrapper {
    .uploadImg {
      // height: 200px;
      .el-dialog__body {
        .el-button:not(.el-button--primary) {
          color: #606266;
          border: 1px solid #dcdfe6;
          &:hover {
            color: #1890ff;
          }
        }
        .btn {
          display: flex;
          align-items: center;
          justify-content: flex-end;
        }
      }
    }
  }
</style>
