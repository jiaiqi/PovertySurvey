<template>
	<view>
		<bxform
			:service="serviceName"
			ref="bxForm"
			:pageType="type"
			:defaultCondition="defaultCondition"
			:BxformType="type"
			:fields="fields"
			@changeFieldModel="changeFieldModel"
			:moreConfig="colsV2Data && colsV2Data.more_config ? colsV2Data.more_config : null"
		></bxform>
		<bxButtons :buttons="buttons" @on-button-change="onButton($event)" v-if="buttons && buttons.length > 0"></bxButtons>
		<!-- <button class="bg-green cu-btn lg">查看列表</button> -->
		<view class="sublist-content">
			<!-- <view class="sublist-content" v-if="type === 'detail'&&hasChildService"> -->
			<view class="sublist-box" v-if="showSublist">
				<view class="child-service bg-blue" v-for="item in childService" :key="item.service_name" @click="toChildList(item)">
					{{ item.foreign_key.section_name }}({{ item.childData.data.length }})
				</view>
			</view>
			<button class="cu-btn  bg-blue margin-tb-sm" v-if="!showSublist&&hasChildService" @click="showSublist = !showSublist">
				展开子表
				<text class="lg text-white cuIcon-down margin-left-xs"></text>
			</button>
			<button class="cu-btn  bg-blue margin-tb-sm" v-if="showSublist" @click="showSublist = !showSublist">
				收起子表
				<text class="lg text-white cuIcon-top margin-left-xs"></text>
			</button>
		</view>
	</view>
</template>

<script>
import bxform from '@/components/bx-form/bx-form.vue';
import bxButtons from '@/components/bx-buttons/bx-buttons.vue';
export default {
	components: { bxform, bxButtons },
	data() {
		return {
			fields: [],
			loadedType: 'srvV2',
			colsV2Data: null,
			type: '',
			serviceName: '',
			condition: [],
			defaultCondition: [],
			params: {},
			formData: {},
			childService: [], // 子表
			hasChildService: false, //是否拥有子表
			showSublist: false //显示子表
		};
	},
	computed: {
		buttons: function() {
			let buttons = [];
			if (this.colsV2Data && this.colsV2Data._buttonInfo) {
				buttons = this.colsV2Data._buttonInfo;
				// return this.colsV2Data._buttonInfo;
			} else if (this.colsV2Data && this.colsV2Data._formButtons) {
				buttons = this.colsV2Data._formButtons;
				// return this.colsV2Data._formButtons;
			}
			let data = {};
			if (Array.isArray(this.fields)) {
				this.fields.forEach(item => {
					data[item['column']] = item['value'];
				});
				let fieldModel = data;
				console.log(data, 'getDetailfieldModel');
			}

			buttons.forEach(btn => {
				if (btn.disp_exps) {
					btn['display'] = eval(btn.disp_exps);
				}
				if (btn.operate_params) {
					let fieldData = btn.operate_params['data'];
					if (fieldData && Array.isArray(fieldData) && fieldData.length > 0) {
						fieldData = fieldData[0];
						let newData = {};
						Object.keys(fieldData).forEach(data_item => {
							let item = fieldData[data_item];
							if (item.value_type && item.value_type === 'rowData') {
								newData[data_item] = fieldModel[item.value_key];
							}
						});
						btn['requestData'] = newData;
						btn['requestCondition'] = this.condition;
					}
				}
			});
			return buttons;
		}
	},
	created() {
		// #ifdef H5
		const destApp = this.$route.query.destApp;
		if (destApp) {
			uni.setStorageSync('activeApp', destApp);
		}
		// #endif
	},
	onLoad(option) {
		let query = JSON.parse(decodeURIComponent(option.query ? option.query : option.params ? option.params : '{}'));
		console.log('---------', option);
		const destApp = query.destApp;
		if (destApp) {
			uni.setStorageSync('activeApp', destApp);
		}
		if (option.serviceName) {
			query = option;
		}
		if (query.params) {
			this.params = JSON.parse(query.params);
		}
		if (query.formData) {
			try {
				this.formData = JSON.parse(decodeURIComponent(query.formData));
				console.log('formData-formPage', this.formData);
			} catch (e) {
				console.log('formData', e);
				//TODO handle the exception
			}
		}
		if (option.params) {
			this.params = query;
		}
		if (query.cond || query.condition) {
			let cond = '';
			if (typeof query.cond === 'string' && query.cond) {
				cond = JSON.parse(query.cond);
			}
			if (typeof query.condition === 'object') {
				cond = query.condition;
			}
		}
		if (option.hasOwnProperty('loadedType')) {
			this.loadedType = option.loadedType;
		} else if (option.hasOwnProperty('params')) {
			this.serviceName = this.params.serviceName;
			this.type = this.params.type;
			this.condition = this.params.condition;
			this.defaultVal = this.params.defaultVal;
			let cond = [];
			if (this.params.cond && Array.isArray(this.params.cond)) {
				cond = this.params.cond;
			}
			this.getFieldsV2(cond);
		} else if (query.serviceName && query.type) {
			this.serviceName = query.serviceName;
			this.type = query.type;
			if (option.hasOwnProperty('cond')) {
				try {
					this.condition = JSON.parse(decodeURIComponent(option.cond));
				} catch (e) {
					//TODO handle the exception
					console.log(e);
				}
			}
			this.getFieldsV2();
		} else {
			uni.showToast({
				title: '加载错误',
				icon: 'none'
			});
		}
	},

	methods: {
		toChildList(e) {
			uni.navigateTo({
				url: '/pages/list/list?serviceName=' + e.service_name
			});
		},
		changeFieldModel(e) {
			if (e) {
				this.formData = this.deepClone(e);
			}
		},
		async selectList(item) {
			let app = uni.getStorageSync('activeApp');
			let url = this.getServiceUrl(app, item.service_name, 'select');
			let formData = this.params.defaultVal;
			if (item.foreign_key && item.foreign_key.referenced_column_name && formData[item.foreign_key.referenced_column_name]) {
				let req = {
					serviceName: item.service_name,
					colNames: ['*'],
					condition: [{ colName: item.foreign_key.column_name, ruleType: 'eq', value: formData[item.foreign_key.referenced_column_name] }],
					relation_condition: {},
					page: { pageNo: 1, rownumber: 5 },
					order: []
				};
				let res = await this.$http.post(url, req);
				if (res.data.state === 'SUCCESS') {
					return { data: res.data.data, page: res.data.page };
				} else {
					return res.data.state;
				}
			} else {
				return false;
			}
		},
		getFieldsV2: async function(condition) {
			let app = uni.getStorageSync('activeApp');
			let colVs = await this.getServiceV2(this.serviceName, this.type, this.type, app);
			if (colVs.child_service && Array.isArray(colVs.child_service) && colVs.child_service.length > 0) {
				// 有子表
				this.hasChildService = true;
				this.childService = colVs.child_service;
				this.childService.forEach((item, index) => {
					this.selectList(item).then(res => {
						item.childData = res;
						this.$set(this.childService, index, item);
					});
				});
			}
			if (!this.navigationBarTitle) {
				uni.setNavigationBarTitle({
					title: colVs.service_view_name
				});
			}
			this.colsV2Data = colVs;
			let self = this;
			switch (this.type) {
				case 'update':
				//
				//       this.fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.params.defaultVal);
				//       break;
				case 'add':
					console.log(this.formData, 'this.formData');
					if (Object.values(this.formData).length > 0) {
						Object.keys(this.formData).forEach(item => {
							colVs._fieldInfo.forEach(field => {
								if (item === field.column) {
									field['value'] = this.formData[item];
								}
							});
						});
					}
					if (this.defaultCondition && Array.isArray(this.defaultCondition) && colVs._fieldInfo && Array.isArray(colVs._fieldInfo)) {
						console.log('this.defaultCondition', this.defaultCondition, colVs._fieldInfo);
						let arr = [];
						this.defaultCondition.forEach(cond => {
							colVs._fieldInfo.forEach(field => {
								if (cond.colName === field.column) {
									field['value'] = cond['value'];
									field['disabled'] = true;
								}
							});
						});
					}
					if (condition && Array.isArray(condition)) {
						condition.forEach(item => {
							colVs._fieldInfo.forEach(field => {
								if (field.column === item.colName) {
									field.condition = item.value;
								}
								if (typeof item.value !== 'string' && Array.isArray(item.value)) {
									item.value.forEach(cond => {
										if (cond.colName === cond.value && field.column === cond.value) {
											field['display'] = false;
										}
									});
								}
								if (this.params.defaultVal) {
									// 赋默认值
								}
							});
						});
					}
					this.fields = colVs._fieldInfo;
					break;
				case 'detail':
					this.fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.params.defaultVal);
					break;
				default:
					break;
			}
		},
		async onButton(e) {
			let req = this.$refs.bxForm.getFieldModel();
			console.log(this.condition);
			if ((e.service_name == 'srvzhxq_syrk_wuye_add' || e.service_name == 'srvzhxq_syrk_add') && (!req.proc_status || req.proc_status != '完成') && req) {
				req.proc_status = '完成';
			}
			console.log(e, req);
			switch (e.button_type) {
				case 'edit':
					if (e.page_type === '详情') {
					} else {
						if (req) {
							req = [{ serviceName: e.service_name, data: [req], condition: this.condition }];
							let app = uni.getStorageSync('activeApp');
							let url = this.getServiceUrl(app, e.service_name, 'add');
							this.onRequest('update', e.service_name, req).then(res => {
								console.log('res:' + e.service_name, res);
								if (res.data.state === 'SUCCESS') {
									let resData = this.getResData(res.data.response);
									console.log('resData', resData);
									uni.showModal({
										title: '提示',
										content: '修改成功',
										showCancel: false,
										success(res) {
											if (res.confirm) {
												uni.navigateBack();
											}
										}
									});
								}
							});
						}
					}
					break;
				case 'submit':
					console.log('addServiceName:', e.service_name);
					if (req) {
						req = [{ serviceName: e.service_name, data: [req] }];
						let app = uni.getStorageSync('activeApp');
						let url = this.getServiceUrl(app, e.service_name, 'add');
						console.log(url, e);
						let res = await this.$http.post(url, req);
						console.log(url, res.data);
						if (res.data.state === 'SUCCESS') {
							uni.showModal({
								title: '提示',
								content: '添加成功',
								showCancel: false,
								success(res) {
									if (res.confirm) {
										uni.navigateBack();
									}
								}
							});
						}
					}
					break;
				case 'reset':
					let res = this.$refs.bxForm.onReset();
					if (res) {
						uni.showToast({
							title: '已重置'
						});
					} else {
						uni.showToast({
							title: '无效操作'
						});
					}
					break;
				case 'customize':
					if (e.application && e.operate_service) {
						const url = this.getServiceUrl(e.application, e.operate_service, 'operate');
						const req = [
							{
								data: [e.requestData],
								serviceName: e.operate_service,
								srvApp: e.application
							}
						];
						let res = await this.$http.post(url, req);
						if (res.data.state === 'SUCCESS') {
							uni.showModal({
								title: '提示',
								content: e.tip_msg ? e.tip_msg : res.data.resultMessage,
								showCancel: false,
								success(res) {
									if (res.confirm) {
										setTimeout(() => {
											uni.navigateBack();
										}, 2000);
									}
								}
							});
						} else {
							uni.showToast({
								title: res.data.resultMessage,
								mask: false,
								icon: 'none'
							});
						}
					}
					// const url = this.getServiceUrl(e.application, e.operate_service, 'operate');
					// const req = [
					//   {
					//     data: [e.requestData],
					//     serviceName: e.operate_service,
					//     srvApp: e.application
					//   }
					// ];
					// let res = await this.$http.post(url, req);
					break;
				default:
					uni.showToast({
						title: e.button_name
					});
					break;
			}
		},
		async getUserInfo() {
			let user_no = uni.getStorageSync('basics_info').picp;
			let urls = this.getServiceUrl('zhxq', 'srvzhxq_syrk_select', 'select');
			let reqs = {
				serviceName: 'srvzhxq_syrk_select',
				colNames: ['*'],
				condition: [
					{
						colName: 'gmsfhm',
						ruleType: 'eq',
						value: user_no
					},
					{
						colName: 'proc_status',
						ruleType: 'eq',
						value: '完成'
					},
					{
						colName: 'status',
						ruleType: 'eq',
						value: '有效'
					}
				]
				// order: [{ colName: 'seq', orderType: 'asc' }] ,
			};
			let ress = await this.$http.post(urls, reqs);
			console.log('------------', ress);
			return ress.data.data[0];
		}
	}
};
</script>

<style lang="scss" scoped>
.sublist-content {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	margin: 50rpx 20rpx 0;
	.sublist-box {
		width: 100%;
		display: flex;
		border: 1px dashed #efefef;
		justify-content: space-around;
		padding: 30rpx 20rpx;
		.child-service {
			width: 50%;
			height: 80rpx;
			display: flex;
			justify-content: center;
			align-items: center;
			border-radius: 10rpx;
			margin: 10rpx;
			text-align: center;
		}
	}
}
</style>
