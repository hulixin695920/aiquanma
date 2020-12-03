<template>
	<view class="container">
		<u-toast ref="uToast" />

		<u-popup v-model="show" mode="center" border-radius="14" width="400rpx" height="400rpx">
			<view class="tips">
				<text>点击订阅之后，系统将会定时提醒你去领取优惠券</text>

				<button class="mbutton" @click="addSubscribe"><text class="mtext">{{subscibeStatus === 1 ? '取消订阅' : '点击订阅'}}</text></button>
			</view>
		</u-popup>
	
		
		<view class="user-section">
			<view class="user-info-box">
				<view class="portrait-box">
					<image class="portrait" :src="avatarUrl || '/static/images/user/user-img.png'"></image>
				</view>
				<view class="info-box">
					<view class="user-title">
						<text v-if="canIUse" class="username">{{ nickName }}</text>
						<button v-else lang="zh_CN" :disabled="disabled" style="border: none;background: #FFFFFF;" open-type="getUserInfo"
						 @getuserinfo="bindGetUserInfo" class="username">
							点击登录
						</button>
					</view>
				</view>
			</view>
		</view>

		<view class="menu-tapbar">
			<view class="menu-item" @click="subscribleData">
				<image src="/static/images/user/feedback.png" class="menu-image"></image>
				<text class="grid-text">订阅通知</text>
			</view>

			<view class="menu-item" @click="goToAuthor">
				<image src="/static/images/user/call.png" class="menu-image"></image>
				<text class="grid-text">联系作者</text>
			</view>
			<view class="menu-item" @click="feedBack">
				<image src="/static/images/user/contact.png" class="menu-image"></image>
				<text class="grid-text">问题反馈</text>
			</view>

			<view style="clear: both;"></view>
		</view>

		<u-cell-group>
			<u-cell-item icon="setting-fill" title="信息完善" @click="goToUserDatail"></u-cell-item>
			<u-cell-item icon="thumb-up-fill" title="短视频去水印" @click="goToRemoveWater"></u-cell-item>
			<u-cell-item icon="integral-fill" title="更多功能" value="敬请期待"></u-cell-item>
		</u-cell-group>
	</view>
</template>

<script>
	export default {
		components: {},
		data() {
			return {
				show: false,
				mask: false,
				subscibeStatus: 0,
				coverTransform: 'translateY(0px)',
				coverTransition: '0s',
				moving: false,
				canIUse: 0,
				avatarUrl: '',
				nickName: '',
				baseInfo: {},
				userId: 0,
				opendId: '',
				loadStatus: 'nomore',
				isEnd: 0,
				page: 1,
				total: 0,
				pageSize: this.pageNum,
				iconType: 'flower',
				state: 2,
				loadText: {
					loadmore: '轻轻上拉',
					loading: '努力加载中',
					nomore: '到底了'
				},
				current: 0,
				disabled: false,
				userType: 0,
			};
		},
		onShow() {
			this.getBaseUser();
		},
		onLoad() {
			this.orderList = [];
			var user = uni.getStorageSync('user') || {};
			if (Object.keys(user).length > 0) {
				this.userId = user.userId;
				this.canIUse = user.canIUse;
				this.nickName = user.nickName;
				this.avatarUrl = user.avatarUrl;
			}
			// if (!this.canIUse) {
			// 	uni.redirectTo({
			// 		url: "/pages/user/login"
			// 	})
			// }
		},
		methods: {
			showContact() {
				this.contactShow = true;
			},
			getBaseUser() {
				this.mask = true;
				uni.request({
					url: this.websiteUrl + '/api/get-user-info?userId=' + this.userId,
					method: 'GET',
					data: {},
					success: res => {
						var obj = {};
						if (res.data.code === 0) {
							this.isMember = res.data.data.is_member;
							this.userType = res.data.data.user_type;
							this.subscibeStatus = res.data.data.msg_subscible;
						}
						this.mask = false;
					},
					fail: () => {},
					complete: () => {}
				});
			},
			getBaseInfo() {
				var user = uni.getStorageSync('user') || {};
				if (user.nickName === '' || typeof user.nickName === 'undefined') {
					wx.getSetting({
						success(res) {
							if (res.authSetting['scope.userInfo']) {
								// 已经授权，可以直接调用 getUserInfo 获取头像昵称
								wx.getUserInfo({
									success: function(res) {
										console.log(res.userInfo, 11111);
									}
								});
							}
						}
					});
				}
			},
			bindGetUserInfo(e) {
				var that = this;
				this.disabled = true;
				var user = uni.getStorageSync('user') || {};
				var userInfo = e.detail.userInfo;
				user.avatarUrl = userInfo.avatarUrl;
				user.city = userInfo.city;
				user.country = userInfo.country;
				user.province = userInfo.province;
				user.gender = userInfo.gender;
				user.nickName = userInfo.nickName;
				user.canIUse = 1;
				this.avatarUrl = userInfo.avatarUrl;
				this.canIUse = 1;
				this.nickName = userInfo.nickName;
				uni.setStorageSync('user', user);

				// 新增用户 或者更新用户
				uni.request({
					url: this.websiteUrl + '/api/save-user-info',
					method: 'POST',
					data: user,
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: res => {
						that.disabled = false;
					},
					fail: () => {},
					complete: () => {}
				});
			},
			goToAuthor(index) {
				uni.navigateTo({
					url: '/pages/me/me'
				});
			},
			feedBack() {
				uni.navigateTo({
					url: '/pages/feedback/feedback'
				});
			},
			goToUserDatail() {
				uni.navigateTo({
					url: '/pages/user/detail?userId=' + this.userId
				})
			},
			goToRemoveWater(){
				uni.navigateTo({
					url:"/pages/tools/water_remove"
				})
			},
			subscribleData() {
				this.show = true;
			},
			addSubscribe() {
				var user = uni.getStorageSync('user') || {};
				user.userId = this.userId;
				user.status = this.subscibeStatus === 1 ? 0 : 1;
				// 新增用户 或者更新用户
				uni.request({
					url: this.websiteUrl + '/api/add-subscribe-log',
					method: 'POST',
					data: user,
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: res => {
						const data = res.data;
						if (data.code !== 0) {
							uni.showToast({
								title: data.msg,
							})
							this.show = false;
						} else {
							this.subscibeStatus = user.status;
							this.show = false;
							if (user.status == 1) {
								uni.requestSubscribeMessage({
									tmplIds: [
										'qJohCVdOyxpFMzLUe8qOQpK9lglXZcnxLmpXSo4Hs3s',
									],
									success(res) {
										uni.showToast({
											title:'操作成功',
										})
									},
									fail(res) {
										uni.reLaunch({
											url: '/pages/user/user'
										});
									}
								});
							} else {
								uni.showToast({
									title:'操作成功',
								})
							}
							
						}
					},
					fail: () => {},
					complete: () => {}
				});
			}
		},
		computed: {}
	};
	customStyle: {
		color: 'red';
	}
</script>

<style lang="scss">
	.order-content {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		margin-bottom: 14.6rpx;
	}

	%flex-center {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	%section {
		display: flex;
		justify-content: space-around;
		align-content: center;
		background: #fff;
		border-radius: 10rpx;
	}

	.user-section {
		padding: 20rpx 30rpx 0;
		position: relative;

		.bg {
			position: absolute;
			left: 0;
			top: 0;
			width: 100%;
			height: 100%;
			filter: blur(1px);
			opacity: 0.7;
		}
	}

	.member-bg {
		display: flex;
	}

	.my-vip {
		width: 702rpx;
		height: 158rpx;
		margin: 0 auto;
		position: relative;
	}

	.card {
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-start;
		position: absolute;
		margin-top: 20rpx;
		margin-left: 75rpx;
		align-items: center;
	}

	.charge {
		display: flex;
		flex-flow: row nowrap;
		justify-content: flex-end;
		position: absolute;
		color: #ffe8a5;
		border: 1px solid #a19779;
		border-radius: 20rpx;
		text-align: center;
		margin: 70rpx;
		width: 109rpx;
		margin-top: 80rpx;
		margin-left: 564rpx;
	}

	.tip1 {
		font-size: 25rpx;
		font-weight: 500;
		color: #ffe8a5;
		font-family: PingFangSC-Medium, PingFang SC;
	}

	.tip4 {
		font-size: 29rpx;
		font-weight: 500;
		color: #ffe8a5;
		font-family: PingFangSC-Medium, PingFang SC;
		line-height: 33rpx;
	}

	.tip2 {
		font-size: 18rpx;
		line-height: 35rpx;
		padding-left: 220rpx;
		font-weight: 400;
		padding-right: 10rpx;
		color: #F8E8B6;
		font-family: PingFangSC-Medium, PingFang SC;
	}

	.tip3 {
		font-size: 20rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #ffe8a5;
		line-height: 36rpx;
		padding-right: 15rpx;
	}

	.user-info-box {
		height: 180rpx;
		display: flex;
		align-items: center;
		position: relative;
		z-index: 1;

		.portrait {
			width: 105.6rpx;
			height: 105.6rpx;
			border-radius: 50%;
		}

		.username {
			font-size: 30rpx;
			color: #000000;
			margin-left: 20rpx;
		}

		.desc {
			font-size: 25rpx;
			color: #cccccc;
			margin-left: 20rpx;
			padding-top: 20rpx;
		}
	}

	.info-box {
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: center;
		text-align: left;
	}

	.menu-tapbar {
		display: flex;
		padding: 20rpx;
		margin: 20rpx;
		border-radius: 30rpx;
		background: #f3f3f3;
		// box-shadow:  17rpx 17rpx 100rpx #e7e7e7,
		//              -17rpx -17rpx 100rpx #ffffff;
	}

	button::after {
		border: none;
	}

	.menu-image {
		width: 70rpx;
		height: 70rpx;
	}

	.menu-item {
		width: 33.33%;
		float: left;
		text-align: center;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.grid-text {
		font-size: 28rpx;
		margin-top: 20rpx;
		color: $u-type-info;
	}

	.goods-item {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;
		background: #f3f3f3;
		width: 96%;
		height: 256rpx;
		padding: 20rpx;
		border-radius: 20rpx;
		margin-top: 14.6rpx;
	}

	.tips {
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		padding: 40rpx;
	}

	.mbutton {
		margin-top: 40rpx;
		background: #1296db;
	}

	.mtext {
		color: #FFFFFF;
	}
</style>
