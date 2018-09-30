<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item :to="{ name: 'dashboard' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>管理员信息</el-breadcrumb-item>
            </el-breadcrumb>
            <div class="operation-nav">
                <el-button type="primary" @click="goBackPage" icon="arrow-left">返回</el-button>
            </div>
        </div>
        <div class="content-main">
            <div class="form-table-box">
                <el-form ref="infoForm" :rules="infoRules" :model="infoForm" label-width="120px">
                    <el-form-item label="用户名" name="username">
                        <el-input v-model="infoForm.username" :disabled="true"></el-input>
                    </el-form-item>
                    <el-form-item label="原密码" prop="oldPassword">
                        <el-input v-model="infoForm.oldPassword" ></el-input>
                    </el-form-item>
                    <el-form-item label="新密码" prop="newPassword">
                        <el-input v-model="infoForm.newPassword"></el-input>
                    </el-form-item>
                    <el-form-item label="上次登录时间">
                         {{ infoForm.last_login_time | formatDate }}
                    </el-form-item>
                    <el-form-item label="上次登录IP" >
                        {{infoForm.last_login_ip}}
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
    import api from '@/config/api';
    import DateUtil from "@/js/DateUtil";

    export default {
        data() {
            return {
                uploaderHeader: {
                    'X-Nideshop-Token': localStorage.getItem('token') || '',
                },
                infoForm: {
                    id: 0,
                    oldPassword: "",
                    newPassword: "",
                    last_login_time: '',
                    last_login_ip:'',
                },
                infoRules: {
                    oldPassword: [
                        { required: true, message: '请输入原密码', trigger: 'blur' },
                    ],
                    newPassword: [
                        {required: true, min: 6, message: '密码不得低于6个字符', trigger: 'blur'},
                    ],
                },
            }
        },
        methods: {
            goBackPage() {
                this.$router.go(-1);
            },
            onSubmitInfo() {
                this.$refs['infoForm'].validate((valid) => {
                    if (valid) {
                        this.axios.post('auth/store', this.infoForm).then((response) => {
                            if (response.data.errno === 0) {
                                this.$message({
                                    type: 'success',
                                    message: response.data.data
                                });

                                // 注销登录
                                localStorage.clear();
                                this.$router.replace({name: 'login'});
                            } else {
                                this.$message({
                                    type: 'error',
                                    message: response.data.errmsg
                                })
                            }
                        })
                    } else {
                        return false;
                    }
                });
            },

            getInfo() {
                
                let userInfo = localStorage.getItem('userInfo');

                //加载详情
                let that = this
                this.axios.get('auth/info', {
                    params: {
                        id: userInfo.id
                    }
                }).then((response) => {
                    let resInfo = response.data.data;
                    Object.assign(that.infoForm, resInfo);
                })
            }
        },
        components: {},
        mounted() {
            this.infoForm.id = this.$route.query.id || 0;
            this.getInfo();
            
        },
         filters: {
            formatDate(time) {
                var date = new Date(time * 1000 );
                return DateUtil.formatDate(date, "yyyy-MM-dd hh:mm:ss");
            }
        }
        
    }

</script>

<style>
    .image-uploader{
        height: 105px;
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
