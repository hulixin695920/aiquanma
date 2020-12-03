<template>
	<view class="wrap">
		<u-form ref="uForm">
			<u-form-item label="姓名">
				<u-input v-model="name" />
			</u-form-item>

			<u-form-item label-width="auto" label="手机号">
				<u-input v-model="phone" />
			</u-form-item>
			<u-button @click="submit">提交</u-button>
		</u-form>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				name: '',
				phone: '',
				userId: 0,
				canIUse: 0,
			}
		},
		onLoad() {
			var user = uni.getStorageSync('user') || {};
			if (Object.keys(user).length > 0) {
				this.userId = user.userId;
				this.canIUse = user.canIUse;
			}
			this.getBaseUser();
		},
		methods: {
			getBaseUser() {
				uni.request({
					url: this.websiteUrl + '/api/get-user-info?userId=' + this.userId,
					method: 'GET',
					data: {},
					success: res => {
						var obj = {};
						if (res.data.code === 0) {
							this.name = res.data.data.realname;
							this.phone = res.data.data.mobile_phone == 0 ? '' : res.data.data.mobile_phone;
						}
					},
					fail: () => {},
					complete: () => {}
				});
			},
			submit() {
				var user = uni.getStorageSync('user') || {};
				user.realname = this.name;
				user.mobile_phone = this.phone;
				// 新增用户 或者更新用户
				uni.request({
					url: this.websiteUrl + '/api/save-user-info',
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
						} else {
							uni.showToast({
								title: "保存成功",
							})
						}
					},
					fail: () => {},
					complete: () => {}
				});
			}
		}
	}
</script>

<style lang="scss">
	.wrap {
		padding: 40rpx;
	}
</style>
