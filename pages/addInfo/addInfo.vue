<template>
	<view>
		<bxform
			:service="serviceName"
			ref="bxForm"
			:pageType="type"
			:BxformType="type"
			:fields="fields"
			:moreConfig="colsV2Data && colsV2Data.more_config ? colsV2Data.more_config : null"
		></bxform>
		<bxButtons v-if="fields&&fields.length>0" :buttons="buttons" @on-button-change="onButton($event)"></bxButtons>
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
			type: 'add',
			serviceName: 'srvdaq_tkry_users_add',
			condition: [],
			defaultCondition: [],
			params: {}
		};
	},
	computed: {
		buttons: function() {
			let buttons = [];
			if (this.colsV2Data && this.colsV2Data._buttonInfo) {
				buttons = this.colsV2Data._buttonInfo;
			} else if (this.colsV2Data && this.colsV2Data._formButtons) {
				buttons = this.colsV2Data._formButtons;
			}
			let data = {};
			this.fields.forEach(item => {
				data[item['column']] = item['value'];
			});
			let fieldModel = data;
			console.log(data, 'getDetailfieldModel');

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
	onShow() {
		// #ifdef MP-WEIXIN
		this.getWxUserInfo();
		// #endif
		// this.getFieldsV2()
	},
	onLoad() {
		this.getFieldsV2()
	},
	methods: {
		getWxUserInfo() {
			// 检测是否有userInfo授权
			let userInfo = uni.getStorageSync('login_user_info');
			let self = this;
			wx.getSetting({
				success(res) {
					if (!res.authSetting['scope.userInfo']) {
						uni.showModal({
							title: '提示',
							content: '您还未授权获取用户信息,点击确定按钮跳转到授权页面',
							success(res) {
								uni.setStorageSync('hasToAuthPage', true);
								if (res.confirm) {
									self.judgeClientEnviroment();
									uni.navigateTo({
										url: '/pages/public/accountExec/accountExec'
									});
								} else {
									uni.setStorageSync('hasToAuthPage', false);
								}
							}
						});
					} else {
						wx.authorize({
							scope: 'scope.userInfo',
							success() {
								// 用户已经授权访问用户信息
								wx.getUserInfo({
									success: function(infoRes) {
										let resData = [
											{
												colName: 'openid',
												value: userInfo.user_no
											},
											{
												colName: 'nick_name',
												value: infoRes.userInfo.nickName
											},
											{
												colName: 'avatar',
												value: infoRes.userInfo.avatarUrl
											},
											{
												colName: 'gender',
												value: infoRes.userInfo.gender
											},
											{
												colName: 'country',
												value: infoRes.userInfo.country
											},
											{
												colName: 'province',
												value: infoRes.userInfo.province
											},
											{
												colName: 'city',
												value: infoRes.userInfo.city
											}
										];
										self.defaultCondition = resData;
										uni.setStorageSync('wxuserinfo', infoRes.userInfo);
										self.getWxUserInfo(infoRes.userInfo);
									}
								});
							}
						});
					}
				}
			});
		},
		getFieldsV2: async function() {
			let app = uni.getStorageSync('activeApp');
			if(!app){
				app = 'daq'
				uni.setStorageSync('activeApp',app)
			}
			let colVs = await this.getServiceV2(this.serviceName, this.type, this.type, app);
			uni.setNavigationBarTitle({
				title: colVs.service_view_name
			});
			let userInfo = uni.getStorageSync('login_user_info');
			let wxUserInfo = uni.getStorageSync('wxuserinfo');
			let backWxUserInfo = uni.getStorageSync('backWxUserInfo')
			let defaultCondition=[
				{
					colName: 'openid',
					value: userInfo.user_no
				},
				{
					colName: 'nick_name',
					value:backWxUserInfo.nickName?backWxUserInfo.nickName: wxUserInfo.nickName
				},
				{
					colName: 'avatar',
					value: backWxUserInfo.headimgurl?backWxUserInfo.headimgurl:wxUserInfo.avatarUrl
				}
			]
			this.defaultCondition = defaultCondition
			this.colsV2Data = colVs;
			switch (this.type) {
				case 'update':
					if (this.defaultCondition && Array.isArray(this.defaultCondition) && colVs._fieldInfo && Array.isArray(colVs._fieldInfo)) {
						console.log('this.defaultCondition', this.defaultCondition, colVs._fieldInfo);
						let arr = [];
						this.defaultCondition.forEach(cond => {
							colVs._fieldInfo.forEach(field => {
								if (cond.colName === field.column && field.isRequire !== true) {
									field['value'] = cond['value'];
									field['disabled'] = true;
									field['display'] = false;
									field['showExp'] = false;
								} else if (cond.colName === field.column && field.isRequire === true) {
									field['value'] = cond['value'];
								}
							});
						});
					}
					this.fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.params.defaultVal);
					break;
				case 'add':
					if (this.defaultCondition && Array.isArray(this.defaultCondition) && colVs._fieldInfo && Array.isArray(colVs._fieldInfo)) {
						console.log('this.defaultCondition', this.defaultCondition, colVs._fieldInfo);
						let arr = [];
						this.defaultCondition.forEach(cond => {
							colVs._fieldInfo.forEach(field => {
								if (cond.colName === field.column && field.isRequire !== true) {
									field['value'] = cond['value'];
									field['disabled'] = true;
									field['display'] = false;
									field['showExp'] = false;
								} else if (cond.colName === field.column && field.isRequire === true) {
									field['value'] = cond['value'];
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
			let self = this;
			console.log(e, req);
			debugger
			switch (e.button_type) {
				case 'edit':
					if (e.page_type === '详情') {
					} else {
						if (req) {
							req = [{ serviceName: e.service_name, data: [req], condition: this.condition }];
							let app = uni.getStorageSync('activeApp');
							let url = self.getServiceUrl(app, e.service_name, 'add');
							self.onRequest('update', e.service_name, req).then(res => {
								console.log('res:' + e.service_name, res);
								if (res.data.state === 'SUCCESS') {
									let resData = self.getResData(res.data.response);
									console.log('resData', resData);
									uni.showModal({
										title: '提示',
										content: '修改成功',
										confirmText: '返回',
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
					if (req) {
						let name = req.real_name;
						let id = req.picp;
						let openid = uni.getStorageSync('login_user_info').user_no;
						let backWxUserInfo = uni.getStorageSync('backWxUserInfo')
						if(backWxUserInfo){
							
						}
						uni.showModal({
							title: '提示',
							content: '基础信息只能提交一次，本次提交后将不能再更改，是否点击确定提交?',
							success(res) {
								if (res.confirm) {
									req = [{ serviceName: e.service_name, data: [req] }];
									let app = uni.getStorageSync('activeApp');
									let url = self.getServiceUrl(app, e.service_name, 'add');
									console.log(url, e);
									self.$http.post(url, req).then(res => {
										console.log(url, res.data);
										if (res.data.state === 'SUCCESS') {
											console.log('res.data', res.data.response[0].response.effect_data[0]);
											uni.setStorageSync('basics_info', res.data?.response[0]?.response?.effect_data[0]);
											uni.showModal({
												title: '提示',
												content: '登记成功',
												showCancel: false,
												success(res) {
													if (res.confirm) {
														// self.wxLogin();
														uni.navigateBack({
															
														})
													}
												}
											});
										}
									});
								} else if (res.cancel) {
									//不提交
								}
							}
						});
					}
					break;
				case 'reset':
					let res = self.$refs.bxForm.onReset();
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
						const url = self.getServiceUrl(e.application, e.operate_service, 'operate');
						const req = [
							{
								data: [e.requestData],
								serviceName: e.operate_service,
								srvApp: e.application
							}
						];
						let res = await self.$http.post(url, req);
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
					break;
				default:
					uni.showToast({
						title: e.button_name
					});
					break;
			}
		}
	}
};
</script>

<style></style>
