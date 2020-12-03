<template>
	<view class="content">
		<u-toast ref="uToast" />
		<u-input v-model="value" :type="type" :border="border" :clearable="false" :disabled="false" :height="height"
		 :auto-height="autoHeight" />
		<!-- <u-upload ref="uUpload" :file-list="fileList" :max-count="limitNum" :action="action" :auto-upload="true" @on-success="onUploaded"></u-upload> -->
		<view class="button-click" @click="addFeedBack"><text class="button-text">提交</text></view>

		<button open-type="contact" class="contact-button" @click="contactShow = false">
			<image src="../../static/images/user/kefu.png" style="width: 29rpx; height: 29rpx;"></image>
			<text class="contact-button-text">点击此处联系客服</text>
		</button>
	</view>
</template>

<script>
	export default {
		comments() {},
		data() {
			return {
				value: '',
				type: 'textarea',
				border: true,
				height: 600,
				autoHeight: true,
				userId: 0,
				limitNum: 4,
				openId: '',
				fileList: [],
				name: '',
				showUploadList: true,
				header: {},
				img: [],
				formData: {
					userId: this.userId
				},
			};
		},
		onLoad() {
			var user = uni.getStorageSync('user') || {};
			if (Object.keys(user).length > 0) {
				this.userId = user.userId;
				this.openId = user.openid;
			}
		},
		methods: {
			onUploaded(lists) {
				this.img.push(lists);
			},
			addFeedBack() {
				var data = {};
				if (this.$u.trim(this.value) === '') {
					this.$refs.uToast.show({
						title: '内容不能为空',
						type: 'error'
					});
					return;
				}
				data.userId = this.userId;
				data.openId = this.openId;
				data.value = this.value;
				data.image = JSON.stringify(this.img);
				uni.showLoading({
						title: '加载中...'
					}),
					uni.request({
						url: this.websiteUrl + '/api/add-feedback',
						method: 'POST',
						data: data,
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						success: res => {
							var result = res.data;
							if (result.code === 0) {
								this.$refs.uToast.show({
									title: '谢谢您的反馈，我们会努力改进',
									type: 'success'
								});
								uni.hideLoading();
								uni.reLaunch({
									url: '/pages/user/user'
								})
							} else {
								uni.hideLoading();
								this.$refs.uToast.show({
									title: result.msg,
									type: 'error'
								});
								return;
							}
						},
						fail: () => {
							this.$refs.uToast.show({
								title: '额。。网络貌似开小差了',
								type: 'error'
							});
							uni.hideLoading();
						},
						complete: () => {
							uni.hideLoading();
						}
					});
			},
			goToUserIndex() {
				uni.redirectTo({
					url: '/pages/user/user'
				});
			}
		}
	};
</script>

<style lang="scss" scoped>
	.content {}

	.content-text {
		display: flex;
		width: 96%;
		margin: 0 auto;
		min-height: 600rpx;
		flex-flow: row nowrap;
		align-items: center;
		background: #ffffff;
		border-radius: 12rpx;
	}

	.button-click {
		display: flex;
		flex-flow: row nowrap;
		align-items: center;
		justify-content: center;
		width: 327.6rpx;
		height: 91rpx;
		background: #1296db;
		border-radius: 25.5rpx;
		margin: 0 auto;
		margin-top: 60rpx;
	}

	.button-text {
		font-size: 29.1rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #FFFFFF;
	}

	button::after {
		border: none;

	}

	.contact-button {
		display: flex;
		flex-flow: row nowrap;
		justify-content: center;
		align-items: center;
		text-align: center;
		margin-top: 40rpx;
		background: #FFFFFF;
	}

	.contact-button-text {
		margin-left: 5rpx;
		font-size: 29.1rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color:#0000FF;
	}
</style>
