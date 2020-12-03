<template>
	<view class="wrap">
		<view>
			<image :src="src" style="width: 100%;height: 1000rpx;"></image>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				src:'https://chwl.inumei.com/me.JPG',
			};
		}
	}
</script>

<style lang="scss">
.warp {
	padding: 40rpx;
}
</style>
