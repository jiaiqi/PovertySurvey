<template>
	<view class="wrapper">
		<view class="topContent">
			<view class="wrapheadr">
				<view class="title bg-white">{{ title }}</view>
				<view class="top bg-white">
					<view class="">
						<view class="titlecolor">{{ pageInfo.total }}</view>
						<view class="text">特困人数</view>
					</view>
					<view class="">
						<view class="titlecolor">{{ nglum }}</view>
						<view class="text">管理人员</view>
					</view>
				</view>
			</view>
			<view class="middle">
				<button class="cu-btn block bg-white " @click="mine()">
					<text class="lg cuIcon-comment text"></text>
					纪实感悟
				</button>
				<button class="cu-btn block bg-white " @click="mine('mine')">
					<text class="lg cuIcon-friendfamous text"></text>
					我的
				</button>
			</view>
		</view>
		<s-pull-scroll ref="pullScroll" :back-top="true" :pullDown="pullDown" :top="scrollTop" :pullUp="loadData">
			<view class="wrapCont">
				<view class="Jlsuccess bg-white" style="" v-for="(item, index) in list" :key="index">
					<image class="heaserImg" :src="item.photo ? item.photo : headerimg" mode=""></image>
					<view class="infrom" style="">
						<view class="">
							<text class="nametitle">{{ item.full_name }}</text>
							<text class="tran">***</text>
							<text>{{ item._sex_disp }}</text>
						</view>
						<view class="Inumber">身份证号码：{{ item.id_card ? item.id_card : '暂无' }}</view>
					</view>
				</view>
			</view>
		</s-pull-scroll>
		<view class="public-button-box"><view class="add-button" @click="adddesc()"></view></view>
	</view>
</template>

<script>
import sPullScroll from '@/components/s-pull-scroll/index.vue';
export default {
	components: {
		sPullScroll
	},
	data() {
		return {
			list: [],
			title: '',
			headerimg: 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3491929439,4106440758&fm=26&gp=0.jpg',
			pageInfo: {
				pageNo: 1,
				rownumber: 10,
				total: 0
			},
			nglum: 0,
			scrollTop:''
		};
	},
	onLoad(option) {
		let item = JSON.parse(decodeURIComponent(option.itemDetail));
		this.title = item.data.path_name.substring(1, item.data.path_name.length - 1);
		let titleArray = item.data.path_name.split('/').filter(item => item && item);
		if (titleArray.length > 0) {
			uni.setNavigationBarTitle({
				title: titleArray[0]
			});
		}
		// #ifdef MP-WEIXIN
		this.scrollTop = '440'
		// #endif
		// #ifdef H5
		this.scrollTop = '350'
		// #endif
		this.refresh();
		this.getNum(this.title);
	},
	methods: {
		adddesc() {
			uni.navigateTo({
				url: '/pages/formPage/formPage?serviceName=' + 'srvdaq_tkry_visit_record_select' + '&type=add'
			});
		},
		mine(val) {
			let con = [
				{
					colName: 'create_user',
					ruleType: 'eq',
					value: uni.getStorageSync('login_user_info').user_no
				}
			];
			uni.navigateTo({
				url: '/pages/list/list?serviceName=srvdaq_tkry_visit_record_select' + `&cond=${val == 'mine' ? decodeURIComponent(JSON.stringify(con)) : ''}`
			});
		},
		refresh() {
			this.$nextTick(() => {
				this.$refs.pullScroll.refresh();
			});
		},
		loadData(pullScroll) {
			this.pageInfo.pageNo++;
			setTimeout(() => {
				this.getInfoList(this.title);
			}, 1000);
		},
		pullDown(pullScroll) {
			this.pageInfo.pageNo = 1;
			setTimeout(() => {
				this.getInfoList(this.title);
			}, 500);
		},
		async getNum(val) {
			let optionType = 'select';
			let srv = 'srvdaq_tkry_users_select';
			let app = 'daq';
			let req = {
				serviceName: 'srvdaq_tkry_users_select',
				colNames: ['*'],
				condition: [],
				order: []
			};
			let con = {
				colName: 'area_id',
				ruleType: 'like',
				value: val
			};
			if (val) {
				req.condition.push(con);
			}
			let res = await this.onRequest(optionType, srv, req, app);
			if (res.data.data.length > 0) {
				this.nglum = res.data.data.length;
			}
		},
		async getInfoList(val) {
			let optionType = 'select';
			let srv = 'srvdaq_tkry_people_select';
			let app = 'daq';
			let req = {
				serviceName: 'srvdaq_tkry_people_select',
				colNames: ['*'],
				condition: [],
				order: [],
				page: {
					pageNo: this.pageInfo.pageNo,
					rownumber: this.pageInfo.rownumber
				}
			};
			let con = {
				colName: 'area_id',
				ruleType: 'like',
				value: val
			};
			if (val) {
				req.condition.push(con);
			}
			let res = await this.onRequest(optionType, srv, req, app);
			if (res.data && res.data.data && res.data.data.length > 0) {
				let itemData = res.data.data;
				this.pageInfo.total = res.data.page.total;
				itemData.forEach((item, index, itemData) => {
					this.getFilePath(item.photo).then(a => {
						if (a && a[0] && a[0].fileurl) {
							let urls = this.$api.downloadImg + a[0].fileurl + '&bx_auth_ticket=' + uni.getStorageSync('bx_auth_ticket');
							item.photo = urls;
						}
					});
				});
				if (this.pageInfo.pageNo == 1) {
					this.list = itemData;
				} else {
					this.list = this.list.concat(itemData);
				}
				let page = this.pageInfo;
				this.$nextTick(() => {
					if (page.rownumber * page.pageNo >= page.total) {
						this.$refs.pullScroll.finish();
					} else {
						this.$refs.pullScroll.success();
					}
				});
			} else {
				this.list = [];
				this.$refs.pullScroll.finish();
			}
		}
	}
};
</script>

<style lang="less" scoped>
.wrapheadr {
	margin: 0 20rpx 20rpx;
	display: flex;
	flex-direction: column;
	text-align: center;
	.title {
		padding: 30rpx;
		max-height: 100rpx;
		overflow: hidden;
		border-radius: 10rpx;
		margin: 20rpx 0;
	}
	.top {
		padding: 30rpx;
		border-radius: 10rpx;
		display: flex;
		// background-color: #98a79640;
		justify-content: space-between;
	}
	.titlecolor {
		font-size: 35rpx;
		line-height: 30px;
	}
	.text {
		font-size: 25rpx;
		line-height: 30px;
	}
}
.Jlsuccess {
	display: flex;
	margin: 20rpx;
	padding: 26rpx;
	border-radius: 8rpx;
	.infrom {
		height: 80rpx;
		margin-left: 18rpx;
		.nametitle {
			font-size: 30rpx;
			font-weight: bold;
		}
		.tigb {
			font-size: 24rpx;
			padding: 4rpx 10rpx;
			text-align: center;
			border-radius: 7rpx;
		}
		.jointime {
			display: flex;
			justify-content: space-between;
		}
	}
	.Inumber {
		font-size: 26rpx;
		color: #959595;
		line-height: 50rpx;
	}
	.heaserImg {
		display: inline-block !important;
		width: 80rpx !important;
		height: 80rpx !important;
		border-radius: 50% !important;
	}
}
.tran {
	opacity: 0;
}
.middle {
	display: flex;
	justify-content: space-between;
	font-size: 32rpx;
	margin: 20rpx;
	.block {
		border-radius: 10rpx;
		width: calc(50% - 10rpx);
		height: 100rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		.text {
			margin-right: 8rpx;
			font-size: 36rpx;
		}
	}
}
.add-button {
	position: fixed;
	bottom: 100rpx;
	left: calc(50% - 50rpx);
	width: 100rpx;
	height: 100rpx;
	border-radius: 50%;
	background-color: #007aff;
	z-index: 100;
	&::before {
		position: absolute;
		width: 8rpx;
		height: 60rpx;
		left: calc(50% - 4rpx);
		top: 20rpx;
		content: '';
		background-color: #fff;
	}
	&::after {
		position: absolute;
		width: 60rpx;
		height: 8rpx;
		content: '';
		top: calc(50% - 4rpx);
		left: 20rpx;
		background-color: #fff;
	}
	&:active {
		transform: rotate(45deg);
		transition: all 0.2s;
	}
}
.wrapper {
	height: 100vh;
	background-color: #f1f1f1;
}
.topContent {
	width: 100%;
	// position: absolute;
	// top: 50px;
}
</style>
