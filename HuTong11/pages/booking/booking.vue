<template>
	<view class="container">
		<view class="parking-info">
			<text class="parking-name">{{ parking.name }}</text>
			<text class="parking-spots">剩余车位：{{ parking.spots }} 个</text>
			<text class="parking-price">价格：{{ parking.price }} 元/小时</text>
		</view>

		<view class="form">
			<view class="form-item">
				<label>预约日期：</label>
				<picker mode="date" :value="date" @change="onDateChange">
					<view class="picker-text">{{ date }}</view>
				</picker>
			</view>

			<view class="form-item">
				<label>预约时间：</label>
				<picker mode="time" :value="time" @change="onTimeChange">
					<view class="picker-text">{{ time }}</view>
				</picker>
			</view>

			<view class="form-item">
				<label>停车时长：</label>
				<picker mode="selector" :range="durationOptions" :value="durationIndex" @change="onDurationChange">
					<view class="picker-text">{{ durationOptions[durationIndex] }}</view>
				</picker>
			</view>

			<button class="submit-btn" @click="submitBooking">提交预约</button>
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
				parking: {
					id: 0,
					name: '',
					spots: 0,
					price: 0
				},
				date: '',
				time: '',
				durationIndex: 0,
				durationOptions: ['1小时', '2小时', '3小时', '4小时', '5小时', '6小时']
			}
		},
		onLoad(options) {
			const parkId = Number(options.id)

			const mockParkingList = [{
					id: 1,
					name: '五道营胡同停车场',
					spots: 15,
					price: 10
				},
				{
					id: 2,
					name: '雍和宫停车场',
					spots: 8,
					price: 12
				},
				{
					id: 3,
					name: '国子监停车场',
					spots: 20,
					price: 8
				}
			]

			const park = mockParkingList.find(p => p.id === parkId)
			if (park) {
				this.parking = park
			} else {
				uni.showToast({
					title: '未找到该停车场',
					icon: 'none'
				})
				uni.navigateBack()
			}

			const today = new Date()
			this.date = today.getFullYear() + '-' + String(today.getMonth() + 1).padStart(2, '0') + '-' + String(today
				.getDate()).padStart(2, '0')
			this.time = String(today.getHours()).padStart(2, '0') + ':' + String(today.getMinutes()).padStart(2, '0')
		},
		methods: {
			onDateChange(e) {
				this.date = e.detail.value
			},
			onTimeChange(e) {
				this.time = e.detail.value
			},
			onDurationChange(e) {
				this.durationIndex = e.detail.value
			},
			formatDateTime(dateStr, timeStr) {
			    const [h, m] = timeStr.split(':').map(Number);
			    const period = h < 12 ? '上午' : '下午';
			    const twelve = h > 12 ? h - 12 : (h === 0 ? 12 : h);
			    const sec = new Date().getSeconds();
			
			    const [y, M, d] = dateStr.split('-');
			    return `${y}/${Number(M)}/${Number(d)} ${period}${twelve}:${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
			},
			
			    submitBooking() {
			        if (!this.date || !this.time) {
			            uni.showToast({ title: '请选择日期和时间', icon: 'none' });
			            return;
			        }
			        const dateTime = this.formatDateTime(this.date, this.time); // ← 用这里
			
			        const newBooking = {
			            id: Date.now(),
			            type: 'parking',
			            shopName: this.parking.name,
			            date: dateTime,              // 2026/1/6 上午10:58:19
			            duration: this.durationOptions[this.durationIndex],
			            isCancel: false
			        };
			
			        let bookings = uni.getStorageSync('BOOKINGS') || [];
			        bookings.unshift(newBooking);
			        uni.setStorageSync('BOOKINGS', bookings);
			
			        uni.showToast({ title: '预约成功', icon: 'success' });
			        setTimeout(() => uni.navigateBack(), 1500);
			    }
		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: #f5f5f5;
		padding: 40rpx;
	}

	.parking-info {
		background: #fff;
		padding: 30rpx;
		border-radius: 12rpx;
		margin-bottom: 30rpx;
	}

	.parking-name {
		font-size: 36rpx;
		font-weight: bold;
		color: #2c3e50;
	}

	.parking-spots,
	.parking-price {
		font-size: 28rpx;
		color: #7f8c8d;
		margin-top: 10rpx;
	}

	.form {
		background: #fff;
		padding: 30rpx;
		border-radius: 12rpx;
	}

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	label {
		font-size: 28rpx;
		color: #2c3e50;
		width: 140rpx;
	}

	.picker-text {
		flex: 1;
		font-size: 28rpx;
		color: #7f8c8d;
		border-bottom: 1px solid #ddd;
		padding-bottom: 10rpx;
	}

	.submit-btn {
		background: #007aff;
		color: #fff;
		border: none;
		border-radius: 8rpx;
		padding: 20rpx;
		font-size: 32rpx;
		margin-top: 30rpx;
	}
</style>