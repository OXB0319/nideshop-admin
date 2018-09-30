<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item :to="{ name: 'dashboard' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>商城运营</el-breadcrumb-item>
                <el-breadcrumb-item>{{infoForm.id ? '编辑首页Banner' : '添加首页Banner'}}</el-breadcrumb-item>
            </el-breadcrumb>
            <div class="operation-nav">
                <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
            </div>
        </div>
        <div class="content-main">
            <div class="form-table-box">
                <el-form ref="infoForm" :rules="infoRules" :model="infoForm" label-width="120px">

                    <el-form-item prop="deletedPics">
                    </el-form-item>

                    <el-form-item label="标题" prop="name">
                        <el-input v-model="infoForm.name"></el-input>
                    </el-form-item>
                    <el-form-item label="封面" prop="image_url">
                        <el-upload class="image-uploader" name="pic"
                                   :action="actionGoodsPic"
                                   :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{type:'image_url', preUrl:infoForm.image_url}">
                            <img v-if="infoForm.image_url" :src="infoForm.image_url" class="image-show" >
                            <i v-else class="el-icon-plus image-uploader-icon"></i>
                        </el-upload>
                        <div class="form-tip">图片尺寸：750*415</div>
                    </el-form-item>

                    <el-form-item label="跳转类型">
                      <el-radio-group v-model="infoForm.media_type">
                        <el-radio-button label="0" :key="0">商品</el-radio-button>
                        <el-radio-button label="1" :key="1">专题</el-radio-button>
                        <el-radio-button label="9" :key="9">无</el-radio-button>
                      </el-radio-group>
                    </el-form-item>

                    <el-form-item label="跳转id">
                     <el-input type='number' v-model="infoForm.media_id" :min="0"></el-input>
                    </el-form-item>

                    <el-form-item label="排序">
                        <el-input-number v-model="infoForm.ad_position_id" :min="1" :max="1000"></el-input-number>
                    </el-form-item>
                    <el-form-item label="启用">
                        <el-switch v-model="infoForm.enabled"></el-switch>
                    </el-form-item>

                    <el-form-item>
                        <el-button type="primary" @click="onSubmitInfo">确定保存</el-button>
                        <el-button @click="goBackPage">取消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>
    </div>
</template>

<script>
import api from "@/config/api";
export default {
  data() {
    return {
      uploaderHeader: {
        "X-Nideshop-Token": localStorage.getItem("token") || ""
      },
      actionGoodsPic: api.rootUrl + "/upload/brandPic",
      infoForm: {
        id: 0,
        name: "",
        ad_position_id: 100,
        media_type:0, // 跳转类型：0-商品，1-专题
        media_id:0, // 跳转id，如果meida_type为0则为商品id，如果为1则为专题id
        enabled: true,
        image_url: "",
        deletedPics: [], // 删除的图片
        price_info : 0,
      },
      infoRules: {
        name: [{ required: true, message: "请输入标题", trigger: "blur" }],
        subtitle: [
          { required: true, message: "请输入子标题", trigger: "blur" }
        ],
        image_url: [
          { required: true, message: "请选择缩略图", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    goBackPage() {
      this.$router.go(-1);
    },
    onSubmitInfo() {
      this.$refs["infoForm"].validate(valid => {
        if (valid) {
          this.axios.post("banner/store", this.infoForm).then(response => {
            if (response.data.errno === 0) {
              this.$message({
                type: "success",
                message: "保存成功"
              });
              this.$router.go(-1);
            } else {
              this.$message({
                type: "error",
                message: "保存失败"
              });
            }
          });
        } else {
          return false;
        }
      });
    },

    handleUploadImageSuccess(res, file) {
      if (res.errno === 0) {
        switch (res.data.params.type) {
          // 封面
          case "image_url":
          {
            let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除

              if(preUrl)
              {
                this.infoForm.deletedPics.push(preUrl);
              }
              this.infoForm.image_url = res.data.fileUrl;
          }
            break;
        }
      }
    },

     /**
     *  处理删除图片
     * @param type 类型 desc、poster、gallery
     */
    handleDeleteImg(type, index) {
      switch (type) {
        case "content":
          {
            if (index >= 0 && index < this.infoForm.content.length) {
              let deletedUrls = this.infoForm.content.splice(index, 1);
              this.infoForm.deletedPics.push(deletedUrls[0]);
            }
          }
          break;
      }
    },

    getInfo() {
      if (this.infoForm.id <= 0) {
        return false;
      }

      //加载专题详情
      let that = this;
      this.axios
        .get("banner/info", {
          params: {
            id: that.infoForm.id
          }
        })
        .then(response => {
          let resInfo = response.data.data;
          resInfo.enabled = resInfo.enabled ? true : false;
          Object.assign(that.infoForm, resInfo);
        //   that.infoForm = resInfo;
        });
    }
  },
  components: {},
  mounted() {
    this.infoForm.id = this.$route.query.id || 0;
    this.getInfo();
    console.log(api);
  }
};
</script>

<style>
.image-uploader {
  height: 105px;
  display: inline-block;
}
.image-uploader .el-upload {
  border: 1px solid #d9d9d9;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.image-uploader .el-upload:hover {
  border-color: #20a0ff;
}

.image-uploader .image-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 187px;
  height: 105px;
  line-height: 105px;
  text-align: center;
}

.image-uploader .image-show {
  width: 187px;
  height: 105px;
  display: block;
  float: left;
}

.image-delete {
  overflow: hidden;
  bottom: 10px;
}

.image-uploader.new-image-uploader {
  font-size: 28px;
  color: #8c939d;
  width: 165px;
  height: 105px;
  line-height: 105px;
  text-align: center;
}

.image-uploader.new-image-uploader .image-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 165px;
  height: 105px;
  line-height: 105px;
  text-align: center;
}

.image-uploader.new-image-uploader .image-show {
  width: 165px;
  height: 105px;
  display: block;
}
</style>
