<template>
	<div class="content-page">
		<div class="content-nav">
			<el-breadcrumb class="breadcrumb" separator="/">
				<el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
				<el-breadcrumb-item>用户管理</el-breadcrumb-item>
				<el-breadcrumb-item>反馈列表</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="content-main">
			<div class="filter-box">
				<el-form :inline="true" :model="filterForm" class="demo-form-inline">
					<el-form-item label="会员名称">
						<el-input v-model="filterForm.name" placeholder="会员名称"></el-input>
					</el-form-item>
					<el-form-item label="会员昵称">
						<el-input v-model="filterForm.nickname" placeholder="会员昵称"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" @click="onSubmitFilter">查询</el-button>
					</el-form-item>
				</el-form>
			</div>
			<div class="form-table-box">
				<el-table :data="tableData" style="width: 100%" border stripe>
					<el-table-column prop="id" label="ID" width="50">
					</el-table-column>
					<el-table-column prop="username" label="会员名称">
					</el-table-column>
					<el-table-column prop="nickname" label="呢称">
					</el-table-column>
					<el-table-column prop="gender" label="性别" width="80">
						<template scope="scope">
							{{ scope.row.gender == 1 ? '男' : scope.row.gender == 2 ? '女' : '未知'}}
						</template>
					</el-table-column>
					<el-table-column prop="avatar" label="头像" width="100">
						<template scope="scope">
							<img v-if="scope.row.avatar" :src="scope.row.avatar" class="image-show">
						</template>
					</el-table-column>
					<el-table-column prop="user_email" label="联系方式">
					</el-table-column>
					<el-table-column prop="msg_type" label="反馈类型">
						<template scope="scope">
							{{ scope.row.msg_type == 1 ? '商品相关' : scope.row.msg_type == 2 ? '物流状况' : scope.row.msg_type == 3 ? '客户服务' : scope.row.msg_type == 4 ? '优惠活动' : scope.row.msg_type == 5 ? '功能异常' : scope.row.msg_type == 6 ? '产品建议' : scope.row.msg_type == 7 ? '其他' : '未知'}}
						</template>
					</el-table-column>
					<el-table-column prop="msg_content" label="反馈内容" width="400">
					</el-table-column>
					<el-table-column prop="msg_time" label="反馈时间">
						<template scope="scope">
							{{ scope.row.msg_time | formatDate }}
						</template>
					</el-table-column>
				</el-table>
			</div>
			<div class="page-box">
				<el-pagination @current-change="handlePageChange" :current-page="page" :page-size="10" layout="total, prev, pager, next, jumper" :total="total">
				</el-pagination>
			</div>
		</div>
	</div>
</template>

<script>


import DateUtil from "@/js/DateUtil";
import api from "@/config/api";
export default {
	data() {
		return {
			isShowDelete : false,
			page: 1,
			total: 0,
			filterForm: {
				name: '',
				nickname:'',
			},
			tableData: []
		}
	},
	methods: {
		handlePageChange(val) {
			this.page = val;
			//保存到localStorage
			localStorage.setItem('userPage', this.page)
			localStorage.setItem('userFilterForm', JSON.stringify(this.filterForm));
			this.getList()
		},
		onSubmitFilter() {
			this.page = 1
			this.getList()
		},
		getList() {
			this.axios.get('feedback', {
				params: {
					page: this.page,
					name: this.filterForm.name,
					nickname: this.filterForm.nickname,
				}
			}).then((response) => {
                this.tableData = response.data.data.data
                this.page = response.data.data.currentPage
                this.total = response.data.data.count
			})
		}
	},
	components: {

	},
	mounted() {
		 this.isShowDelete = api.isShowDelete;
		this.getList();
	},

	filters: {
        formatDate(time) {
			var date = new Date(time*1000);
            return DateUtil.formatDate(date, "yyyy-MM-dd hh:mm:ss");
        }
    }

}

</script>

<style>
.image-show {
  width: 60px;
  height: 60px;
  display: block;
}

</style>
