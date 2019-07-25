<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters" @keyup.enter.native="getContractList()">
				<el-form-item>
					<!--	v-model双向绑定		-->
					<el-input v-model="filters.name" placeholder="合作方名字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getContractList()">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" style="right: 0px" @click="gotoAddContract()">新增合同</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表   v-loading="listLoading"-->
		<el-table :data="contractList" v-loading="contractListLoading" highlight-current-row style="width: 100%;">
			<el-table-column type="index" width="40">
			</el-table-column>
			<el-table-column prop="contractNum" label="合同编号" width="120" sortable>
			</el-table-column>
			<el-table-column prop="partner" label="合作商名称" width="130" sortable>
			</el-table-column>
			<el-table-column prop="contractName" label="合同名称" width="170" sortable>
			</el-table-column>
			<el-table-column prop="versionNum" label="版本号" width="120" sortable>
			</el-table-column>
            <el-table-column prop="status" label="生效?" min-width="100" :formatter="formatSex" sortable>
            </el-table-column>
			<el-table-column prop="startDate" label="生效日期" min-width="120" sortable>
			</el-table-column>
			<el-table-column prop="endDate" label="生效日期" min-width="120" sortable>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">停用</el-button>
				</template>
			</el-table-column>
		</el-table>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	//import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					name: ''
				},
				contractList:[],
				// contractList: [{
                //     contractNum: 'SF20190728',
                //     partner: '顺丰集团',
                //     contractName: '顺丰2019-2020合同',
                //     versionNum: 'V-1.0',
                //     startDate: '2019-07-28',
                //     endDate: '2020-07-28',
                //     status: '1'
                // },{
                //     contractNum:'ST20190606',
                //     partner:'申通集团',
                //     contractName:'申通2019-2021合作细则',
                //     versionNum:'V-1.3',
                //     startDate:'2019-06-11',
                //     endDate:'2021-06-10',
                //     status:'0'
                // },{
                //     contractNum:'YD20191102',
                //     partner:'韵达集团',
                //     contractName:'韵达2019-2021合同',
                //     versionNum:'V-1.66',
                //     startDate:'2019-11-02',
                //     endDate:'2021-06-11',
                //     status:'0'
				// }],
                contractListLoading: false,
                //takeEffect:'<span style="color: green" v-html="生效"></span>',
			}
		},
		methods: {
			//状态显示转换
			formatSex: function (row, column) {
				return row.status == 1 ? '生效' : row.status == 0 ? '失效' : '未知';
			},
			//获取用户列表
            getContractList() {
			    this.contractListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/api/contract/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'name': this.filters.name
                    })
                }).then(({data}) => {
                    if(data && data.code ===0){
                        this.contractList = data.contractList
                    }else {
                        this.contractList = []
                    }
                    this.contractListLoading = false
                })
			},
            handleEdit:function(index,row){
                this.$router.push({
                    path:'/add_contract',
                    query:{
                        index:index,
                        row:row
                    }
                })
            },
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认停用此合同吗?', '提示', {
					type: 'warning'
				}).then(() => {
					//this.listLoading = true;
					this.status = '0';
					// //NProgress.start();
					// let para = { id: row.id };
					// removeUser(para).then((res) => {
					// 	this.listLoading = false;
					// 	//NProgress.done();
					// 	this.$message({
					// 		message: '删除成功',
					// 		type: 'success'
					// 	});
					// 	this.getUsers();
					// });
				}).catch(() => {

				});
			},
			gotoAddContract:function () {
				this.$router.replace('/add_contract')
			}
		},
		mounted() {
			// this.getUsers();
			this.contractListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/api/contract/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'name': this.filters.name
                    })
                }).then(({data}) => {
                    if(data && data.code ===0){
                        this.contractList = data.contractList
                    }else {
                        this.contractList = []
                    }
                    this.contractListLoading = false
                })
		}
	}

</script>
