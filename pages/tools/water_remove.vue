<template>
	<view class="water-url">
		<u-toast ref="uToast" />
		<view class="title-content">
			<text class="title">
				支持各大短视频解析，例如西瓜视频，抖音短视频，快手短视频等等，复制链接解析即可！
			</text>
		</view>

		<u-input v-model="value" :type="type" :border="border" :height="height" :auto-height="autoHeight" />

		<view class="button-click" @click="explainUrl"><text class="button-text">提交</text></view>

		<view v-if="isSuccess === 1" class="copy-data">
			<view class="url-info">
				<text>{{videoTitle}}</text>
				<view class="tel-button" >
					<u-input v-model="url" :type="type2" :disabled="disabled" :border="border" />
					<text class="tel-button-text" @click="copyData">复制</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				value: '',
				type: 'textarea',
				height: 400,
				border: true,
				autoHeight: true,
				isSuccess: 0,
				url: '',
				videoTitle: '',
				type2:'input',
				disabled:true,
			}
		},
		methods: {
			explainUrl() {
				var data = {};
				if (this.$u.trim(this.value) === '') {
					this.$refs.uToast.show({
						title: '内容不能为空',
						type: 'error'
					});
					return;
				}
				uni.showLoading({
						title: '解析中...'
					}),
					data.url = this.value;
				uni.request({
					url: this.websiteUrl + '/api/url-explain',
					method: 'POST',
					data: data,
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: res => {
						uni.hideLoading();
						var result = res.data;
						if (result.code === 0) {
							this.isSuccess = 1;
							if (result.data.video_url == '') {
								this.$refs.uToast.show({
									title: '暂不支持，努力添加中',
									type: 'error'
								});
								return;
							}
							this.videoTitle = result.data.desc;
							this.url = result.data.video_url;


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
			copyData() {
				uni.setClipboardData({
					data: this.url
				})
			}
		}
	}
</script>

<style lang="scss">
	.water-url {
		padding: 40rpx;
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

	.title-content {
		display: flex;
		margin-bottom: 20rpx;
	}

	.title {
		font-size: 29.1rpx;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #606266;
	}

	.url-info {
		display: flex;
		align-items: center;
		flex-direction: column;
		justify-content: center;

	}
	.copy-data {
		margin-top: 40rpx;
	}
	.tel-button {
		margin-top: 40rpx;
		display: flex;
		flex-wrap: row nowrap;
		justify-content: space-between;
		align-items: center;
	}
	.tel-button-text {
		margin-left: 5rpx;
	}
</style>
