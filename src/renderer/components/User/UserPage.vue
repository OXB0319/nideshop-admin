<template>
	<div class="content-page">
		<div class="content-nav">
			<el-breadcrumb class="breadcrumb" separator="/">
				<el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
				<el-breadcrumb-item>用户管理</el-breadcrumb-item>
				<el-breadcrumb-item>会员列表</el-breadcrumb-item>
			</el-breadcrumb>
			<div class="operation-nav">
				<router-link to="/dashboard/user/add">
					<!--<el-button type="primary" icon="plus">添加会员</el-button>-->
				</router-link>
			</div>
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
					<el-table-column prop="id" label="ID" width="100">
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
					<el-table-column prop="mobile" label="手机号">
					</el-table-column>
					<el-table-column prop="register_time" label="注册时间">
						<template scope="scope">
							{{ scope.row.register_time | formatDate }}
						</template>
					</el-table-column>
					<el-table-column prop="register_time" label="最近登录时间">
						<template scope="scope">
							{{ scope.row.last_login_time | formatDate }}
						</template>
					</el-table-column>
					<el-table-column label="操作" width="140">
						<template scope="scope">
							<!--<el-button size="small" @click="handleRowEdit(scope.$index, scope.row)">编辑</el-button>-->
							<el-button size="small" v-if="isShowDelete" type="danger" @click="handleRowDelete(scope.$index, scope.row)">删除</el-button>
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
		handleRowEdit(index, row) {
			this.$router.push({ name: 'user_add', query: { id: row.id } })
		},
		handleRowDelete(index, row) {

			this.$confirm('确定要删除?', '提示', {
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning'
			}).then(() => {

				this.axios.post('user/destory', { id: row.id }).then((response) => {
					console.log(response.data)
					if (response.data.errno === 0) {
						this.$message({
							type: 'success',
							message: '删除成功!'
						});

						this.getList();
					}
				})


			});
		},
		onSubmitFilter() {
			this.page = 1
			this.getList()
		},
		getList() {
			this.axios.get('user', {
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
