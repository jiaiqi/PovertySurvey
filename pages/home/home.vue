<template>
	<view class="page-wrap">
		<view
			v-for="(pageItem, itemIndex) in pageItemList"
			:key="itemIndex"
			:class="{ 'swiper-view': pageItem.div_type === 'carousel', 'menu-view': pageItem.div_type === 'buttons', 'tab-list-view': pageItem.div_type === 'tablist' }"
		>
			<SwiperCarousel :swiperList="swiperList" v-if="pageItem.div_type === 'carousel'" @clickSwiper="clickSwiper" :imgCol="'picUrl'"></SwiperCarousel>
			<SwiperMenu
				v-if="pageItem.div_type === 'buttons'"
				:menuList="pageItem.data"
				:menuLabel="pageItem.item_label"
				:gridWidth="pageItem.grid_width"
				@clickMenu="clickMenu"
			></SwiperMenu>
			<TabList
				@clickListItem="clickListItem"
				@showMore="showMore($event, pageItem)"
				v-if="pageItem.div_type === 'tablist'"
				:pageItem="pageItem"
				:srvApp="tabListConfig['srvApp']"
				:contentService="tabListConfig['contentService']"
				:cateService="tabListConfig['cateService']"
				:contentTemplate="tabListConfig['contentTemplate']"
			></TabList>
		</view>
	</view>
</template>

<script>
import TabList from '@/components/bx-tablist/bx-tablist.vue';
import SwiperCarousel from '@/components/bx-swiper/bx-swiper.vue';
import SwiperMenu from '@/components/bx-swiper-menu/bx-swiper-menu.vue';
export default {
	components: { TabList, SwiperCarousel, SwiperMenu },
	data() {
		return {
			scrollTop: 0,
			dotStyle: true,
			pageItemList: [], //页面项列表，控制页面显示内容
			menuList: [], //按钮列表
			newsList: [], //新闻列表
			swiperList: [], //轮播图列表
			categoryList: [], //分类列表
			webNo: '',
			webNo2: '',
			isOwner: '',
			tabListConfig: {
				srvApp: 'daq',
				contentService: 'srvdaq_cms_content_select',
				cateService: 'srvdaq_page_item_tablist_select',
				contentTemplate: {
					imgCol: 'icon_image',
					titleCol: 'title',
					dateCol: 'create_time'
				}
			},
			collectorInfo: null
		};
	},
	onShow() {
		this.selectCollectorInfo().then(res => {
			console.log(res);
			if (res) {
				this.collectorInfo = res;
			} else {
				// 没有登记采集人员信息
				// uni.navigateTo({
				// 	url:'/pages/addInfo/addInfo'
				// })
			}
		});
	},
	onLoad(option) {
		const bxAuthTicket = uni.getStorageSync('bx_auth_ticket');
		if (!bxAuthTicket) {
			this.throttle(this.wxLogin(), 3000);
		}
		uni.setStorageSync('activeApp', 'daq');
		if (option.website_no || option.no) {
			this.webNo = option.website_no ? option.website_no : option.no;
			if (option.destApp) {
				uni.setStorageSync('activeApp', option.destApp);
			}
		} else {
			this.webNo = 'WS2020080315200011';
		}
		if (this.webNo) {
			console.log('-----onLoad-homePage-------', this.webNo);
			this.getPageItemList();
		}
	},

	computed: {
		tabNewsList() {
			//根据栏目分类的新闻列表
			let NewsList = [];
			let tabNewsList = [];
			this.categoryList.forEach((cate, index) => {
				tabNewsList[index] = [];
				this.newsList.forEach(listItem => {
					if (cate.no === listItem.cate_no) {
						tabNewsList[index] = tabNewsList[index].concat([listItem]);
						NewsList[index] = {};
						NewsList[index]['data'] = tabNewsList[index];
						NewsList[index]['name'] = listItem.cate_name;
					}
				});
			});
			return NewsList;
		}
	},
	methods: {
		async selectCollectorInfo() {
			let user_no = uni.getStorageSync('login_user_info').user_no;
			let url = this.getServiceUrl('daq', 'srvdaq_tkry_users_select', 'select');
			let req = {
				serviceName: 'srvdaq_tkry_users_select',
				colNames: ['*'],
				condition: [{ colName: 'openid', ruleType: 'eq', value: user_no }],
				page: { pageNo: 1, rownumber: 10 },
				order: []
			};
			if (user_no) {
				let res = await this.$http.post(url, req);
				let data = false;
				if (res.data.state === 'SUCCESS' && res.data.data && Array.isArray(res.data.data) && res.data.data.length > 0) {
					data = res.data.data[0];
				}
				return data;
			}
		},
		getPageInfo(website_no, page_no) {
			if (page_no && website_no) {
				this.webNo = website_no;
				this.getPageItemList();
			}
		},
		showMore(e, pageitem) {
			console.log('点击了更多按钮', e, pageitem);
		},
		clickSwiper(e) {
			// 点击了轮播图
			console.log('点击了轮播图,', e);
		},
		async clickMenu(e) {
			// 菜单按钮点击事件
			console.log('点击了按钮：', e);
			let collectorInfo = await this.selectCollectorInfo();
			if (!collectorInfo) {
				uni.showModal({
					title: '提示',
					content: '当前登录用户未登记信息，是否跳转到信息登记页面',
					success(res) {
						if (res.confirm) {
							uni.navigateTo({
								url: '/pages/addInfo/addInfo'
							});
						}
					}
				});
				return;
			}
			if (e.dest_menu_no === '采集统计') {
				if (collectorInfo.is_admin !== '是') {
					uni.showToast({
						title: '没有访问权限',
						icon: 'none'
					});
					return;
				}
			}
			if (e.dest_menu_no === '特困人员') {
				if (collectorInfo.status !== '有效') {
					uni.showModal({
						title: '提示',
						content: '你的信息还未被认证有效,请联系管理人员进行认证',
						showCancel: false,
						confirmText: '知道了'
					});
					return;
				}
			}
			if (e.type && e.type === 'more') {
				// 点击了更多按钮
				console.log('点击了更多按钮');
			} else if (e.dest_page) {
				uni.setStorageSync('activeApp', e.dest_app);
				uni.navigateTo({
					url: e.dest_page,
					fail(res) {
						if (res.errMsg) {
							// #ifdef H5
							window.location.href = e.dest_page;
							// #endif
							// #ifdef MP
							uni.navigateTo({
								url: '/pages/public/webContainer/webContainer?webSrc=' + e.dest_page
							});
							// #endif
						}
					}
				});
			}
		},
		clickListItem(e) {
			// 列表点击事件
			console.log('点击了列表项：', e);
			let query = {
				serviceName: 'srvdaq_cms_content_select',
				content_no: e.content_no,
				destApp: 'daq'
			};
			uni.navigateTo({
				url: '/pages/article/article?query=' + encodeURIComponent(JSON.stringify(query))
			});
		},
		async getPageItemList() {
			//获取页面项列表
			console.log('------------进入请求页面列表');
			uni.showLoading({
				mask: true
			});
			let url = this.getServiceUrl('daq', 'srvdaq_website_page_item_select', 'select');
			let req = {
				serviceName: 'srvdaq_website_page_item_select',
				colNames: ['*'],
				order: [{ colName: 'seq', orderType: 'asc' }],
				condition: [{ colName: 'website_no', ruleType: 'eq', value: this.webNo }]
			};
			let res = await this.$http.post(url, req);
			uni.hideLoading();
			if (res.data.state === 'SUCCESS') {
				this.pageItemList = res.data.data;
				res.data.data.map((item, index) => {
					switch (item.div_type) {
						case 'buttons':
							item['componentsType'] = 'SwiperMenu';
							break;
						case 'carousel':
							item['componentsType'] = 'SwiperCarousel';
							break;
						case 'tablist':
							item['componentsType'] = 'TabList';
							break;
							return;
					}
					this.getPageItem(item).then(data => {
						item['data'] = data;
						this.$set(this.pageItemList, index, item);
					});
					return item;
				});
				return res.data.data;
			}
		},
		async getPageItem(item = {}) {
			// 获取页面项
			uni.showLoading({
				mask: true
			});
			let serviceName = '';
			switch (item.div_type) {
				case 'buttons':
					serviceName = 'srvdaq_page_item_buttons_select';
					break;
				case 'carousel':
					serviceName = 'srvdaq_page_item_carousel_select';
					break;
				case 'tablist':
					serviceName = 'srvdaq_cms_content_select';
					return;
					break;
			}
			if (item.div_type === 'tablist' && serviceName) {
				let urls = this.getServiceUrl('daq', serviceName, 'select');
				let reqs = {
					serviceName: serviceName,
					colNames: ['*'],
					group: [
						{
							colName: 'no',
							type: 'by'
						}
					]
					// order: [{ colName: 'seq', orderType: 'asc' }] ,
				};
				let ress = await this.$http.post(urls, reqs);
				if (ress.data.state === 'SUCCESS') {
					this.categoryList = ress.data.data;
				}
			}
			if (serviceName) {
				let url = this.getServiceUrl('daq', serviceName, 'select');
				let req = {
					serviceName: serviceName,
					colNames: ['*'],
					order: [{ colName: 'seq', orderType: 'asc' }],
					condition: [{ colName: 'item_no', ruleType: 'in', value: item.item_no }]
				};
				let res = await this.$http.post(url, req);
				uni.hideLoading();
				if (res.data.state === 'SUCCESS') {
					let itemList = res.data.data;
					itemList.forEach((pageitem, index) => {
						switch (item.div_type) {
							case 'buttons':
								let itemLists = [];
								if (itemList.length <= 8) {
									itemLists = [itemList];
								} else if (itemList.length > 8 && itemList.length <= 16) {
									itemLists = [itemList.slice(0, 8), itemList.slice(8)];
								} else if (itemList.length > 16) {
									itemLists = [itemList.slice(0, 8), itemList.slice(8, 16)];
								}
								this.menuList = itemLists;
								break;
							case 'carousel':
								pageitem['picUrl'] = this.$api.serverURL + '/file/download?fileNo=' + pageitem.carousel_image;
								this.$set(itemList, index, pageitem);
								this.swiperList = itemList;
								break;
							case 'tablist':
								// this.newsList = itemList;
								// this.getPictureUrl(pageitem.icon_image).then(url => {
								//   pageitem['picUrl'] = url;
								//   this.$set(itemList, index, pageitem);
								//   this.newsList = itemList;
								// });
								// this.getCategoryList(pageitem.no).then(data => {
								//   pageitem['cate_name'] = data.cate_name;
								//   pageitem['cate_no'] = data.no;
								//   this.$set(itemList, index, pageitem);
								//   this.newsList = itemList;
								// });
								break;
						}
						itemList = res.data.data;
					});
					return itemList;
				}
			}
		},
		async getCategoryList(no) {
			//根据分类编号查询对应分类
			uni.showLoading({
				mask: true
			});
			const url = this.getServiceUrl('daq', 'srvdaq_cms_category_select', 'select');
			const req = { serviceName: 'srvdaq_cms_category_select', queryMethod: 'select', colNames: ['*'], condition: [{ colName: 'no', value: no, ruleType: 'eq' }] };
			let res = await this.$http.post(url, req);
			uni.hideLoading();
			if (res.data.state === 'SUCCESS' && res.data.data.length > 0) {
				let data = res.data.data[0];
				return data;
			}
		},
		async getPictureUrl(file_no) {
			const serviceName = 'srvfile_attachment_select';
			const url = this.getServiceUrl('file', serviceName, 'select');
			const req = {
				serviceName: serviceName,
				colNames: ['*'],
				condition: [{ colName: 'file_no', value: file_no, ruleType: 'eq' }]
			};
			if (file_no !== null && file_no !== '' && file_no !== undefined) {
				let res = await this.$http.post(url, req);
				if (res.data.state === 'SUCCESS') {
					const data = res.data.data[0];
					if (data) {
						const fileurl = this.$api.srvHost + '/file/download?filePath=' + data.fileurl;
						return fileurl;
					}
				}
			}
		}
	},
	watch: {
		pageItemList: {
			deep: true,
			handler(newVal) {}
		}
	}
};
</script>

<style lang="scss" scoped>
@media screen and (min-width: 750px) {
	.page-wrap {
		position: relative;
		overflow: hidden;
		display: flex;
		flex-wrap: wrap;
		margin: 0 auto;
		flex-direction: column;
		.swiper-view {
			width: 100%;
			min-height: 500px;
			max-width: 1350px;
			max-height: 700px;
			margin: 0 auto;
			.carousel-view {
				width: 100%;
				height: 100%;
				/deep/.screen-swiper {
					height: 100%;
				}
			}
		}
		.menu-view {
			width: 100%;
			margin: 0 auto;
			/deep/ .screen-swiper {
				width: 100%;
			}
		}
		.tab-list-view {
			max-width: 450px;
			margin: 10px auto;
			&:nth-child(2) {
			}
		}
	}
}
@media screen and (max-width: 750px) {
	.menu-view {
		width: 100%;
		margin: 0 auto;
		/deep/ .screen-swiper {
			width: 100%;
		}
	}
	.tab-list-view {
		width: 100%;
		margin: 10px auto;
		&:nth-child(2) {
		}
	}
}
.page-wrap {
	position: relative;
	display: flex;
	flex-wrap: wrap;
	margin-bottom: 80rpx;
	flex-direction: column;
}
</style>
