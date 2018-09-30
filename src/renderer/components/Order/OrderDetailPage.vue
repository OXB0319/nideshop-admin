<template>
    <div class="content-page">
        <div class="content-nav">
            <div class="breadcrumb">
                订单详情
            </div>
            <div class="operation-nav">
                <el-button type="primary" @click="goBackPage" size="small" icon="arrow-left">返回列表</el-button>
            </div>
        </div>
        <div class="content-main">
            <div class="form-table-box" v-loading="pageLoading" element-loading-text="拼命加载中">
               
               <el-form ref="infoForm" :model="infoForm" label-width="120px">

                    <el-form-item prop="deletedPics">
                    </el-form-item>

                    <el-form-item label="订单号" prop="order_sn">
                        {{infoForm.order_sn}}
                    </el-form-item>
                    <el-form-item label="订单状态" prop="order_status">
                        {{infoForm.orderStatusText}}

                        <el-button v-if="infoForm.order_status==0||infoForm.order_status==1||infoForm.order_status==2" type="primary" @click="modifyOrderStatus">{{ infoForm.order_status == 0 ? '已与买家确认' : infoForm.order_status == 1 ? '买家已付款' : infoForm.order_status == 2 ? '已发货' : '' }}</el-button>
                    </el-form-item>

                    <el-form-item label="订单价格" prop="order_price">
                        ¥{{infoForm.actual_price}}
                    </el-form-item>

                    <el-form-item label="下单时间" prop="add_time">
                        {{ infoForm.add_time | formatDate }}
                    </el-form-item>
                    
                    <el-form-item v-if="infoForm.order_status==2||infoForm.order_status==3" label="快递公司" prop="shipper_id">
                      <el-select v-model="infoForm.expressInfo.shipper_id" placeholder="请选择快递公司">
                        <el-option v-for="item in infoForm.shipperInfos" :key="item.id" :label="item.name" :value="item.id"></el-option>
                      </el-select>
                    </el-form-item>

                    <el-form-item v-if="infoForm.order_status==2||infoForm.order_status==3" label="快递单号" prop="logistic_code">
                      <el-input v-model="infoForm.expressInfo.logistic_code" placeholder='请填写快递单号'></el-input>
                    </el-form-item>

                    <el-form-item label="收货人" prop="consignee">
                        {{infoForm.consignee}}
                    </el-form-item>
                    <el-form-item label="收货地址">
                        {{infoForm.detailAddress}}
                    </el-form-item>

                    <el-form-item label="联系方式" prop="mobile">
                         {{infoForm.mobile}}
                    </el-form-item>

                    <el-form-item label="买家留言" prop="postscript">
                        {{infoForm.postscript}}
                    </el-form-item>

                    <el-form-item label="商品详情">
                        
                        <div v-for="item in infoForm.goodsInfos" :key="item.list_pic_url">
                            <div class="image-uploader"> 
                                <img v-if="item.list_pic_url" :src="item.list_pic_url" class="image-show">
                                {{item.name}}
                                ¥{{item.retail_price}}
                            </div>
                        </div>
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
import DateUtil from "@/js/DateUtil";

export default {
  data() {
    return {
      pageLoading: false,
      infoForm: {
        id: 0,
        order_sn: 0,
        order_status: 0,
        orderStatusText:'',
        order_price: 0,
        add_time: 0,
        consignee: "",
        mobile: "",
        postscript: "",
        detailAddress: "",
        goodsInfos:[],
        shipperInfos:[],
        expressInfo:{
          logistic_code:'',
          shipper_id:null,
        },
      }
    };
  },
  methods: {
    goBackPage() {
      this.$router.go(-1);
    },
    getInfo() {
      if (this.infoForm.id <= 0) {
        return false;
      }

      //加载订单详情
      let that = this;
      this.axios
        .get("order/info", {
          params: {
            id: that.infoForm.id
          }
        })
        .then(response => {
          console.log(response.data);
          let resInfo = response.data.data;
          Object.assign(that.infoForm, resInfo);
          this.pageLoading = false;
        });
    },

    getShipperInfos()
    {
      let that = this;
      this.axios
        .get("order/shipperInfos", {})
        .then(response => {
          console.log(response.data);
          let resInfo = response.data.data;
          that.infoForm.shipperInfos= resInfo;
        });
    },


    /** 修改订单状态 */
    modifyOrderStatus() {

        let that = this;
        this.$confirm('确定要执行此操作?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
            if(that.infoForm.order_status==0) //当前为等待确认状态，处理为已确认状态
            {
                that.infoForm.order_status=1;
            } 
            else if(that.infoForm.order_status==1) //当前为已确认，等待付款状态，处理为已付款，等待发货状态
            {
                that.infoForm.order_status=2;
            }
            else if(that.infoForm.order_status==2) //当前为已付款等待发货状态，处理为已发货状态
            {
                that.infoForm.order_status=3;
            }

            that.axios.get("order/orderStatusToText", {
                    params: {
                        order_status: that.infoForm.order_status
                    }
                    })
                    .then(response => {
                        that.infoForm.orderStatusText = response.data.data;
                    });
        });
    },

    /** 关闭订单 */
    closeOrder() {},

    onSubmitInfo() {
        this.$refs["infoForm"].validate(valid => {
        if (valid) {
          this.axios.post("order/store", this.infoForm).then(response => {
            if (response.data.errno === 0) {
              this.$message({
                type: "success",
                message: "保存成功"
              });
              this.$router.go(-1);
            } else {
              this.$message({
                type: "error",
                message: response.data.errmsg
              });
            }
          });
        } else {
          return false;
        }
      });
    }
  },
  components: {},
  mounted() {
    console.log(this.$route.query);
    this.infoForm.id = this.$route.query.id || 0;
    this.getInfo();
    this.getShipperInfos();
  },

  filters: {
    formatDate(time) {
      var date = new Date(time * 1000 * 1000);
      return DateUtil.formatDate(date, "yyyy-MM-dd hh:mm:ss");
    }
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
</style>
