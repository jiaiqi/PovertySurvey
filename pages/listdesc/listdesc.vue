<template>
	<view class="">
		<cu-custom bgColor="bg-gradual-green" :isBack="true">
			<block slot="backText">返回</block>
			<block slot="content">{{title}}</block>
		</cu-custom>
		<view class="topContent">
			<view class="wrapheadr">
			  <!-- <view class="">
			    <view class="titlecolor">122</view>
			    <view class="text">区域在册人数</view>
			  </view> -->
			  <view class="">
			    <view class="titlecolor">{{pageInfo.total}}</view>
			    <view class="text">特困人数</view>
			  </view>
			  <view class="">
			    <view class="titlecolor">{{nglum}}</view>
			    <view class="text">管理人员</view>
			  </view>
			</view>
			<view class="middle">
				<view class=""  @click="mine()"><text class="lg text-gray cuIcon-comment"></text>纪实感悟</view>
				<view class="" @click="mine('mine')"> <text class="lg text-gray cuIcon-friendfamous"></text>我的</view>
			</view>
		</view>
			
		<s-pull-scroll ref="pullScroll" :back-top="true" :pullDown="pullDown" :top="'255'" :pullUp="loadData">
		<view class="wrapCont">
			<view class="Jlsuccess" style="" v-for="(item,index) in list" :key="index">
			  <image class="heaserImg" :src="item.photo?item.photo:headerimg" mode=""></image>
			  <view class="infrom" style="">
				<view class="">
					<text class="nametitle">{{item.full_name}}</text>
					<text class="tran">***</text>
					<text>{{item._sex_disp}}</text>
				</view>
			    <view class="Inumber">身份证号码：{{item.id_card?item.id_card:'暂无'}} </view>
			  </view>
			</view>
		</view>
	</s-pull-scroll>
		<view class="public-button-box">
			<view class="add-button" @click="adddesc()"></view>
		</view>
	</view>
</template>

<script>
	import sPullScroll from '@/components/s-pull-scroll/index.vue'
	export default{
		components: {
			sPullScroll
		},
		data(){
			return{
				list:[],
				title:'',
				headerimg: 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3491929439,4106440758&fm=26&gp=0.jpg',
				pageInfo: {
					pageNo: 1,
					rownumber: 10,
					total: 0,
				},
				nglum:0
			}
		},
		onLoad(option) {
			let item = (JSON.parse(decodeURIComponent(option.itemDetail)))
			this.title = item.path_name.substring(1,item.path_name.length-1)
			this.refresh()
			this.getNum(this.title)
		},
		methods:{
			adddesc(){
				uni.navigateTo({
					url: '/pages/formPage/formPage?serviceName=' + 'srvdaq_tkry_visit_record_select' + '&type=add'
				});
			},
			mine(val){
				let con={
					colName: 'create_user',
					ruleType: 'eq',
					value: uni.getStorageSync('login_user_info').user_no
				}
				uni.navigateTo({
					url: '/pages/list/list?serviceName=srvdaq_tkry_visit_record_select'+`&cond=${val=='mine'? decodeURIComponent(JSON.stringify(con)):''}`
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
					this.getInfoList(this.title).then(() => {})
				}, 1000)
			},
			pullDown(pullScroll) {
				this.pageInfo.pageNo = 1;
				setTimeout(() => {
					this.getInfoList(this.title)
				}, 500)
			},
			async getNum(val){
				let optionType = "select"
				let srv = "srvdaq_tkry_users_select"
				let app = 'daq'
				let req = {
					"serviceName": "srvdaq_tkry_users_select",
					"colNames": ["*"],
					"condition": [],
					"order": [],
				}
				let con ={
					colName: 'area_id',
					ruleType: 'like',
					value: val
				}
				if(val){
					req.condition.push(con)
				}
				let res = await this.onRequest(optionType, srv, req, app)
				if(res.data.data.length>0){
					this.nglum = res.data.data.length
				}
			},
		 async	getInfoList(val){
				let optionType = "select"
				let srv = "srvdaq_tkry_people_select"
				let app = 'daq'
				let req = {
					"serviceName": "srvdaq_tkry_people_select",
					"colNames": ["*"],
					"condition": [],
					"order": [],
					"page": {
						"pageNo": this.pageInfo.pageNo,
						"rownumber": this.pageInfo.rownumber
					},
				}
				let con ={
					colName: 'area_id',
					ruleType: 'like',
					value: val
				}
				if(val){
					req.condition.push(con)
				}
				let res = await this.onRequest(optionType, srv, req, app)
				if(res.data&&res.data.data&&res.data.data.length>0){
					let itemData = res.data.data
					this.pageInfo.total = res.data.page.total;
					itemData.forEach((item,index,itemData)=>{
						this.getFilePath(item.photo).then(a=>{
							if(a&& a[0]&&a[0].fileurl){
								let urls = this.$api.downloadImg + a[0].fileurl +'&bx_auth_ticket='+uni.getStorageSync('bx_auth_ticket')
								item.photo = urls
							}
						})
					})
					if (this.pageInfo.pageNo == 1) {
						this.list = (itemData)
					} else {
						this.list = this.list.concat(itemData)
					}
					let page = this.pageInfo;
					this.$nextTick(()=>{
						if (page.rownumber * page.pageNo >= page.total) {
							this.$refs.pullScroll.finish();
						} else {
							this.$refs.pullScroll.success();
						}
					})
				}else{
					this.list=[]
					this.$refs.pullScroll.finish();
				}
				
			}
		}
	}
</script>

<style lang="less" scoped>
	.wrapheadr {
	  margin: 20upx;
	  background-color: #98a79640;
	  display: flex;
	  justify-content: space-between;
	  padding: 20upx 50upx;
	  text-align: center;
	  border-radius: 8upx;
	  .titlecolor {
	    color: #5173e1;
	    font-size: 35upx;
	    line-height: 30px;
	  }
	
	  .text {
	    font-size: 25upx;
	    color: #959595;
	    line-height: 30px;
	  }
	}
	.Jlsuccess {
	  display: flex;
	  margin: 20upx;
	  background: #f3f3f3;
	  padding: 26upx;
	  border-radius: 8upx;
	  .infrom {
	    height: 80upx;
	    margin-left: 18upx;
		.nametitle {
		  font-size: 30upx;
		  font-weight: bold;
		}
		.tigb {
		  font-size: 24upx;
		  padding: 4upx 10upx;
		  text-align: center;
		  border-radius: 7upx;
		}
		.jointime {
		  display: flex;
		  justify-content: space-between;
		  
		}
	  }
	  .Inumber {
	    font-size: 26upx;
	    color: #959595;
	    line-height: 50upx;
	  }
	  .heaserImg {
	    display: inline-block !important;
	    width: 80upx !important;
	    height: 80upx !important;
	    border-radius: 50% !important;
	  }
	}
	.tran{
		opacity: 0;
	}
	.middle{
		display: flex;
		justify-content: space-between;
		padding:  0 100upx;
		font-size: 32upx;
		.text-gray{
			margin-right: 8upx;
			font-size: 36upx;
		}
	}
	.add-button {
		position: fixed;
		bottom: 100upx;
		left: calc(50% - 50upx);
		width: 100upx;
		height: 100upx;
		border-radius: 50%;
		background-color: #0bc99d;
		z-index: 100;
		&::before {
			position: absolute;
			width: 8upx;
			height: 60upx;
			left: calc(50% - 4upx);
			top: 20upx;
			content: '';
			background-color: #fff;
		}
		&::after {
			position: absolute;
			width: 60upx;
			height: 8upx;
			content: '';
			top: calc(50% - 4upx);
			left: 20upx;
			background-color: #fff;
		}
		&:active {
			transform: rotate(45deg);
			transition: all 0.2s;
		}
	}
	.topContent{
		width: 100%;
		position: absolute;
		top: 50px;
	}
</style>
