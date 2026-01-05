<template>
	<view class="container">
		<!-- 搜索框 -->
		<view class="search-box">
			<input class="search-input" placeholder="请输入目的地" v-model="destination" />
			<button class="search-btn" @click="searchRoute">搜索</button>
		</view>

		<!-- 地图容器 -->
		<div id="container" class="map-container"></div>

		<!-- 停车场列表 -->
		<view class="section-title">停车场</view>
		<view class="parking-list">
			<view class="parking-item" v-for="(park, index) in parkingList" :key="index">
				<view class="parking-info">
					<text class="parking-name">{{ park.name }}</text>
					<text class="parking-spots">剩余车位：{{ park.spots }} 个</text>
					<text class="parking-price">价格：{{ park.price }} 元/小时</text>
				</view>
				<button class="book-btn" @click="navigateToBooking(park.id)">预约</button>
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
				parkingList: [{
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
				],
				destination: '',
				map: null,
				polyline: null, // 保存路线实例
				startLat: 0,
				startLng: 0,
				key: '514d4dab0037d841b4f5d510a925262d'
			}
		},
		mounted() {
			const script = document.createElement('script');
			script.src = 'https://webapi.amap.com/maps?v=2.0&key=cf6a998bf2739da481e57bef429dcbd1';
			script.onload = () => {
				this.getLocationByName('五道营胡同');
			};
			script.onerror = (err) => {
				console.error('高德地图 API 加载失败', err);
			};
			document.head.appendChild(script);
		},
		methods: {
			getLocationByName(name) {
				const url = `https://restapi.amap.com/v3/geocode/geo?address=${encodeURIComponent(name)}&key=${this.key}`;

				fetch(url)
					.then(res => res.json())
					.then(data => {
						if (data.status === '1' && data.geocodes.length > 0) {
							const [lng, lat] = data.geocodes[0].location.split(',');
							this.startLat = Number(lat);
							this.startLng = Number(lng);
							this.initMap(this.startLat, this.startLng);
						} else {
							console.error('未找到该地点坐标', data);
						}
					})
					.catch(err => {
						console.error('获取坐标失败', err);
					});
			},
			initMap(lat, lng) {
				this.map = new AMap.Map('container', {
					resizeEnable: true,
					zoom: 16,
					center: [lng, lat]
				});
				new AMap.Marker({
					position: [lng, lat],
					map: this.map
				});
			},
			searchRoute() {
				if (!this.destination) {
					uni.showToast({
						title: '请输入目的地',
						icon: 'none'
					});
					return;
				}

				const url =
					`https://restapi.amap.com/v3/geocode/geo?address=${encodeURIComponent(this.destination)}&key=${this.key}`;

				fetch(url)
					.then(res => res.json())
					.then(data => {
						if (data.status === '1' && data.geocodes.length > 0) {
							const [endLng, endLat] = data.geocodes[0].location.split(',');
							this.drawRoute(endLat, endLng);
						} else {
							uni.showToast({
								title: '未找到该地点',
								icon: 'none'
							});
						}
					})
					.catch(err => {
						console.error('获取目的地坐标失败', err);
					});
			},
			drawRoute(endLat, endLng) {
				// 确保地图已初始化
				if (!this.map) {
					uni.showToast({
						title: '地图未初始化',
						icon: 'none'
					});
					return;
				}

				const url =
					`https://restapi.amap.com/v3/direction/driving?origin=${this.startLng},${this.startLat}&destination=${endLng},${endLat}&key=${this.key}`;

				fetch(url)
					.then(res => res.json())
					.then(data => {
						if (data.status === '1' && data.route && data.route.paths.length > 0) {
							const path = data.route.paths[0].steps.map(step => {
								return step.polyline.split(';').map(point => point.split(',').map(Number));
							}).flat();

							// 移除旧路线
							if (this.polyline) {
								this.map.remove(this.polyline);
							}

							// 添加新路线
							this.polyline = new AMap.Polyline({
								path: path,
								borderWeight: 5,
								strokeColor: '#FF3333',
								strokeOpacity: 1,
								strokeStyle: 'solid',
								strokeDasharray: [10, 5]
							});

							this.map.add(this.polyline);
							this.map.setFitView([this.polyline]);
						} else {
							uni.showToast({
								title: '无法规划路线',
								icon: 'none'
							});
						}
					})
					.catch(err => {
						console.error('获取路线失败', err);
					});
			},
			navigateToBooking(parkId) {
				uni.navigateTo({
					url: `/pages/booking/booking?id=${parkId}`
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
		display: flex;
		flex-direction: column;
		/* #ifdef H5 */
		padding-bottom: calc(40rpx + 120rpx);
		/* #endif */
	}

	.search-box {
		display: flex;
		margin-bottom: 20rpx;
	}

	.search-input {
		flex: 1;
		height: 80rpx;
		padding: 0 20rpx;
		border: 1px solid #ddd;
		border-radius: 40rpx 0 0 40rpx;
		font-size: 28rpx;
	}

	.search-btn {
		height: 80rpx;
		width: 160rpx;
		background: #007aff;
		color: #fff;
		border: none;
		border-radius: 0 40rpx 40rpx 0;
		font-size: 28rpx;
	}

	.map-container {
		width: 100%;
		height: 300px;
		background-color: #e0e0e0;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		padding: 20rpx 0;
	}

	.parking-list {
		flex: 1;
	}

	.parking-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20rpx;
		background: #fff;
		border-radius: 12rpx;
		margin-bottom: 20rpx;
		box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
	}

	.parking-info {
		flex: 1;
	}

	.parking-name {
		font-size: 32rpx;
		font-weight: bold;
		color: #2c3e50;
	}

	.parking-spots,
	.parking-price {
		font-size: 26rpx;
		color: #7f8c8d;
	}

	.book-btn {
		background: #007aff;
		color: #fff;
		border: none;
		border-radius: 8rpx;
		padding: 12rpx 24rpx;
		font-size: 26rpx;
	}
</style>