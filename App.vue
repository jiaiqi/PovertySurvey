<script>
import Vue from 'vue';
export default {
	onLaunch: function() {
		uni.getSystemInfo({
			success: function(e) {
				// #ifndef MP
				Vue.prototype.StatusBar = e.statusBarHeight;
				if (e.platform == 'android') {
					Vue.prototype.CustomBar = e.statusBarHeight + 50;
				} else {
					Vue.prototype.CustomBar = e.statusBarHeight + 45;
				}
				// #endif

				// #ifdef MP-WEIXIN
				Vue.prototype.StatusBar = e.statusBarHeight;
				let custom = wx.getMenuButtonBoundingClientRect();
				Vue.prototype.Custom = custom;
				Vue.prototype.CustomBar = custom.bottom + custom.top - e.statusBarHeight;
				// #endif

				// #ifdef MP-ALIPAY
				Vue.prototype.StatusBar = e.statusBarHeight;
				Vue.prototype.CustomBar = e.statusBarHeight + e.titleBarHeight;
				// #endif
			}
		});

		Vue.prototype.ColorList = [
			{
				title: '嫣红',
				name: 'red',
				color: '#e54d42'
			},
			{
				title: '桔橙',
				name: 'orange',
				color: '#f37b1d'
			},
			{
				title: '明黄',
				name: 'yellow',
				color: '#fbbd08'
			},
			{
				title: '橄榄',
				name: 'olive',
				color: '#8dc63f'
			},
			{
				title: '森绿',
				name: 'green',
				color: '#39b54a'
			},
			{
				title: '天青',
				name: 'cyan',
				color: '#1cbbb4'
			},
			{
				title: '海蓝',
				name: 'blue',
				color: '#0081ff'
			},
			{
				title: '姹紫',
				name: 'purple',
				color: '#6739b6'
			},
			{
				title: '木槿',
				name: 'mauve',
				color: '#9c26b0'
			},
			{
				title: '桃粉',
				name: 'pink',
				color: '#e03997'
			},
			{
				title: '棕褐',
				name: 'brown',
				color: '#a5673f'
			},
			{
				title: '玄灰',
				name: 'grey',
				color: '#8799a3'
			},
			{
				title: '草灰',
				name: 'gray',
				color: '#aaaaaa'
			},
			{
				title: '墨黑',
				name: 'black',
				color: '#333333'
			},
			{
				title: '雅白',
				name: 'white',
				color: '#ffffff'
			}
		];
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		console.log('App Hide');
	}
};
</script>

<style lang="less">
@import 'colorui/main.css';
@import 'colorui/icon.css';
// @media (prefers-color-scheme: dark) {
// 	// dark mode
// 	html {
// 		filter: invert(1) hue-rotate(180deg);
// 	}
// 	html img {
// 		filter: invert(1) hue-rotate(180deg);
// 	}
// }
// html {
// 	transition: color 300ms, background-color 300ms;
// }
/*每个页面公共css */
#app,
uni-page-body,
uni-page-wrapper {
	background: #ffffff !important;
}
html,
body {
	height: auto;
	background: #ffffff !important;
}
uni-page-body > uni-view {
	height: 100%;
}
uni-page-wrapper,
uni-page-body {
	height: calc(100vh - 44px);
}
.cu-modal {
	text-align: left;
}
.cu-card.article > .cu-item .title {
	padding: 0;
}
.uni-input-placeholder,
.uni-textarea-placeholder {
	box-sizing: border-box;
	padding: 10upx;
}
.cu-form-group {
	background-color: #ffffff;
	padding: 1upx 30upx;
	display: flex;
	flex-direction: column;
	align-items: self-start;
	min-height: 140upx;
	justify-content: flex-start;
	.flex_row_c {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}
	input,
	textarea {
		flex: 1;
		font-size: 32upx;
		color: #555;
		box-sizing: border-box;
		padding: 10upx;
		text-indent: 0;
		background: transparent;
		border: 1rpx solid #d0d4d6;
		resize: none;
		border-radius: 4upx;
		outline: none; /*清除选中效果的默认蓝色边框 */
		-webkit-appearance: none; /*清除浏览器默认的样式 */
		line-height: normal; /*光标问题*/

		&:focus {
			border: 1upx solid #0055ff;
			color: #333333;
			// background-color: #e9eef0;
			border-bottom-left-radius: 1px;
			border-bottom-right-radius: 1px;
		}
	}
	.item-group input,
	> .item-group textarea {
		flex: 1;
		font-size: 32upx;
		color: #555;
		box-sizing: border-box;
		padding: 10upx;
		text-indent: 0;
		background: transparent;
		border: 1rpx solid #d0d4d6;
		resize: none;
		border-radius: 4upx;
		outline: none; /*清除选中效果的默认蓝色边框 */
		-webkit-appearance: none; /*清除浏览器默认的样式 */
		line-height: normal; /*光标问题*/

		&:focus {
			border: 1upx solid #0055ff;
			color: #333333;
			// background-color: #e9eef0;
			border-bottom-left-radius: 1px;
			border-bottom-right-radius: 1px;
		}
	}
	uni-input,
	input {
		line-height: 2.8em;
		height: 2.8em;
		min-height: 2.8em;
		overflow: hidden;
		margin: 0upx;
		width: 100%;
		padding: 0;
		.uni-input-input,
		input {
			flex: 1;
			font-size: 32upx;
			color: #555;
			box-sizing: border-box;
			padding: 10upx;
			text-indent: 0;
			background: transparent;
			border: 1rpx solid #d0d4d6;
			resize: none;
			border-radius: 4upx;
			outline: none; /*清除选中效果的默认蓝色边框 */
			-webkit-appearance: none; /*清除浏览器默认的样式 */
			line-height: normal; /*光标问题*/
		}
		.uni-input-input:focus,
		.uni-textarea-textarea:focus {
			border: 1upx solid #0055ff;
			color: #333333;
			// background-color: #e9eef0;
			border-bottom-left-radius: 1px;
			border-bottom-right-radius: 1px;
		}
		&.valid_error {
			.uni-input-input {
				border: 1rpx solid #ff0000;
			}
		}
		&.date-input {
			.uni-input-wrapper {
				.uni-input-input {
					padding-right: 60upx !important;
				}
			}
		}
	}

	.title {
		// min-width: calc(4em + 15px);
		width: 100vw;
		display: block;
		height: 60upx;
		line-height: 60upx;
		box-sizing: border-box;
	}
	> view {
		width: 100%;
	}
	.title.valid_error {
		color: #ff0000;
	}
	uni-textarea,
	textarea {
		margin: 0;
		.uni-textarea-textarea {
			flex: 1;
			font-size: 32upx;
			color: #555;
			box-sizing: border-box;
			padding: 10upx;
			text-indent: 0;
			background: transparent;
			border: 1rpx solid #d0d4d6;
			resize: none;
			border-radius: 4upx;
			outline: none; /*清除选中效果的默认蓝色边框 */
			-webkit-appearance: none; /*清除浏览器默认的样式 */
			line-height: normal; /*光标问题*/
		}
		&.valid_error {
			.uni-textarea-textarea {
				border: 1rpx solid #ff0000;
			}
		}
		.uni-input-input:focus,
		.uni-textarea-textarea:focus {
			border: 1upx solid #0055ff;
			color: #333333;
			// background-color: #e9eef0;
			border-bottom-left-radius: 1px;
			border-bottom-right-radius: 1px;
		}
	}
}

// uni-radio-group{
// 	display: flex;
// 	flex-direction: row;
// 	flex-wrap:wrap;
// 	width: 100%;
// 	justify-content: flex-start;
// 	flex: 2;
// 	uni-radio{
// 		padding: 20upx;
// 		width: 40%;
// 		.uni-radio-wrapper{
// 			span{
// 				font-size: 1.4em;
// 				padding: 0 10upx;
// 			}
// 		}
// 	}
// }
// uni-checkbox .uni-checkbox-input.uni-checkbox-input-checked{
// 	background-color: #0081FF;
// }
uni-page-head .uni-page-head {
	display: none;
}
uni-app uni-page-head {
	// display: none;
	display: none;
	box-sizing: border-box;
}
.cu-dialog {
	height: 100vh;
}
.show {
	display: block;
}
.hide {
	display: none !important;
}
.navTabBox {
	font-size: 36upx !important;
}
</style>
