<template>
	<view class="container">
		<!-- 顶部轮播图 -->
		<swiper class="swiper" indicator-dots="true" autoplay="true" interval="3000" duration="500" circular="true">
			<swiper-item v-for="(item, index) in bannerList" :key="index">
				<image class="swiper-image" :src="item.image" mode="aspectFill"></image>
				<view class="swiper-overlay">
					<text class="swiper-title">{{ item.title }}</text>
				</view>
			</swiper-item>
		</swiper>

		<!-- 标题区域 -->
		<view class="header-section">
			<view class="title-wrapper">
				<text class="main-title">五道营胡同</text>
				<text class="subtitle">Wudaoying Hutong</text>
			</view>
			<view class="intro-text">
				<text>北京最具文艺气息的胡同之一，汇聚了众多特色小店、精品咖啡馆和创意餐厅。</text>
			</view>
		</view>

		<!-- 底部信息 -->
		<view class="footer-section">
			<view class="info-item">
				<text class="info-label">📍</text>
				<text class="info-value">北京市东城区五道营胡同</text>
			</view>
			<view class="info-item">
				<text class="info-label">🚇</text>
				<text class="info-value">地铁2号线/5号线雍和宫站</text>
			</view>
		</view>

		<!-- H5端自定义Tabbar -->
		<!-- #ifdef H5 -->
		<custom-tabbar></custom-tabbar>
		<!-- #endif -->
	</view>
</template>

<script>
	// #ifdef H5
	import CustomTabbar from '@/components/custom-tabbar/custom-tabbar.vue'
	// #endif

	export default {
		// #ifdef H5
		components: {
			CustomTabbar
		},
		// #endif
		data() {
			return {
				bannerList: [{
						image: '/static/1.jpg',
						title: '胡同里的慢生活'
					},
					{
						image: '/static/2.jpg',
						title: '文艺与传统的碰撞'
					},
					{
						image: '/static/3.jpg',
						title: '发现城市的美好'
					}
				]
			}
		},
		onLoad() {

		},
		onShow() {
			// 触发 tabbar 更新
			// #ifdef H5
			uni.$emit('tabbar-update')
			// #endif
		},
		methods: {

		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: #f5f5f5;
		padding-bottom: 40rpx;
		/* #ifdef H5 */
		padding-bottom: calc(40rpx + 120rpx);
		/* #endif */
	}

	/* 轮播图样式 */
	.swiper {
		width: 100%;
		height: 500rpx;
		position: relative;
	}

	.swiper-image {
		width: 100%;
		height: 100%;
	}

	.swiper-overlay {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		background: linear-gradient(to top, rgba(0, 0, 0, 0.6), transparent);
		padding: 60rpx 40rpx 30rpx;
	}

	.swiper-title {
		color: #ffffff;
		font-size: 40rpx;
		font-weight: bold;
		text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
	}

	/* 标题区域 */
	.header-section {
		padding: 50rpx 40rpx 40rpx;
		background: #ffffff;
		margin-bottom: 20rpx;
	}

	.title-wrapper {
		margin-bottom: 30rpx;
	}

	.main-title {
		display: block;
		font-size: 56rpx;
		font-weight: bold;
		color: #2c3e50;
		margin-bottom: 10rpx;
		letter-spacing: 2rpx;
	}

	.subtitle {
		display: block;
		font-size: 28rpx;
		color: #95a5a6;
		letter-spacing: 1rpx;
	}

	.intro-text {
		line-height: 1.8;
	}

	.intro-text text {
		font-size: 28rpx;
		color: #5a6c7d;
		line-height: 48rpx;
	}

	/* 底部信息 */
	.footer-section {
		background: #ffffff;
		padding: 40rpx;
	}

	.info-item {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
		padding: 20rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.info-item:last-child {
		margin-bottom: 0;
		border-bottom: none;
	}

	.info-label {
		font-size: 32rpx;
		margin-right: 20rpx;
		flex-shrink: 0;
	}

	.info-value {
		font-size: 28rpx;
		color: #2c3e50;
		flex: 1;
		line-height: 1.6;
	}
</style>