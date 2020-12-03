<template>
	<view class="wrap">
		<view style="margin-bottom: 20rpx;" v-if="notice.length > 0">
			<u-notice-bar type="primary" mode="horizontal" :duration="3000" :is-circular="true" :list="notice"></u-notice-bar>
		</view>

		<u-swiper :list="list"></u-swiper>

		<view class="haochi">
			<view class="section-view">
				<image class="section-image" src="/static/images/common/coupons_fill.png"></image>
				<text class="section-title">优惠列表</text>
			</view>
		</view>

		<u-popup class="sheet-class" v-model="show" mode="bottom">
			<view class="share-button-group">
				<button open-type="share" class="share-button-list">发送给好友</button>
				<button @click="shareTimeLine" class="share-button-list">分享到朋友圈</button>
				<!-- <button @click="createImage" lang="zh_CN" open-type="getUserInfo" @getuserinfo="bindGetUserInfo" class="share-button-list">生成海报</button> -->
				<u-gap height="10" bg-color="#bbb"></u-gap>
				<button @click="show = false" class="share-button-list" share-button-list>取消</button>
			</view>
		</u-popup>

		<view class="goods-view" v-for="(item, index) in projects" :key="index" @click="goToProject(index)">
			<view></view>
			<view class="goods-content-wrap">
				<view>
					<!-- <view class="share-bar">
						<text class="share-text">万水千山总是情，点个分享行不行</text>
					<image style="height: 40rpx;width: 40rpx;" src="../../static/images/common/send.png"></image>
					</view> -->
					<view class="goods-main-content">
						<image :src="item.image" class="goods-image"></image>
					</view>

					<view class="goods-title-view">
						<view style="width: 10%;">
							<view class="goods-tag"><text class="tag-text">限时</text></view>
						</view>

						<view style="display: flex;flex-flow: row wrap;justify-content: flex-start;">
							<text class="goods-title">{{ item.title}}</text>
						</view>

					</view>
					<view class="price-buy">
						<view class="goods-price" @click.stop="shareData(index)">
							<image class="share-image"  src="/static/images/common/send.png"></image>
							<text class="price-text">
								{{item.description}}
							</text>
						</view>

						<view class="goods-buy" @click.stop="goToProject(index)"><text class="buy-text">立即领取</text></view>
					</view>
				</view>
			</view>
		</view>
		<u-loadmore :status="loadStatus" @loadmore="getProjects" :icon-type="iconType" :load-text="loadText"></u-loadmore>
	</view>

</template>

<script>
	export default {
		data() {
			return {
				show: false,
				index: 0,
				userId: 0,
				canIUse: 0,
				projects: [],
				loadStatus: 'loadmore',
				isEnd: 0,
				page: 1,
				total: 0,
				pageSize: 10,
				iconType: 'flower',
				notice: [],
				loadText: {
					loadmore: '轻轻上拉',
					loading: '努力加载中',
					nomore: '到底了'
				},
				list: [{
					image: 'https://cdn.uviewui.com/uview/swiper/1.jpg',
					title: '昨夜星辰昨夜风，画楼西畔桂堂东'
				}, ],
			};
		},
		onLoad() {
			var user = uni.getStorageSync('user') || {};
			if (Object.keys(user).length > 0) {
				this.userId = user.userId;
				this.canIUse = user.canIUse;
			}
			this.getNotice();
			this.getBanner();
			this.getProjects();
		},
		onShareAppMessage(res) {
			this.show = false;
			let index = this.index;
			let project = this.projects[index]
			let title = project.description
			if (!title) {
				title = "优惠券多多，优惠多多！"
			}
			if (res.from === 'button') { // 来自页面内分享按钮
				return {
					title: title,
					imageUrl: project.share_image,
					path: '/pages/index/index'
				}
			}

		},
		methods: {
			shareTimeLine() {
				uni.showToast({
					title: "敬请期待..."
				})
				this.show = false;
			},
			shareData(index) {
				this.show = true;
				this.index = index;
			},
			onReachBottom() {
				if (this.isEnd) {
					this.loadStatus = 'nomore';
					return;
				}
				this.loadStatus = 'loading';
				this.page = ++this.page;
				this.getProjects();
				if (this.isEnd) {
					this.loadStatus = 'nomore';
				} else {
					this.loadStatus = 'loading';
				}
			},
			getProjects() {
				uni.showLoading({
					title: '加载中'
				});
				uni.request({
					url: this.websiteUrl + '/api/get-list-data?page=' + this.page + '&pageSize=' + this.pageSize,
					method: 'GET',
					success: res => {
						var t = res.data.data.list;

						this.projects = this.projects.concat(t);
						this.isEnd = res.data.data.isEnd;
						if (this.isEnd) {
							this.loadStatus = 'nomore';
						}
						uni.hideLoading();
					},
					fail: () => {
						uni.hideLoading();
					},
					complete: () => {}
				});
			},
			getNotice() {
				uni.request({
					url: this.websiteUrl + '/api/notice-list',
					method: 'GET',
					success: res => {
						this.notice = res.data.data;
					},
					fail: () => {},
					complete: () => {}
				});
			},
			getBanner() {
				uni.showLoading({
					title: '加载中'
				});
				uni.request({
					url: this.websiteUrl + '/api/banner-list',
					method: 'GET',
					success: res => {
						this.list = res.data.data;
						uni.hideLoading();
					},
					fail: () => {
						uni.hideLoading();
					},
					complete: () => {}
				});
			},
			goToProject(index) {
				let project = this.projects[index]
				var that = this;
				if (project.is_text == 1) {
					uni.setClipboardData({
						data: project.share_text
					})
					uni.showToast({
						title: "复制成功"
					})
				} else {
					uni.navigateToMiniProgram({
						appId: project.app_id,
						path: project.path,
						success(res) {
							// 打开成功
							let params = {
								app_id: project.app_id,
								project_id: project.id,
								userId: that.userId,
								remark: project.title,
							}
							uni.request({
								url: that.websiteUrl + '/api/add-click-log',
								method: 'POST',
								data: params,
								header: {
									'content-type': 'application/x-www-form-urlencoded'
								},
								success: res => {
									console.log(res)
								},
								fail: () => {},
								complete: () => {}
							});
						}
					})
				}

			},
			shareProject: function(index) {
				let project = this.projects[index]
				uni.navigateToMiniProgram({
					appId: project.app_id,
					path: project.path,
					success(res) {

					}
				})
			},
			previewImage: function(index) {

			},
		}
	}
</script>

<style lang="scss">
	.wrap {
		padding: 20rpx;
	}

	.goods-view {
		opacity: 0.99;
		margin-top: 20rpx;
	}

	.goods-view-title {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
	}

	.goods-content-wrap {
		background: #ffffff;
		box-shadow: 0px 0px 10.9rpx 0px rgba(225, 225, 225, 0.5);
		border-radius:23.7rpx 23.7rpx;
		border: 1px solid #ccc;
	}

	.goods-main-content {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 100%;
		padding-bottom: 20rpx;
	}

	.goods-image {
		width: 100%;
		height: 250rpx;
		border-radius: 23.7rpx 23.7rpx 0 0;
	}

	.goods-tip {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
	}

	.goods-tip-text {
		font-size: 20rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: rgba(179, 179, 179, 1);
		margin-left: 40rpx;
		margin-right: 40rpx;
	}

	.goods-title-view {
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		margin-top: 31rpx;
		margin-left: 40rpx;
		margin-right: 40rpx;
	}

	.goods-tag {
		width: 71rpx;
		height: 38rpx;
		background: #fc6885;
		border-radius: 13rpx;
		text-align: center;
		line-height: 33rpx;
	}

	.tag-text {
		width: 42rpx;
		height: 29rpx;
		font-size: 20rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: rgba(255, 255, 255, 1);
		line-height: 29rpx;
	}

	.goods-title {
		margin-left: 31rpx;
		font-size: 25rpx;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: rgba(67, 65, 65, 1);
		line-height: 33rpx;
	}

	.price-buy {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
		margin-left: 40rpx;
		margin-right: 40rpx;
		margin-bottom: 20rpx;
	}

	.goods-price {
		margin-top: 18rpx;
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		align-items: center;
		
	}

	.goods-buy {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-top: 18rpx;
		width: 142rpx;
		height: 51rpx;
		text-align: center;
		line-height: 51rpx;
		background: #1296db;
		border-radius: 25rpx;
	}

	.price-text {
		margin-left: 5rpx;
		font-size: 24rpx;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #C0C4CC;
	}

	.buy-text {
		font-size: 22rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #FFFFFF;
		line-height: 31rpx;
	}

	.price-text-2 {
		font-size: 22rpx;
	}

	.botton-tip {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
		margin-left: 40rpx;
		margin-right: 40rpx;
		margin-top: 15rpx;
	}

	.market-price-text {
		font-size: 22rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: rgba(179, 179, 179, 1);
		text-decoration: line-through;
	}

	.buy-num-text {
		font-size: 22rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: rgba(179, 179, 179, 1);
	}

	button::after {
		border: none;
	}

	.sheet-class {
		z-index: 999;
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
	}

	.share-button-group {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.share-button-list {
		width: 100%;
		background: #ffffff;
	}
	.haochi {
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		align-items: center;
		margin-top: 20rpx;
		opacity: 0.99;
	}
	.section-title {
		font-weight: 500;
		color: #20201f;
		font-size: 29rpx;
		height: 40rpx;
		line-height: 40rpx;
		font-family: PingFangSC-Medium, PingFang SC;
	}
	
	.section-view {
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		align-items: center;
	}
	
	.section-image {
		width: 30.9rpx;
		height: 30.9rpx;
		padding-right: 7rpx;
	}
	.share-bar {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
		margin: 0 auto;
		height: 60rpx;
		border-radius: 23.7rpx 23.7rpx 0 0;
		width: 90%;
		align-items: center;
		
	}
	.share-text {
		color: #C0C4CC;
		font-size: 18rpx;
	}
	.share-image {
		width: 40rpx;
		height: 40rpx;
	}
</style>
