<script>
export default {
	onLaunch: function() {
		// #ifdef MP-WEIXIN
		var userInfo = uni.getStorageSync('user') || {};
		var arr = Object.keys(userInfo);
		if (arr.length === 0 || !userInfo.userId) {
			// this.userLogin();
		var that = this;
			uni.login({
				provider: 'weixin',
				success: function(res) {
					if (res.code) {
						// 获取openId
						uni.request({
							url: that.websiteUrl + '/api/get-wechat-user-info?code=' + res.code,
							method: 'GET',
							data: {},
							success: res => {
								var obj = {};
								obj.openid = res.data.data.openid;
								obj.session_key = res.data.data.session_key;
								// 用户管理
								uni.request({
									url: that.websiteUrl + '/api/save-user-info',
									method: 'POST',
									data: {
										openid: res.data.data.openid
									},
									header: {
										'content-type': 'application/x-www-form-urlencoded'
									},
									success: result => {
										if (result.data.code === 0) {
											obj.userId = result.data.data;
											uni.setStorageSync('user', obj);
										} else {
											this.$refs.uToast.show({
												title: '额。。网络貌似开小差了',
												type: 'error'
											});
										}
									},
									fail: () => {},
									complete: () => {}
								});
							},
							fail: () => {},
							complete: () => {}
						});
					} else {
						console.log('获取用户登录态失败！' + res.errMsg);
					}
				}
			});
		} else {
			uni.checkSession({
				success(res) {},
				fail() {
					uni.login({
						provider: 'weixin',
						success: function(res) {
							if (res.code) {
								// 获取openId
								uni.request({
									url: that.websiteUrl + '/api/get-wechat-user-info?code=' + res.code,
									method: 'GET',
									data: {},
									success: res => {
										var obj = {};
										obj.openid = res.data.data.openid;
										obj.session_key = res.data.data.session_key;
										// 用户管理
										uni.request({
											url: that.websiteUrl + '/api/save-user-info',
											method: 'POST',
											data: {
												openid: res.data.data.openid
											},
											header: {
												'content-type': 'application/x-www-form-urlencoded'
											},
											success: result => {
												if (result.data.code === 0) {
													obj.userId = result.data.data;
													uni.setStorageSync('user', obj);
												} else {
													this.$refs.uToast.show({
														title: '额。。网络貌似开小差了',
														type: 'error'
													});
												}
											},
											fail: () => {},
											complete: () => {}
										});
									},
									fail: () => {},
									complete: () => {}
								});
							} else {
								console.log('获取用户登录态失败！' + res.errMsg);
							}
						}
					});
				}
			});
		}
		// var that = this;
		// var user = uni.getStorageSync('user') || {};
		// var userInfo = uni.getStorageSync('userInfo') || {};
		// var userId = uni.getStorageSync('userId') || {};
		// var openId = '';
		// if (!user.openid || !userId.userId) {
		// 	console.log(user.userId)
		// 	// 登录
			
		// }
		// if (!userInfo.nickName) {
		// 	var user = uni.getStorageSync('user') || {};
		// 	// 优先取数据库数据
		// 	uni.showLoading({
		// 		title: '加载中...'
		// 	}),
		// 		uni.request({
		// 			url: that.websiteUrl + '/api/get-user-info?openid=' + user.openid,
		// 			method: 'GET',
		// 			data: {},
		// 			success: res => {
		// 				var obj = {};
		// 				if (res.data.data.code === 0) {
		// 					if (res.data.data.nickname !== '' && res.data.data.avatar !== '') {
		// 						baseInfo.avatarUrl = res.data.data.avatar;
		// 						baseInfo.nickName = res.data.data.nickname;
		// 						baseInfo.gender = res.data.data.gender;
		// 						baseInfo.country = res.data.data.country;
		// 						baseInfo.province = res.data.data.province;
		// 						baseInfo.city = res.data.data.city;
		// 						baseInfo.openid = user.openid;
		// 						console.log(baseInfo, '接口请求获取用户信息');
		// 						uni.setStorageSync('userInfo', baseInfo); //存储userInfo
		// 					} else {
		// 						uni.getSetting({
		// 							success(res) {
		// 								if (res.authSetting['scope.userInfo']) {
		// 									uni.getUserInfo({
		// 										lang: 'zh_CN',
		// 										success: function(res) {
		// 											baseInfo.avatarUrl = res.userInfo.avatarUrl;
		// 											baseInfo.nickName = res.userInfo.nickName;
		// 											baseInfo.gender = res.userInfo.gender;
		// 											baseInfo.country = res.userInfo.country;
		// 											baseInfo.province = res.userInfo.province;
		// 											baseInfo.city = res.userInfo.city;
		// 											baseInfo.openid = user.openid;
		// 											uni.setStorageSync('userInfo', baseInfo); //存储userInfo

		// 											// 新增用户 或者更新用户
		// 											uni.request({
		// 												url: that.websiteUrl + '/api/save-user-info',
		// 												method: 'POST',
		// 												data: baseInfo,
		// 												header: {
		// 													'content-type': 'application/x-www-form-urlencoded'
		// 												},
		// 												success: res => {
		// 													console.log(res);
		// 												},
		// 												fail: () => {},
		// 												complete: () => {}
		// 											});
		// 										},
		// 										fail: () => {}
		// 									});
		// 								}
		// 							}
		// 						});
		// 					}
		// 				}
		// 				uni.hideLoading();
		// 			},
		// 			fail: () => {},
		// 			complete: () => {}
		// 		});
		// }

		//获取用户当前位置
		// uni.getSetting({
		// 	success(res) {
		// 		console.log(res,'tttttt')
		// 		if (!res.authSetting['scope.userLocation']) {
		// 			uni.authorize({
		// 				scope: 'scope.userLocation',
		// 				success() {
		// 					console.log(res,'uyyyyyy')
		// 					uni.getLocation({
		// 						success(res) {
		// 							var location = {};
		// 							location.longitude = res.longitude;
		// 							location.latitude = res.latitude;
		// 							uni.setStorageSync('userLocation', location); //存储位置信息
		// 							console.log('当前位置的经度：' + res.longitude);
		// 							console.log('当前位置的纬度：' + res.latitude);
		// 						}
		// 					});
		// 				},fail() {
		// 					console.log(res,'eeeeeee')
		// 				}
		// 			});
		// 		} else {
		// 			uni.getLocation({
		// 				success(res) {
		// 					var location = {};
		// 					location.longitude = res.longitude;
		// 					location.latitude = res.latitude;
		// 					uni.setStorageSync('userLocation', location); //存储位置信息
		// 					console.log('当前位置的经度：' + res.longitude);
		// 					console.log('当前位置的纬度：' + res.latitude);
		// 				}
		// 			});
		// 		}
		// 	}
		// });

		// #endif
		console.log('App Launch');
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		console.log('App Hide');
	}
};
</script>

<style lang="scss">
@import 'uview-ui/index.scss';
/*每个页面公共css */
/* H5的时候，隐藏滚动条 */
/* #ifdef H5 */
::-webkit-scrollbar {
	display: none;
	width: 0 !important;
	height: 0 !important;
	-webkit-appearance: none;
	background: transparent;
}
/* #endif */
</style>
