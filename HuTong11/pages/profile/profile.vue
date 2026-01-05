<template>
	<view class="container">
		<view class="content">
			<!-- 顶部用户信息 -->
			<view class="user-info">
				<image class="avatar" src="/static/avatar.png" mode="aspectFill"></image>
				<view class="user-details">
					<text class="nickname">张三</text>
					<text class="phone">138****6789</text>
				</view>
				<view class="vip-tag">VIP会员</view>
			</view>

			<!-- 个人信息卡片 -->
			<view class="info-card">
				<view class="info-item">
					<text class="label">会员等级</text>
					<text class="value">黄金会员</text>
				</view>
				<view class="info-item">
					<text class="label">积分</text>
					<text class="value">1200</text>
				</view>
				<view class="info-item">
					<text class="label">优惠券</text>
					<text class="value">3张</text>
				</view>
			</view>

			<!-- 预约信息标题 -->
			<view class="section-title">我的停车场预约</view>

			<!-- 预约列表 -->
			<view class="booking-list">
				<view class="booking-item" v-for="(item, index) in bookingList" :key="index">
					<view class="booking-left">
						<text class="park-name">{{ item.parkName }}</text>
						<text class="booking-time">{{ item.date }} {{ item.time }}</text>
						<text class="duration">时长：{{ item.duration }}</text>
					</view>
					<view class="booking-right">
						<view class="status-tag" :class="item.status">{{ item.statusText }}</view>
						<button class="action-btn" @click="handleAction(item)">{{ item.actionText }}</button>
					</view>
				</view>
			</view>

			<!-- 其他功能入口 -->
			<view class="function-list">
				<view class="function-item" @click="navToFavorites">
					<image class="func-icon" src="/static/icon-favorites.png"></image>
					<text class="func-text">我的收藏</text>
				</view>
				<view class="function-item" @click="navToReviews">
					<image class="func-icon" src="/static/icon-reviews.png"></image>
					<text class="func-text">我的评价</text>
				</view>
				<view class="function-item" @click="navToSettings">
					<image class="func-icon" src="/static/icon-settings.png"></image>
					<text class="func-text">设置</text>
				</view>
			</view>
		</view>

		<!-- H5端自定义Tabbar -->
		<!-- #ifdef H5 -->
		<custom-tabbar></custom-tabbar>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 新的已预约停车场数据
				bookingList: []
			}
		},
		onLoad() {
			// 从本地读取预约列表
			this.bookingList = uni.getStorageSync('bookingList') || []
		},
		methods: {
			handleAction(item) {
				if (item.actionText === '取消预约') {
					uni.showModal({
						title: '提示',
						content: '确定要取消该预约吗？',
						success: (res) => {
							if (res.confirm) {
								// 1. 从本地存储中删除该条记录
								let bookings = uni.getStorageSync('bookingList') || [];
								bookings = bookings.filter(b => b.id !== item.id); // 过滤掉要删除的
								uni.setStorageSync('bookingList', bookings);

								// 2. 更新页面数据
								this.bookingList = bookings;

								uni.showToast({
									title: '预约已取消',
									icon: 'success'
								});
							}
						}
					});
				} else if (item.actionText === '查看详情') {
					uni.navigateTo({
						url: `/pages/booking/detail?id=${item.id}`
					});
				} else if (item.actionText === '重新预约') {
					uni.navigateTo({
						url: `/pages/booking/booking?parkId=${item.id}`
					});
				}
			},
			navToFavorites() {
				uni.navigateTo({
					url: '/pages/profile/favorites'
				});
			},
			navToReviews() {
				uni.navigateTo({
					url: '/pages/profile/reviews'
				});
			},
			navToSettings() {
				uni.navigateTo({
					url: '/pages/profile/settings'
				});
			}
		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: #f5f5f5;
		padding: 40rpx;
		/* #ifdef H5 */
		padding-bottom: calc(40rpx + 120rpx);
		/* #endif */
	}

	.content {
		display: flex;
		flex-direction: column;
	}

	/* 用户信息 */
	.user-info {
		display: flex;
		align-items: center;
		padding: 30rpx;
		background-color: #fff;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
	}

	.avatar {
		width: 120rpx;
		height: 120rpx;
		border-radius: 60rpx;
		margin-right: 30rpx;
	}

	.user-details {
		flex: 1;
	}

	.nickname {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.phone {
		font-size: 26rpx;
		color: #999;
		margin-top: 10rpx;
	}

	.vip-tag {
		background-color: #ffd700;
		color: #333;
		padding: 10rpx 20rpx;
		border-radius: 20rpx;
		font-size: 24rpx;
	}

	/* 信息卡片 */
	.info-card {
		display: flex;
		justify-content: space-around;
		background-color: #fff;
		padding: 30rpx 0;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
	}

	.info-item {
		text-align: center;
	}

	.label {
		font-size: 24rpx;
		color: #999;
	}

	.value {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-top: 10rpx;
	}

	/* 预约标题 */
	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		padding: 20rpx 0;
	}

	/* 预约列表 */
	.booking-list {
		background-color: #fff;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
	}

	.booking-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 30rpx;
		border-bottom: 1px solid #eee;
	}

	.park-name {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.booking-time,
	.duration {
		font-size: 26rpx;
		color: #999;
		margin-top: 10rpx;
	}

	.status-tag {
		padding: 10rpx 20rpx;
		border-radius: 20rpx;
		font-size: 24rpx;
		margin-bottom: 20rpx;
	}

	.status-tag.pending {
		background-color: #4caf50;
		color: #fff;
	}

	.status-tag.completed {
		background-color: #2196f3;
		color: #fff;
	}

	.status-tag.cancelled {
		background-color: #f44336;
		color: #fff;
	}

	.action-btn {
		background-color: #007aff;
		color: #fff;
		border: none;
		border-radius: 8rpx;
		padding: 12rpx 24rpx;
		font-size: 26rpx;
	}

	/* 功能入口 */
	.function-list {
		display: flex;
		justify-content: space-around;
		background-color: #fff;
		padding: 30rpx 0;
		border-radius: 12rpx;
	}

	.function-item {
		text-align: center;
	}

	.func-icon {
		width: 60rpx;
		height: 60rpx;
	}

	.func-text {
		font-size: 26rpx;
		color: #333;
		margin-top: 10rpx;
	}
</style>