<template>
	<view class="collect-wrap">
		<view class="camera-wrap" v-if="showCamera">
			<!-- <view class="collect-title" v-if="!showCamera">照片预览</view> -->
			<!-- #ifdef APP-PLUS||MP-WEIXIN -->
			<camera device-position="back" flash="auto" @error="error" style="width:750upx; height: 1115upx">
				<cover-image :src="addType === 'car' ? '../../static/img/car-horizonal.png' : '../../static/img/idcard-vertical.png'" class="scan-img"></cover-image>
			</camera>
			<view class="take-photo-box"><image src="../../static/img/bd_ocr_take_photo_normal.png" @click="takePhoto" class="take-photo"></image></view>
			<!-- #endif -->

			<!-- #ifdef H5 -->
			<view class="collect-button"><button type="primary" @click="takePhoto">从相册选择照片</button></view>
			<!-- #endif -->
		</view>
		<view class="collect-box" v-if="!showCamera">
			<view class="image-box">
				<image src="../../static/img/idcard-horizonal.png" mode="" class="image" v-if="addType === 'person'"></image>
				<image src="../../static/img/car-horizonal.png" mode="" class="image carPreviewImg" v-if="addType === 'car'"></image>
				<image :src="previewImg" mode="aspectFit" class="image previewImg"></image>
				<!-- <image :src="previewImg" mode="aspectFit" class="image previewImg" v-if="addType==='car'"></image> -->
			</view>
			<view class="collect-button">
				<button type="primary" @click="clipImages">识别文字</button>
				<button type="primary" @click="toCarema">重新拍照</button>
			</view>
			<form v-if="addType === 'car'">
				<view class="cu-form-group margin-top">
					<view class="title">车牌号</view>
					<input placeholder="车牌号" name="input" v-model="formData.car_no" />
				</view>
			</form>
			<form v-if="addType === 'person'">
				<view class="cu-form-group margin-top">
					<view class="title">姓名</view>
					<input placeholder="姓名" name="input" v-model="formData.name" />
				</view>
				<view class="cu-form-group margin-top">
					<view class="title">地址</view>
					<input placeholder="地址" name="input" v-model="formData.native" />
				</view>
				<view class="cu-form-group margin-top">
					<view class="title">身份证号</view>
					<input placeholder="身份证号" name="input" v-model="formData.idNo" />
				</view>
			</form>
			<button type="primary" @click="toForm">下一步</button>
			<canvas style="width: 688px; height:1024px;position:fixed;top:99999999999999upx;" canvas-id="previewCarCanvas"></canvas>
			<canvas style="width:110px; height: 145px;position:fixed;top:99999999999999upx;" canvas-id="profileCanvas"></canvas>
			<canvas style="width: 375px; height: 250px;position:fixed;top:99999999999999upx;" canvas-id="previewCanvas"></canvas>
			<canvas style="width: 160px; height: 35px;position:fixed;top:99999999999999upx;" canvas-id="firstCanvas"></canvas>
			<canvas style="width:170px; height: 70px;position:fixed;top:99999999999999upx;" canvas-id="secondCanvas"></canvas>
			<canvas style="width: 235px; height: 40px;position:fixed;top:99999999999999upx;" canvas-id="thirdCanvas"></canvas>
			<canvas style="width: 307px; height: 105px;position:fixed;top:99999999999999upx;" canvas-id="carCandCanvas"></canvas>
		</view>
	</view>
</template>

<script>
// @ is an alias to /src
import uniPopup from '@/components/uni-popup/uni-popup.vue';
import bxform from '@/components/form.vue';
export default {
	name: 'idCardInfoCollect',
	components: { uniPopup, bxform },
	data() {
		return {
			addType: '',
			showCamera: true,
			tempFilePath: '',
			beforeImg: '',
			previewImg: '',
			compressImg: '', //压缩后的图片
			afterImg: '', //裁剪之后的图
			fileList: [],
			picList: [],
			responseList: [],
			dept_no: '',
			formData: {
				name: '',
				id_card_addr: '',
				id_card_no: '',
				profile_picture: '',
				car_photo: '',
				car_no: ''
			},
			fromService: '',
			fieldData:{},
			fieldMapping: {}
		};
	},
	onLoad(option) {
		if (option.fromService) {
			this.fromService = option.fromService;
		}
		if(option.fieldData){
			this.fieldData = option.fieldData
		}
		if (option.fieldMapping) {
			try {
				this.fieldMapping = JSON.parse(option.fieldMapping);
				this.fieldMapping.cardImg = 'identity_image';
				this.fieldMapping.avatar = '';
				// avatar:"",
				// cardImg:"identity_image"
				// idNo: "identity_no"
				// name: "name"
				// native: "native_place"
				Object.values(this.fieldMapping).forEach(item => {
					this.formData[item] = '';
				});
				console.log('this.fieldMapping', this.fieldMapping);
			} catch (e) {
				//TODO handle the exception
			}
		}
		if (option.addType) {
			this.addType = option.addType;
		} else {
			this.addType = 'person';
		}
	},
	methods: {
		toForm() {
			const formData = this.formData;
			// if (this.addType === 'car') {
			// 	uni.redirectTo({
			// 		url: '../checkInCarInfo/checkInCarInfo?formData=' + encodeURIComponent(JSON.stringify(formData)) + '&dept_no=' + this.dept_no
			// 	});
			// } else {
			console.log('formData-takephotos', formData);
			uni.navigateTo({
				url: '/pages/formPage/formPage?formData=' + encodeURIComponent(JSON.stringify(formData)) + '&type=add&serviceName=' + this.fromService+'&fieldData='+this.fieldData
			})
			// uni.redirectTo({
			// 	url: '/pages/formPage/formPage?formData=' + encodeURIComponent(JSON.stringify(formData)) + '&type=add&serviceName=' + this.fromService
			// });
			// }
		},
		getPreviewPic() {
			let that = this;
			let path = this.beforeImg;
			// 将身份证照片转为canvas图像(缩放为375px*250px)
			const uploadUrl = that.$api.upload;
			if (that.addType === 'car') {
				const ctx0 = uni.createCanvasContext('previewCarCanvas', this);
				// 改变中心点carCandCanvas
				// ctx0.translate(0, 688);
				// 旋转270°
				// ctx0.rotate((270 * Math.PI) / 180);
				// ctx0.drawImage(path, 0, 0, 688, 1024);
				ctx0.drawImage(path, 0, 0, 688, 1024);
				ctx0.draw(false, function(e) {
					that.fileList = [];
					that.picList = [];
					that.responseList = [];
					uni.canvasToTempFilePath({
						x: 0,
						y: 0,
						width: 688,
						height: 1024,
						// width: 1024,
						// height: 688,
						destWidth: 688,
						destHeight: 1024,
						fileType: 'jpg',
						canvasId: 'previewCarCanvas',
						success: res => {
							// 车牌
							that.previewImg = res.tempFilePath;
							console.log('car-preview', res.tempFilePath);
							uni.uploadFile({
								url: uploadUrl,
								filePath: res.tempFilePath,
								name: 'file',
								header: {
									bx_auth_ticket: that.$api.onTicket ? that.$api.ticket : uni.getStorageSync('bx_auth_ticket')
								},
								formData: {
									serviceName: 'srv_bxfile_service',
									interfaceName: 'add',
									app_no: 'daq',
									table_name: 'bxdaq_check_car',
									columns: 'car_photo'
								},
								success: uploadFileRes => {
									const data = JSON.parse(uploadFileRes.data);
									that.formData.car_photo = data.file_no;
								}
							});
						}
					});
				});
			} else {
				const ctx0 = uni.createCanvasContext('previewCanvas', this);
				// 改变中心点
				ctx0.translate(0, 250);
				// 旋转270°
				ctx0.rotate((270 * Math.PI) / 180);
				ctx0.drawImage(path, 0, 0, 250, 375);
				ctx0.draw(false, function(e) {
					that.fileList = [];
					that.picList = [];
					that.responseList = [];
					uni.canvasToTempFilePath({
						x: 0,
						y: 0,
						width: 375,
						height: 250,
						destWidth: 375,
						destHeight: 250,
						fileType: 'jpg',
						canvasId: 'previewCanvas',
						success: res => {
							// 身份证
							that.previewImg = res.tempFilePath;
							console.log(that.previewImg);
							uni.uploadFile({
								url: uploadUrl,
								filePath: res.tempFilePath,
								name: 'file',
								header: {
									bx_auth_ticket: that.$api.onTicket ? that.$api.ticket : uni.getStorageSync('bx_auth_ticket')
								},
								formData: {
									serviceName: 'srv_bxfile_service',
									interfaceName: 'add',
									app_no: 'daq',
									table_name: '',
									columns: that.fieldMapping.file ? that.fieldMapping.file : ''
								},
								success: uploadFileRes => {
									const data = JSON.parse(uploadFileRes.data);
									that.formData.id_card_photo = data.file_no;
									that.formData[that.fieldMapping.cardImg] = data.file_no;
								}
							});
						}
					});
				});
			}
		},
		clipImages() {
			uni.showLoading({
				title: '正在识别...'
			});
			// 将身份证照片裁剪为姓名、住址、身份证三个canvas图像
			const that = this;
			if (this.addType === 'person') {
				const ctx = uni.createCanvasContext('firstCanvas', this);
				ctx.drawImage(that.previewImg, 75, 35, 160, 45, 0, 0, 160, 45);
				// ctx.drawImage(that.previewImg, 60, 20, 160, 35, 0, 0, 160, 35);
				ctx.draw(false, function(e) {
					const attr = {
						width: 160,
						height: 40,
						id: 'firstCanvas',
						label: 'name'
					};
					that.getImg(attr);
				});
				// 头像
				const ctx1 = uni.createCanvasContext('profileCanvas', that);
				ctx1.drawImage(that.previewImg, 225, 35, 110, 145, 0, 0, 110, 145);
				ctx1.draw(false, function(e) {
					const attr = {
						width: 110,
						height: 145,
						id: 'profileCanvas',
						label: 'avatar'
					};
					that.getImg(attr);
				});
				// 住址
				const ctx2 = uni.createCanvasContext('secondCanvas', that);
				ctx2.drawImage(that.previewImg, 75, 110, 170, 70, 0, 0, 170, 70);
				// ctx2.drawImage(that.previewImg, 60, 110, 170, 70, 0, 0, 170, 70);
				ctx2.draw(false, function(e) {
					const attr = {
						width: 170,
						height: 70,
						id: 'secondCanvas',
						label: 'native'
					};
					that.getImg(attr);
				});
				// 身份证号
				const ctx3 = uni.createCanvasContext('thirdCanvas', that);
				ctx3.drawImage(that.previewImg, 130, 160, 235, 70, 0, 0, 235, 70);
				ctx3.draw(false, function(e) {
					const attr = {
						width: 235,
						height: 40,
						id: 'thirdCanvas',
						label: 'idNo'
					};

					that.getImg(attr);
				});
			} else if (this.addType === 'car') {
				const ctx = uni.createCanvasContext('carCandCanvas', this);
				ctx.drawImage(that.previewImg, 190, 637, 317, 107, 0, 0, 318, 107);
				// ctx.drawImage(that.previewImg, 354, 433, 307, 105, 0, 0, 307, 105);
				ctx.draw(false, function(e) {
					const attr = {
						width: 307,
						height: 105,
						id: 'carCandCanvas',
						label: 'car_no'
					};
					that.getImg(attr);
				});
			}
		},
		getImg(canvasData) {
			// 将canvas图像转换成png格式图片
			const that = this;
			uni.canvasToTempFilePath({
				x: 0,
				y: 0,
				width: canvasData.width,
				height: canvasData.height,
				destWidth: canvasData.width,
				destHeight: canvasData.height,
				canvasId: canvasData.id,
				success: res => {
					that.picList.push(res.tempFilePath);
					const uploadUrl = that.$api.upload;
					console.log(res.tempFilePath);
					uni.uploadFile({
						url: uploadUrl,
						filePath: res.tempFilePath,
						name: 'file',
						header: {
							bx_auth_ticket: that.$api.onTicket ? that.$api.ticket : uni.getStorageSync('bx_auth_ticket')
						},
						formData: {
							serviceName: 'srv_bxfile_service',
							interfaceName: 'add',
							app_no: 'daq',
							table_name: '',
							columns: ''
						},
						success: uploadFileRes => {
							// 图片上传成功后的回调
							const data = JSON.parse(uploadFileRes.data);
							if (canvasData.label === 'profile_picture') {
								console.log('profile_picture', data);
								console.log(res.tempFilePath);
								that.formData.profile_picture = data.file_no;
							} else {
								that.fileList.push(data.file_no);
								that.responseList.push({
									file_no: data.file_no,
									label: canvasData.label,
									content: ''
								});
								const reqUrl = that.getServiceUrl('daq', 'srvdaq_orc_text_extraction', 'operate');
								const reqData = [
									{
										serviceName: 'srvdaq_orc_text_extraction',
										data: [
											{
												file_no: data.file_no
											}
										]
									}
								];
								that.$http.post(reqUrl, reqData).then(response => {
									// 得到识别的文字
									if (response.data.state === 'SUCCESS') {
										let list = that.responseList;
										const resp = response.data.response[0].response; // {file_no:text}
										if (resp) {
											console.log(
												list,
												'list',
												that.formData,
												that.fieldMapping,
												that.formData[that.fieldMapping.name],
												that.formData[that.fieldMapping.native],
												that.formData[that.fieldMapping.idNo]
											);
											list.map(listItem => {
												if (resp[listItem.file_no]) {
													listItem['value'] = resp[listItem.file_no];
													console.log(listItem);
													that.$set(that.formData, canvasData.label, resp[listItem.file_no]);
													that.$set(that.formData, that.fieldMapping[canvasData.label], resp[listItem.file_no]);
													console.log(that.formData, 'that.formData');
													uni.hideLoading();
												}
											});
											console.log(response.data.response[0].response);
										}
										console.log(response.data.response);
									} else {
										console.log(response.data.resultMessage);
									}
								});
							}
						}
					});
				}
			});
		},
		toCarema() {
			this.formData = {
				name: '',
				id_card_addr: '',
				id_card_no: ''
			};
			this.showCamera = true;
		},
		takePhoto() {
			const that = this;
			// this.showCanvas = true;
			// #ifdef H5
			// 从本地选择图片
			uni.chooseImage({
				count: 1, //默认9
				sizeType: ['original'], //可以指定是原图还是压缩图，默认二者都有
				sourceType: ['album'], //从相册选择
				success: function(res) {
					that.beforeImg = res.tempFilePaths[0];
					that.clipImages();
				}
			});
			// #endif
			// #ifdef APP-PLUS||MP-WEIXIN
			// 得到camera组件拍到的照片
			const ctx = uni.createCameraContext();
			ctx.takePhoto({
				quality: 'high',
				success: res => {
					// 拍到的照片
					this.beforeImg = res.tempImagePath;
					this.showCamera = false;
					// this.clipImages();
					this.getPreviewPic();
				}
			});
			// #endif
		},
		error(e) {
			console.log(e.detail);
		}
	}
};
</script>

<style lang="scss">
.collect-wrap {
	width: 100%;
	height: 100vh;
	display: flex;
	flex-direction: column;
	.camera-wrap {
		background-color: #000;
		width: 100%;
		height: 100%;
		overflow: auto;
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		.collect-title {
			width: 100%;
			color: #fff;
			background-color: #000;
			font-size: 36upx;
			font-weight: 600;
			line-height: 100upx;
			text-align: center;
		}
		.scan-img {
			opacity: 0.4;
			width: 750upx;
			height: 1115upx;
			margin: 0 auto;
			// padding: 30upx;
		}
		.take-photo-box {
			width: 100%;
			height: 140upx;
			background-color: #000;
			display: flex;
			justify-content: center;
			align-items: center;
			.take-photo {
				width: 100upx;
				height: 100upx;
			}
		}
	}
	.collect-box {
		width: 100%;
		display: flex;
		flex-direction: column;
		.image-box {
			width: 750upx;
			height: 504upx;
			// width: 504upx;
			// height: 750upx;
			text-align: center;
			position: relative;
			.previewImg {
				width: 750upx;
				height: 504upx;
			}

			.image {
				width: 100%;
				height: 100%;
				&:first-child {
					z-index: 2;
					position: absolute;
					top: 0;
				}
				&:last-child {
					z-index: 1;
				}
			}
			.carPreviewImg {
				width: 340upx;
				height: 504upx;
				left: calc(50% - 170upx);
			}
		}
		.collect-button {
			width: 100%;
			display: flex;
			justify-content: space-around;
			align-items: center;
			height: 150upx;
		}
	}
}
</style>
