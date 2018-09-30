<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item :to="{ name: 'dashboard' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>商城运营</el-breadcrumb-item>
                <el-breadcrumb-item>{{infoForm.id ? '编辑专题' : '添加专题'}}</el-breadcrumb-item>
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

                    <el-form-item label="标题" prop="title">
                        <el-input v-model="infoForm.title"></el-input>
                    </el-form-item>
                    <el-form-item label="子标题" prop="subtitle">
                        <el-input type="textarea" v-model="infoForm.subtitle" :rows="3"></el-input>
                        <div class="form-tip"></div>
                    </el-form-item>
                    <el-form-item label="缩略图" prop="scene_pic_url">
                        <el-upload class="image-uploader" name="pic"
                                   :action="actionGoodsPic"
                                   :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{type:'scene_pic_url', preUrl:infoForm.scene_pic_url}">
                            <img v-if="infoForm.scene_pic_url" :src="infoForm.scene_pic_url" class="image-show" >
                            <i v-else class="el-icon-plus image-uploader-icon"></i>
                        </el-upload>
                        <div class="form-tip">图片尺寸：750*415</div>
                    </el-form-item>

                    
                    <el-form-item label="价格¥">
                        <el-input type='number' v-model="infoForm.price_info" :min="0" :max="999999"></el-input>
                    </el-form-item>

                    <el-form-item label="排序">
                        <el-input-number v-model="infoForm.sort_order" :min="1" :max="1000"></el-input-number>
                    </el-form-item>
                    <el-form-item label="启用">
                        <el-switch v-model="infoForm.is_show"></el-switch>
                    </el-form-item>

                    <el-form-item label="专题详情">
                        <div v-for="(item, curIndex) in infoForm.content" :key="item">
                            <el-upload  class="image-uploader" name="pic"
                                        :action="actionGoodsPic" :show-file-list="true"
                                        :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:curIndex,type:'content',preUrl:item}">
                                <img v-if="item" :src="item" class="image-show">
                                <i v-else class="el-icon-plus image-uploader-icon"></i>
                            </el-upload>                
                            <el-button class='image-delete' size="small" type="danger" @click="handleDeleteImg('content', curIndex)">删除</el-button>    
                        </div>

                        <el-upload class="image-uploader" name="pic"
                                    :action="actionGoodsPic" :show-file-list="true"
                                    :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:infoForm.content.length,type:'content'}">
                            <i class="el-icon-plus image-uploader-icon"></i>
                        </el-upload>
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
        title: "",
        subtitle: "",
        sort_order: 100,
        is_show: true,
        scene_pic_url: "",
        content : [],
        deletedPics: [], // 删除的图片
        price_info : 0,
      },
      infoRules: {
        title: [{ required: true, message: "请输入标题", trigger: "blur" }],
        subtitle: [
          { required: true, message: "请输入子标题", trigger: "blur" }
        ],
        scene_pic_url: [
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
          this.axios.post("topic/store", this.infoForm).then(response => {
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
          // 专题封面
          case "scene_pic_url":
          {
            let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除

              if(preUrl)
              {
                this.infoForm.deletedPics.push(preUrl);
              }
              this.infoForm.scene_pic_url = res.data.fileUrl;
          }
            
            break;

          // 商品描述图片
          case "content":
            {
              let index = parseInt(res.data.params.index);
              let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除

              if(preUrl)
              {
                this.infoForm.deletedPics.push(preUrl);
              }

              if (index >= this.infoForm.content.length) {
                // 新增图片
                this.infoForm.content.push(res.data.fileUrl);
              } else {
                // 替换图片
                this.infoForm.content.splice(index, 1, res.data.fileUrl);
              }
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
        .get("topic/info", {
          params: {
            id: that.infoForm.id
          }
        })
        .then(response => {
          let resInfo = response.data.data;
          resInfo.is_show = resInfo.is_show ? true : false;
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
