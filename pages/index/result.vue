<template>
	<view class="content">
		<view class="navheight"></view>
		<!--头部 开始-->
		<view class="view-header">
			<view @click="goIndex()" class="view-header-icon">
				<view class="view-header-icon-left">
					<u-icon name="arrow-left" color="#ffffff" size="50"></u-icon>
				</view>
				<view class="view-header-icon-position">本次成绩</view>
			</view>
			<view class="view-header-num">
				<view class="view-header-num-num">{{score}}</view>
				<view class="view-header-num-text">{{score<90?'考试未通过':'恭喜你,考试通过'}}</view>
			</view>
		</view>
		<!--头部 结束-->

		<!--cell单元格 开始-->
		<view class="view-cell">
			<view @click="goCuoti()" class="view-cell-left">
				<view class="view-cell-left-icon">
					<image src="https://img.alicdn.com/imgextra/i4/2911614117/O1CN01XG05b21gHdM38S7Qb_!!2911614117.png">
					</image>
				</view>
				<view class="view-cell-left-text">查看错题</view>
			</view>
			<view @click="goCuoti()" class="view-cell-right">
				<view class="view-cell-right-text">{{tips}}</view>
				<view class="view-cell-right-icon">
					<u-icon name="arrow-right" color="#dfdfdf" size="16"></u-icon>
				</view>
			</view>
		</view>
		<!--cell单元格 结束-->

		<!--底部 开始-->
		<view class="view-footer">
			<view @click="goSingleTest()" class="view-footer-list view-footer-list1">再考一次</view>
			<!-- <view 
				@click="goMncj()"
				class="view-footer-list">历史成绩</view> -->
		</view>
		<!--底部 开始-->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				score: 0,
				tips: ''
			}
		},
		methods: {
			goSingleTest() {
				uni.navigateTo({
					url:'./index'
				})
			},
			goIndex() {
				uni.navigateTo({
					url:'./index'
				})
			}
		},
		onReady() {
			// #ifdef H5
			var ua = window.navigator.userAgent.toLowerCase();
			if (ua.match(/micromessenger/i) == 'micromessenger') {
				this.$jwx.hideShareMenu();
			}
			// #endif
		},
		onLoad(options) {
			console.log(JSON.parse(options.item), '带古来');
			let result = JSON.parse(options.item)
			this.score = result.results
			this.tips = '作对' + result.success + '道，做错' + result.error + '道';
		}

	}
</script>

<style scoped lang="scss">
	/*顶部返回键 开始*/
	.view-back {
		z-index: 10;
		background-color: #ffffff;

		.view-back-in {
			position: fixed;
			top: 6rpx;
			left: 6rpx;
			width: 44rpx;
			height: 44rpx;
			z-index: 3000;

			image {
				width: 100%;
				height: 100%;
			}
		}

	}

	/*顶部返回键 结束*/
	.content {
		background-color: #FFFFFF;
	}

	.navheight {
		// height: var(--status-bar-height);
		//height: calc( 10rpx + var(--status-bar-height));
		width: 100%;
	}

	/*头部 开始*/
	.view-header {
		padding-top: 20rpx;
		position: relative;
		background-image: url('https://img.alicdn.com/imgextra/i1/2911614117/O1CN018hbM8C1gHdMA6PjC6_!!2911614117.png');
		background-size: 100% 700rpx;
		height: 700rpx;
	}

	.view-header-icon {
		position: relative;
		top: 70rpx;

		.view-header-icon-left {
			padding: 0 20rpx;
		}

		.view-header-icon-position {
			font-size: 16px;
			color: #ffffff;
			position: absolute;
			top: 4rpx;
			left: 50%;
			transform: translate(-50%);
		}
	}

	.view-header-num {
		position: absolute;
		bottom: 120rpx;
		left: 50%;
		transform: translate(-50%);
		color: #f1f0eb;
		text-align: center;
		font-size: 16px;

		.view-header-num-num {
			color: #FFFFFF;
			font-size: 40px;
		}

		.view-header-num-text {
			margin-top: 80rpx;
		}
	}

	/*头部 结束*/

	/*cell单元格 开始*/
	.view-cell {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 42rpx 40rpx;
		border-bottom: 2rpx solid #f5f5f5;
	}

	.view-cell-left {
		display: flex;
		align-items: center;

		.view-cell-left-icon {
			width: 50rpx;
			height: 46rpx;

			image {
				width: 100%;
				height: 100%;
			}
		}

		.view-cell-left-text {
			font-size: 16px;
			font-weight: 700;
			color: #555;
			margin-left: 16rpx;
		}
	}

	.view-cell-right {
		display: flex;
		align-items: center;
		color: #dfdfdf;
		font-size: 13px;

		.view-cell-right-icon {
			margin-left: 30rpx;
			padding-top: 3rpx;
		}
	}

	/*cell单元格 结束*/

	/*底部 开始*/
	.view-footer {
		padding: 50rpx 194rpx 0 194rpx;
	}

	.view-footer-list {
		height: 84rpx;
		line-height: 84rpx;
		border: 4rpx solid #f2cca3;
		border-radius: 40rpx;
		margin-top: 24rpx;
		font-size: 16px;
		color: #e9ac5d;
		font-weight: 700;
		text-align: center;
	}

	.view-footer-list1 {
		background-color: #f19333;
		color: #fae4c1;
		border: 0;
	}

	/*底部 开始*/
</style>
