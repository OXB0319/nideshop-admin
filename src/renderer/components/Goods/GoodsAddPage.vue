<template>
  <div class="content-page">

    <div class="content-nav">
      <el-breadcrumb class="breadcrumb" separator="/">
        <el-breadcrumb-item :to="{ name: 'dashboard' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>{{infoForm.id ? '编辑商品' : '添加商品'}}</el-breadcrumb-item>
      </el-breadcrumb>
      <div class="operation-nav">
        <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
      </div>
    </div>

    <div class="content-main">
      <div class="form-table-box">
        <el-form ref="infoForm" :rules="infoRules" :model="infoForm" label-width="120px">
          
          
          <el-form-item prop="deletedDescPics">
          </el-form-item>

          <el-form-item label="商品名称" prop="name">
            <el-input v-model="infoForm.name"></el-input>
          </el-form-item>
          <el-form-item label="商品简单描述" name="goods_brief">
            <el-input v-model="infoForm.goods_brief" placeholder="填写后会在商品详情页的商品名称下展现，非必填"></el-input>
          </el-form-item>

          <el-form-item label="推荐类型">
            <el-radio-group v-model="infoForm.stock_type">
              <el-radio-button label="0" :key="0">海外直购(amazon)</el-radio-button>
              <el-radio-button label="1" :key="1">海外产地直达(丹麦)</el-radio-button>
              <el-radio-button label="2" :key="2">花、多肉</el-radio-button>
              <el-radio-button label="3" :key="3">日本代购</el-radio-button>
              <el-radio-button label="9" :key="9">其他</el-radio-button>
          </el-radio-group>
          </el-form-item>

          <el-form-item label="所属分类" prop="categorySelected">
            <el-cascader  :options="categoryOptions" :props="categoryCascaderConfig" placeholder="请选择分类" v-model="categorySelected" @change="handleChange" >
            </el-cascader>
          </el-form-item>
          <!-- <el-form-item label="所属分类" prop="name">
            <el-select v-model="infoForm.category_id" placeholder="请选择所属分类">
                <el-option v-for="item in categoryOptions" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item> -->

          <!-- <el-form-item label="所属品牌" prop="brand_id" :rules="[
          { type:'number', required: true,  message: '请选择所属品牌'}]">
            <el-select v-model="infoForm.brand_id" placeholder="请选择品牌">
              <el-option v-for="item in brandOptions" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item> -->

          <!-- <el-form-item label="商品简介" prop="goods_desc">
            <el-input type="textarea" v-model="infoForm.goods_desc" :rows="3"></el-input>
            <div class="form-tip"></div>
          </el-form-item> -->

          <el-form-item label="商品封面" prop="list_pic_url">
            <el-upload class="image-uploader" name="pic"
                       :action="actionGoodsPic" :show-file-list="true"
                       :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:0,type:'poster', preUrl:infoForm.list_pic_url}">
              <img v-if="infoForm.list_pic_url" :src="infoForm.list_pic_url" class="image-show">
              <i v-else class="el-icon-plus image-uploader-icon"></i>
              <!-- <div class="form-tip">图片尺寸：750*420</div> -->
            </el-upload>
          </el-form-item>

          <el-form-item label="京东外链" prop="link_jd" style="display:none;">
            <el-input v-model="infoForm.link_jd" placeholder="填写后会在商品详情页展现，非必填"></el-input>
          </el-form-item>

          <el-form-item label="亚马逊外链" prop="link_amazon"  style="display:none;">
            <el-input v-model="infoForm.link_amazon" placeholder="填写后会在商品详情页展现，非必填"></el-input>
          </el-form-item>

          <el-form-item label="商品属性" name="attribute" prop="attribute">
            <div v-for="(item, curIndex) in infoForm.attribute" :key="item.hashCode">
              <el-input style="width:200px;margin-right:5px;margin-bottom:5px;" placeholder="请输入属性名称，如：颜色" v-model="item.name" ></el-input>
              <el-input style="width:200px" placeholder="请输入属性值，如：白色" v-model="item.value"></el-input>
              <el-button size="small" type="danger" @click="handleRemoveAttribute(curIndex)">删除</el-button>    
            </div>

            <el-button size="small" type="primary" @click="handleAddAttribute()">新增</el-button>    
          </el-form-item>

          <el-form-item label="价格¥" prop="retail_price" :rules="[{ type:'number', required: true,  message: '请填写价格', trigger:'blur'}]">
            <el-input type='number' v-model.number="infoForm.retail_price"></el-input>
          </el-form-item>

          <el-form-item label="规格/库存" prop="goods_number" :rules="[{ type:'number', required: true,  message: '请填写库存', trigger:'blur'}]">
            <el-input-number v-model.number="infoForm.goods_number" :min='0' :max='99999'></el-input-number>
          </el-form-item>

          <el-form-item label="推荐类型">
            <!-- <el-checkbox-group v-model="recommendCheckedList">
              <el-checkbox label="新品" v-model="infoForm.is_new"></el-checkbox>
              <el-checkbox label="人气" v-model="infoForm.is_hot"></el-checkbox>
            </el-checkbox-group> -->
            <el-radio-group v-model="recommendChecked">
              <el-radio-button label="0" :key="1">无</el-radio-button>
              <el-radio-button label="1" :key="2">新品</el-radio-button>
              <el-radio-button label="2" :key="3">人气</el-radio-button>
          </el-radio-group>

          </el-form-item>
          <el-form-item label="上架">
            <el-switch on-text="" off-text="" v-model="infoForm.is_on_sale"></el-switch>
          </el-form-item>
          <el-form-item label="排序">
            <el-input-number v-model="infoForm.sort_order" :min="1" :max="1000"></el-input-number>
          </el-form-item>

          <el-form-item label="商品banner" prop="gallery">
              <div v-for="(item, curIndex) in infoForm.gallery" :key="item.img_url">
                <el-upload  class="image-uploader" name="pic"
                            :action="actionGoodsPic" :show-file-list="true"
                            :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:curIndex, type:'banner',preUrl:item.img_url}">
                            
                    <img v-if="item.img_url" :src="item.img_url" class="image-show">
                    <i v-else class="el-icon-plus image-uploader-icon"></i>

                </el-upload>                
                <el-button class='image-delete' size="small" type="danger" @click="handleDeleteImg('banner', curIndex)">删除</el-button>    
                
              </div>

            <el-upload v-if="infoForm.gallery.length<8" class="image-uploader" name="pic"
                        :action="actionGoodsPic" :show-file-list="true"
                        :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:infoForm.gallery.length,type:'banner'}">
                <i class="el-icon-plus image-uploader-icon"></i>
            </el-upload>
          </el-form-item>

          <el-form-item label="商品详情">
              <div v-for="(item, curIndex) in infoForm.goods_desc" :key="item">
                <el-upload  class="image-uploader" name="pic"
                            :action="actionGoodsPic" :show-file-list="true"
                            :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:curIndex,type:'desc', preUrl:item}">
                    <img v-if="item" :src="item" class="image-show">
                    <i v-else class="el-icon-plus image-uploader-icon"></i>
                </el-upload>                
                <el-button class='image-delete' size="small" type="danger" @click="handleDeleteImg('desc', curIndex)">删除</el-button>    
                <el-button v-if="curIndex!=0" class='image-delete' size="small" type="primary" @click="handleImgSort('desc', curIndex, true)">↑上移</el-button>    
                <el-button v-if="curIndex!=infoForm.goods_desc.length-1" class='image-delete' size="small" type="primary" @click="handleImgSort('desc', curIndex, false)">↓下移</el-button>
              </div>

            <el-upload class="image-uploader" name="pic"
                        :action="actionGoodsPic" :show-file-list="true"
                        :on-success="handleUploadImageSuccess" :headers="uploaderHeader" :data="{index:infoForm.goods_desc.length,type:'desc'}">
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
import { log } from "util";
export default {
  data() {
    return {
      uploaderHeader: {
        "X-Nideshop-Token": localStorage.getItem("token") || ""
      },

      categoryOptions: [], // 分类数据
      categoryCascaderConfig: {
        value: "id",
        label: "name",
        children: "children"
      }, // 分类数据配置
      categorySelected: [], // 默认选中的分类

      brandOptions: [], // 品牌数据

      recommendChecked: 0, // 推荐选中数据

      actionGoodsPic: api.rootUrl + "/upload/brandPic",

      infoForm: {
        id: 0,

        name: "",
        goods_brief : "",
        list_pic_url: "",
        goods_desc: [],
        deletedDescPics: [], // 删除的详情图片

        pic_url: "",
        sort_order: 100,
        is_show: true,
        floor_price: 0,
        is_new: false,
        new_pic_url: "",
        new_sort_order: 10,

        goods_number: 0,
        //brand_id: null,
        category_id: 0,
        is_on_sale: true,
        is_hot: false,
        stock_type: 0,
        retail_price: 0,

        link_jd: '', // 京东外链
        link_amazon: '', // 亚马逊外链

        gallery: [], // 商品banner对象集合
        deletedGalleries: [], // 删除的banner对象集合
        attribute : []
      },
      infoRules: {
        name: [{ required: true, message: "请输入名称", trigger: "blur" }],
        attribute: [{ type:'array', required: true, message: "请输入完整的商品属性", trigger: "blur" ,validator: (rule, value, callback) => {
              if (value.length > 0) {
                  for(let i=0;i<value.length;++i)
                  {
                    let item = value[i];
                    if(item.name=='' || item.value=='')
                    {
                      callback(new Error());
                      return;
                    }
                  }
                  callback();
                }else{
                  callback();
                }
            }}],
            
        brand: [
          {
            required: true,
            type: "number",
            message: "请选择品牌",
            trigger: "blur"
          }
        ],
        goods_desc: [
          { required: false, message: "请输入简介", trigger: "blur" }
        ],
        list_pic_url: [
          {
            type: "string",
            required: true,
            message: "请选择商品图片",
            trigger: "blur",
            transform(value) {
              return value;
            }
          }
        ],
        gallery: [
          {
            type: "array",
            required: true,
            message: "请选择商品banner图片",
            trigger: "blur",
            transform(value) {
              return value;
            }
          }
        ],
        goods_number_rule: [
          {
            type: "number",
            required: true,
            message: "请填写库存",
            trigger: "blur"
          }
        ],

        categorySelected: [
          {
            type: "array",
            required: true,
            message : "请选择所属分类",
            validator: (rule, value, callback) => {
              if (this.categorySelected.length <= 0) {
                callback(new Error());
              }else{
                callback();
              }
            },
            trigger: "change"
          }
        ]
      }
    };
  },
  methods: {
    goBackPage() {
      this.$router.go(-1);
    },

    handleRemoveAttribute(index)
    {
        if(index>=0&&index<this.infoForm.attribute.length)
        {
          this.infoForm.attribute.splice(index, 1);
        }
    },

    handleAddAttribute()
    {
        let obj = {"name":"", "value":""};
        this.infoForm.attribute.push(obj);
    },

    handleChange(item) {
      this.infoForm.category_id = item[item.length - 1];
    },
    onSubmitInfo() {
      if (this.recommendChecked == 1) {
        this.infoForm.is_new = true;
        this.infoForm.is_hot = false;
      } else if (this.recommendChecked == 2) {
        this.infoForm.is_new = false;
        this.infoForm.is_hot = true;
      } else {
        this.infoForm.is_new = false;
        this.infoForm.is_hot = false;
      }

      this.$refs["infoForm"].validate(valid => {
        if (valid) {
          this.axios.post("goods/store", this.infoForm).then(response => {
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
          //商品封面
          case "poster":
          {
              let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除
              if(preUrl)
              {
                this.infoForm.deletedDescPics.push(preUrl);
              }
              this.infoForm.list_pic_url = res.data.fileUrl;
          }
            
            break;

          // 商品banner
          case "banner":
            {
              let index = parseInt(res.data.params.index);

              let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除
              if(preUrl)
              {
                this.infoForm.deletedDescPics.push(preUrl);
              }

              if (index >= this.infoForm.gallery.length) {
                // 新增图片
                let galleryObj = new Object();
                galleryObj.id = 0;
                galleryObj.goods_id = this.infoForm.id;
                galleryObj.img_url = res.data.fileUrl;
                this.infoForm.gallery.push(galleryObj);
              } else {
                // 替换图片
                this.infoForm.gallery[index].img_url = res.data.fileUrl;
              }
            }
            break;

          // 商品描述图片
          case "desc":
            {
              let index = parseInt(res.data.params.index);
              let preUrl = res.data.params.preUrl; // 之前的url，如果存在，需要删除

              if(preUrl)
              {
                this.infoForm.deletedDescPics.push(preUrl);
              }

              if (index >= this.infoForm.goods_desc.length) {
                // 新增图片
                this.infoForm.goods_desc.push(res.data.fileUrl);
              } else {
                // 替换图片
                this.infoForm.goods_desc.splice(index, 1, res.data.fileUrl);
              }
            }
            break;
        }
      }
    },

    /**
     *  处理删除图片
     * @param type 类型 desc、banner
     */
    handleDeleteImg(type, index) {
      switch (type) {
        case "desc":
          {
            if (index >= 0 && index < this.infoForm.goods_desc.length) {
              let deletedUrls = this.infoForm.goods_desc.splice(index, 1);
              this.infoForm.deletedDescPics.push(deletedUrls[0]);
            }
          }
          break;

        case "banner":
          {
            if (index >= 0 && index < this.infoForm.gallery.length) {
              let deletedObj = this.infoForm.gallery.splice(index, 1);
              this.infoForm.deletedGalleries.push(deletedObj[0]);
            }
          }
          break;
      }
    },

    /**
     * 处理图片排序
     * @param type 类型 desc、banner
     * @param index 下标
     * @param upOrDown 上移或者下移
     */
    handleImgSort(type, index, upOrDown)
    {
      switch (type) {
        case "desc":
          {

            let arr = this.infoForm.goods_desc;
            if (index >= 0 && index < arr.length) {
              if(upOrDown)
              {
                if(index==0)
                {
                  return;
                }
                [arr[index-1], arr[index]]=[arr[index], arr[index-1]];
              }
              else{
                if(index==arr.length-1)
                {
                  return;
                }
                [arr[index+1], arr[index]]=[arr[index], arr[index+1]];
              }
            }
            this.infoForm.goods_desc = [];
            this.infoForm.goods_desc = arr;
          }
          break;
        case "banner":
          {
            let arr = this.infoForm.gallery;
            if (index >= 0 && index < arr.length) {
              if(upOrDown)
              {
                if(index==0)
                {
                  return;
                }
                [arr[index-1], arr[index]]=[arr[index], arr[index-1]];
              }
              else{
                if(index==arr.length-1)
                {
                  return;
                }
                [arr[index+1], arr[index]]=[arr[index], arr[index+1]];
              }
            }
            this.infoForm.gallery = [];
            this.infoForm.gallery = arr;
          }
          break;
      }
    },

    getCascaderCategory() {
      this.axios.get("category/cascader").then(response => {
        this.categoryOptions = this.categoryOptions.concat(response.data.data);
        this.handleCategorySelected();
      });
    },

    handleCategorySelected() {
      if (this.categoryOptions.length > 0 && this.infoForm.category_id > 0) {
        this.categoryOptions.map(item => {
          item.children.map(itemChild => {
            if (itemChild.id === this.infoForm.category_id) {
              this.categorySelected = [item.id, this.infoForm.category_id];
              return;
            }
          });
        });
      }
    },

    getBrand() {
      this.axios.get("brand", { params: { size: 500 } }).then(response => {
        this.brandOptions = this.brandOptions.concat(response.data.data.data);
        this.handleCategorySelected();
      });
    },

    getInfo() {
      if (this.infoForm.id <= 0) {
        return false;
      }

      //加载商品详情
      let that = this;
      this.axios
        .get("goods/info", {
          params: {
            id: that.infoForm.id
          }
        })
        .then(response => {
          let resInfo = response.data.data;
          resInfo.is_new = resInfo.is_new ? true : false;
          resInfo.is_hot = resInfo.is_hot ? true : false;
          resInfo.is_show = resInfo.is_show ? true : false;
          resInfo.is_on_sale = resInfo.is_on_sale ? true : false;
          resInfo.brand_id = resInfo.brand_id == 0 ? null : resInfo.brand_id;

          if (resInfo.is_new) {
            this.recommendChecked = 1;
          } else if (resInfo.is_hot) {
            this.recommendChecked = 2;
          } else {
            this.recommendChecked = 0;
          }

          that.infoForm = Object.assign(that.infoForm, resInfo);

          this.handleCategorySelected();
        });
    }
  },
  components: {},
  mounted() {
    this.getCascaderCategory();
    this.getBrand();
    this.infoForm.id = this.$route.query.id || 0;
    this.getInfo();
    console.log(api);
  }
};
</script>

<style>
.image-uploader {
  height: 130px;
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
  height: 100px;
  line-height: 100px;
  text-align: center;
}

.image-uploader .image-show {
  width: 187px;
  height: 100px;
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
  height: 100px;
  line-height: 100px;
  text-align: center;
}

.image-uploader.new-image-uploader .image-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 165px;
  height: 100px;
  line-height: 100px;
  text-align: center;
}

.image-uploader.new-image-uploader .image-show {
  width: 165px;
  height: 100px;
  display: block;
}
</style>
