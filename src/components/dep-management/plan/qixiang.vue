<template>
	<div class="school">
		<div class=title>
			<span>位置： </span>
			<Breadcrumb separator=">">
				<BreadcrumbItem to="/home/depManagement/qixiang">部门管理</BreadcrumbItem>
				<BreadcrumbItem>气象站数据维护</BreadcrumbItem>
			</Breadcrumb>
		</div>
		<!--搜索-->
		<Form ref="formInline" inline :label-width="80">
			<Form-item label="名称">
				<Input v-model="formInline.stn_name" :maxlength="16" placeholder="请输入"></Input>
			</Form-item>
			<Form-item label="编号">
				<Input v-model="formInline.stn_no" :maxlength="16" placeholder="请输入"></Input>
			</Form-item>
			<Form-item label="主管部门">
				<Input v-model="formInline.town" :maxlength="16" placeholder="请输入"></Input>
			</Form-item>
			<Button type="ghost" shape="circle" icon="ios-search" class="ziti" @click="submitsearch(1)"></Button>
			<Button class="addBtn pull-right" @click="add"><Icon type="plus-round" ></Icon> 新增</Button>
		</Form>
		<!--表格-->
		<div class="table">
			<Table :loading="loading" border stripe size="small" highlight-row :columns="tableThead" :data="tableTbody"></Table>
			<Page class="pull-right" :total="total" :current="current" size="small" show-total @on-change="changePage"></Page>
		</div>
		<template slot="content">
			<!--确认删除  -->
			<Modal v-model="modal2" width="360">
				<p slot="header" style="color:#f60;text-align:center">
					<Icon type="information-circled"></Icon>
					<span>删除</span>
				</p>
				<div style="text-align:center">
					<p>确认删除？</p>
				</div>
				<div slot="footer">
					<Button type="error" size="large" long :loading="modal_loading" @click="dels">确认</Button>
				</div>
			</Modal>
		</template>
	</div>
</template>

<script>
	import { qixiangThead } from 'common/js/table' ;
	import { getUserList,getaddlist,geteditlist,getdellist } from 'api/qixiang';
	import { ERR_OK } from 'api/config';
//	import { mapMutations } from 'vuex';
	import { successNotice, errorNotice, getLocalStorage } from 'common/js/dom';
	import { cloneObj, likeStrSearch } from 'common/js/util';
	export default {
		data() {
			return {
				loading:false,
				tableThead:qixiangThead(this),
				tableTbody:[],
				total:1,
				current:1,
				modal2: false,
				modal_loading: false,
				formInline:{
					stn_name:'',
					stn_no:'',
					town:''
				}
			}
		},
		methods: {
			//页码
			changePage(page) {
				this.page = page;
				this.submitsearch(page);
//				this._getuserList(page);
			},
			//新增
			add() {
				this.formItem = {
					city:'',
					district:'',
					lat:'',
					lng:'',
					stn_name:'',
					stn_no:'',
					stn_tpye:'',
					town:''
				}
				getLocalStorage('id', 'false')
				this.$router.push('home/depManagement/qixiang/qixiangweihu');
			},
			//搜索
			submitsearch(page) {
				let search = {}
				if(this.formInline.stn_name) {
					search.stn_name = likeStrSearch(this.formInline.stn_name)
				}
				if(this.formInline.stn_no) {
					search.stn_no = likeStrSearch(this.formInline.stn_no)
				}
				if(this.formInline.town) {
					search.town = likeStrSearch(this.formInline.town)
				}
				this._getuserList(page, search);
			},
			//列表
			_getuserList(page,search) {
				this.loading = true
				getUserList(page,search).then(res =>{
					if(res.code === ERR_OK) {
						this.loading = false
						this.tableTbody = res.result.result;
						this.total = res.result.totalSize;
					}
				})
			},
			//点击编辑
			edit(dataObj) {
				getLocalStorage("formItemd", dataObj);
				getLocalStorage("id", dataObj._id);
//				this.SET_SHUIKU(dataObj);
				this.$router.push('home/depManagement/qixiang/qixiangweihu')
			},
			//删除
			del(row) {
				this.rowIng = cloneObj(row)
				this.modal2 = true;
			},
			//删除成功
			dels() {
				let data = {
					_ids: this.rowIng._id
				}
				getdellist(this.rowIng._id).then(res => {
					console.log(res)
					if(res.code === ERR_OK) {
						this._getuserList();
						this.$Notice.success({
							title: '删除成功'
						});
					}
					this.modal2 = false;
				})
				
			},
//			...mapMutations([
//				'SET_SHUIKU'
//			])
		},
		mounted() {
			this._getuserList();
			this.rowIng = {};
		}
	}
</script>

<style scoped lang="scss">
	.title {
		position: absolute;
		top: 0px;
		left: 0px;
		display: flex;
		flex-direction: row;
		width: 100%;
		 background: rgb(237, 246, 250);  
		height: 50px;
		line-height: 50px;
		padding-left: 10px;
		>span {
			font-size: 16px;
			font-weight: 600;
		}
		
	}
	.addBtn {
			background-color: #008cee !important;
			color: #fff;
			margin: 0px;
			height: 40px;
		}
	.ziti {
		display: inline-block;
		background: #18bdff;
		font-size: 20px;
		color: #FFFFFF;
		border-radius: 0;
	}
</style>