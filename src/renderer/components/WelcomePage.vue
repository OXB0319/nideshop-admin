<template>
  <div class="content-page">
    <div class="content-nav">
      <el-breadcrumb class="breadcrumb" separator="/">
        <el-breadcrumb-item :to="{ path: '/dashboard' }">后台主页</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="content-main">
        欢迎使用骨朵儿海外精选后台管理系统{{version}} (上次登录时间：{{userInfo.last_login_time | formatDate }} 上次登录ip：{{userInfo.last_login_ip}})
        
    </div>
  </div>
</template>

<script>

  import api from '@/config/api';
  import DateUtil from "@/js/DateUtil";
  export default {
    data() {
      return {
        dialogVisible: false,
        version : api.version,
        userInfo : Object(),
      }
    },

    components: {
      
    },

    mounted() {
        let userInfo = JSON.parse(localStorage.getItem('userInfo'));

        this.userInfo = userInfo;
    },
    filters: {
      formatDate(time) {
          if(!time)
          {
            return "";
          }
          var date = new Date(time * 1000 );
          return DateUtil.formatDate(date, "yyyy-MM-dd hh:mm:ss");
      }
  }
  }
</script>

<style scoped>

</style>
