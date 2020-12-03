<template>
	<view class="content">
		<view class="logo">
			<image src="../../static/index/logo.png" class="logo-img"></image>
			<text class="logo-tips">
				一起探索优惠之旅
			</text>
		</view>
		<button class="login-button" lang="zh_CN" open-type="getUserInfo" @getuserinfo="bindGetUserInfo">
			<text class="login-text">微信一键登录</text>
		</button>
		<!-- <view class="text-desc">
			<text class="text-desc-1">登录即表示您已经同意</text><text class="text-desc-2" @click="goToPrivate">《用户服务协议》</text>
		</view> -->
	</view>
</template>

<script>
	export default {
		data() {
			return {

			}
		},
		methods: {
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
						uni.switchTab({
							url: "/pages/user/user"
						})
					},
					fail: () => {},
					complete: () => {}
				});
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {}

	.logo {
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		margin-top: 300rpx;
	}

	.logo-img {
		width: 162rpx;
		height: 162rpx;

	}

	.logo-tips {
		font-size: 30rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: rgba(193, 193, 193, 1);
		letter-spacing: 16rpx;
		margin-top: 48rpx;
	}

	.login-button {
		width: 548rpx;
		height: 100rpx;
		background: #1296db;
		border-radius: 64rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 0 auto;
		margin-top: 496rpx;
	}

	.login-text {
		font-size: 32rpx;
		font-family: PingFangSC-Semibold, PingFang SC;
		font-weight: 600;
		color: rgba(255, 255, 255, 1);
	}

	button::after {
		border: none;
	}

	.text-desc {
		width: 496rpx;
		height: 80rpx;
		margin: 0 auto;
		margin-top: 30rpx;
	}

	.text-desc-1 {
		font-size: 24rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #C1C1C1;
	}

	.text-desc-2 {
		font-size: 24rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #474343;
	}
</style>
