<template>
  <div>
    <el-upload
      :disabled="imgLoading"
      action="/"
      list-type="picture-card"
      ref="upload"
      :limit="limit"
      multiple
      :show-file-list="false"
      :on-remove="onRemove"
      :on-success="onSuccess"
      :on-exceed="onExceed"
      :http-request="onHttpRequest"
      :before-upload="beforeUpload"
      :file-list="fileList"
      :accept="accept"
      class="compoments-ggn-upload"
      :hidden-push="pushStatus"
    >
      <div
        class="chrysanthemum"
        v-loading="imgLoading"
        element-loading-spinner="el-icon-loading"
        element-loading-background="rgba(0, 0, 0, 0.8)"
      >
        <i class="el-icon-plus"></i>
      </div>
    </el-upload>
    <div class="tips">
      温馨提示：<br />
      1、图片支持jpg、png、jpeg格式，大小在10M以内<br />
      2、最多可上传200张图片
    </div>
    <div>
      <ul class="imageListshow el-upload-list--picture-card">
        <draggable
          style="
            display: flex;
            width: 870px;
            flex-wrap: wrap;
            justify-content: flex-start;
          "
          v-model="banner_list"
          :options="{ animation: 60 }"
        >
          <li
            class="ul_li_list"
            v-for="(item, index) in banner_list"
            :key="index"
          >
            <div class="el-upload-list__item is-success">
              <topic-image
                ref="topicimageList"
                :src="item.relurl && !item.url ? item.relurl : item.url"
                :TYPE="item.relurl && !item.url ? '1' : '0'"
              ></topic-image>
              <progress-el
                @deleteImage="deleteImage"
                @stopuload="stopuload"
                @listentoprog="listentoprog"
                ref="progressel"
                :index="index"
                :item="item"
                :progressShow="item.progressShow"
                :imgListBig="banner_list"
              ></progress-el>
            </div>
            <div style="width: 100px">
              <topic-upload-label
                v-if="item.showlabel"
                @listentoLabel="listentoLabel"
                ref="topic"
                :index="index"
                :dynamicTagslabel="item.data"
              ></topic-upload-label>
              <span
                v-else
                style="
                  font-size: 12px;
                  width: 160px;
                  display: block;
                  text-align: center;
                "
              >
                {{ item.name }}
              </span>
            </div>
          </li>
        </draggable>
      </ul>
    </div>
  </div>
</template>

<script>
import OSS from "ali-oss";
import { getSignature, fileUpload } from "@api/common";
import draggable from "vuedraggable";
import topicUploadLabel from "@components/topicUploadLabel.vue";
import progressEl from "@components/progressEl.vue";
import ggnImage from "@components/ggn-image.vue";
import topicImage from "@components/topicImage.vue";

let delayTimer;
export default {
  props: {
    limit: {
      type: Number,
      default: 200,
    },
    size: {
      type: [String, Number],
      default: 10,
    },
    placeholder: String,
    disabled: Boolean,
    accept: {
      type: String,
      default: "image/jpg,image/jpeg,image/png",
    },
    fileList: {
      type: Array,
      default: () => [],
    },
    imgLoading: Boolean,
    callback: Function,
    index: String,
    uploadTYpe: String,
  },
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      count: 0,
      showProgress: false,
      progress: 0,
      banner_list: [],
      dynamicTags: [],
      allData: {},
      records: {},
      labelArrays: [],
    };
  },
  computed: {
    pushStatus() {
      const vmProps = this.$props;
      if (vmProps.fileList.length === 0) return false;
      if (vmProps.disabled) return true;
      if (vmProps.fileList.length >= vmProps.limit) return true;
      return false;
    },
  },
  methods: {
    /**
     * 上传之前的回调, 返回false则中止上传, 用于文件大小等的逻辑判断
     */
    beforeUpload(file) {
      var checkSize = file.size / 1024 / 1024 <= this.size; /* 文件大小单位为mb */
      // // 图片格式--过滤后缀为webp的图片
      // var imglist = ["image/webp","image/WEBP"];
      // // 进行图片匹配
      // var result = imglist.some(function (item) {
      //   return item == file.type;
      // });
      // if (result) {
      //   this.$notify.error({
      //     title: "错误",
      //     message:"文件类型必须为.jpg、.png、.jpeg类型",
      //   });
      //   return false;
      // }

			const checkType = this.accept.split(",").includes(file.type); /* 文件类型 */
      if (!(checkSize && checkType)){
        this.$notify.error({
          title: "错误",
         message: ` ${!checkSize ? '上传文件大小不能超过' + this.size + ' MB  ' : ''}
                    ${!checkType ? '文件类型必须为.jpg、.png、.jpeg类型':''} `
        });
        return false;
      }
    },

    // 在图片上传中显示图片名称，上传完成显示标签框
    listentoprog(flag, index) {
      this.$data.banner_list[index].showlabel = flag;
    },

    //图片列表中可以修改各自的标签列表，互不影响
    listentoLabel(data, index) {
      this.$data.banner_list[index].data = data;
      this.$emit("toFatherParams", this.banner_list);
    },

    /**
     * 自定义上传
     */
    async onHttpRequest(e) {
      const _this = this;
      const _e = e;
      /**
       * 获取签名
       */
      const signature = await getSignature({ isVideo: false });
      /**
       * 创建链接对象
       */
      const client = new OSS({
        region: signature.region,
        accessKeyId: signature.accessid,
        accessKeySecret: signature.accessKeySecret,
        bucket: signature.bucket,
      });
      /**
       * 后台录入文件名等
       */
      fileUpload({
        uuid: signature.uuid,
        fileName: e.file.name,
      }).then((res) => {
        const _res = res;
        /**
         * 开始上传---阿里云分片上传
         */
        async function multipartUpload() {
          client
            .multipartUpload(
              `${signature.dir}${res.data.result[0].original}`,
              e.file,
              {
                progress: function (p, checkpoint, res) {
                  _this.$refs.progressel.map((e, i) => {
                    if (e.item.size < 1024 * 1024 * 10) {
                      //把标签进度传入进度组件中
                      _this.$refs.progressel[i].loadProgress = Math.floor(
                        1 * 100
                      );
                      _this.$refs.progressel[i].progressFlag = false;
                    } else if (
                      checkpoint != undefined &&
                      e.item.uid == checkpoint.file.uid
                    ) {
                      _this.$refs.progressel[i].loadProgress = Math.floor(
                        p * 100
                      );
                      if (p == 1) {
                        _this.$refs.progressel[i].progressFlag = false;
                      }
                      _this.$refs.progressel[i].checkpointel = checkpoint;
                    }
                  });
                },
                // 分片大小
                partSize: 1024 * 1024 * 10,
              }
            )
            .then((data) => {
              _this.asdb(_res, _e);
            })
            .catch((err) => {
              console.log(err);
            });
        }
        multipartUpload();
      });
    },

    asdb(_res, _e) {
      let _resdata = JSON.parse(JSON.stringify(_res.data.result[0]));
      _resdata = {
        ..._resdata,
        uid: _e.file.uid,
      };
      if (this.banner_list.length > 0) {
        this.banner_list = this.banner_list.map((item) => {
          if (item.raw != undefined && item.raw.uid == _resdata.uid) {
            item = {
              ...item,
              uniqueId: _resdata.uniqueId,
              relurl: _resdata.filePath,
            };
            return item;
          } else {
            return item;
          }
        });
      }
      this.$emit("toFatherParams", this.banner_list);
    },

    /*
     * 取消上传-删除图片
     */
    stopuload(checkdata, item) {
      this.$refs.upload.handleRemove(item, item);
    },
    /**
     * 获取文件名后缀
     */
    getFileSuffix(fileName) {
      return fileName.substr(fileName.lastIndexOf("."));
    },

    deleteImage(file) {
      this.banner_list.map((e, i) => {
        if (file.uniqueId && file.uniqueId == e.uniqueId) {
          this.banner_list.splice(i, 1);
        } else if (!file.uniqueId && file.id == e.id) {
          this.banner_list.splice(i, 1);
        }
      });
      this.commnFunction();
      this.$emit("toFatherParams", this.banner_list);
    },

    commnFunction() {
      this.banner_list = this.banner_list.map((e, i) => {
        e = {
          ...e,
          progressShow: false,
        };
        return e;
      });
    },

    /**
     * 移除
     */
    onRemove(file, fileList) {
      this.banner_list.map((e, i) => {
        if (file.uid == e.uid) {
          this.banner_list.splice(i, 1);
        }
      });
      this.$emit("toFatherParams", this.banner_list);
    },
    /**
     * 上传成功
     */
    onSuccess(response, file, fileList) {
      let _file = file;
      _file = {
        ..._file,
        data: JSON.parse(JSON.stringify(this.labelArrays)),
        showlabel: false,
        progressShow: true,
      };
      this.banner_list.push(_file);
      this.$emit("toFatherParams", this.banner_list);
    },
    /**
     * 超出提示
     */
    onExceed(files, fileList) {
      this.$message.error("超过最大上传数量!");
    },
    /**
     * 上传土坯那异常处理函数
     */
    catchCallback() {
      this.count = 0;
      this.$props.callback(this.$props.index);
    },
  },
  components: {
    draggable,
    topicUploadLabel,
    progressEl,
    ggnImage,
    topicImage,
  },
};
</script>

<style lang="less">
.compoments-ggn-upload {
  &[hidden-push="true"] {
    & > .el-upload {
      &.el-upload--picture-card {
        display: none;
      }
    }
  }
  & > .chrysanthemum {
    width: 146px;
    height: 146px;
    z-index: 99999;
    /deep/ .el-loading-mask {
      display: flex;
      flex-direction: row;
      align-items: center;
      /deep/ .el-loading-spinner {
        z-index: 999999;
        position: absolute !important;
        top: -50%;
      }
    }
  }
  .tips {
    line-height: 22px;
    font-size: 12px;
    position: absolute;
    top: 37px;
    left: 235px;
  }
  .ul_li_list {
    float: left;
  }
  .button-tag {
    width: 160px;
  }
  .el-tag--small {
    margin-bottom: 3px;
  }
  .disabel {
    display: none;
  }
}
.imageListshow {
  list-style: none;
  overflow: hidden;
  padding-left: 0px;
  margin: 0;
  display: inline;
  vertical-align: top;
  .el-upload-list__item {
    overflow: hidden;
    background-color: #fff;
    border: 1px solid #c0ccda;
    border-radius: 6px;
    box-sizing: border-box;
    width: 160px !important;
    height: 160px !important;
    margin: 10px 8px 0px 0;
    display: inline-block;
    transition: opacity 0.3s;
    .el-upload-list__item-thumbnail {
      width: 100%;
      height: 100%;
    }
    .el-upload-list__item-status-label {
      position: absolute;
      right: -15px;
      top: -6px;
      width: 40px;
      height: 24px;
      background: #13ce66;
      text-align: center;
      transform: rotate(45deg);
      box-shadow: 0 0 1pc 1px rgba(0, 0, 0, 0.2);
    }
  }
  .el-upload-list--picture-card .el-upload-list__item .el-icon-check,
  .el-upload-list--picture-card .el-upload-list__item .el-icon-circle-check {
    color: #fff;
  }
  .el-upload-list--picture-card .el-upload-list__item-status-label i {
    font-size: 12px;
    margin-top: 11px;
    transform: rotate(-45deg);
  }
  .el-upload-list--picture-card .el-upload-list__item-actions {
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    cursor: default;
    text-align: center;
    color: #fff;
    opacity: 0;
    font-size: 20px;
    background-color: rgba(0, 0, 0, 0.5);
  }
}
</style>