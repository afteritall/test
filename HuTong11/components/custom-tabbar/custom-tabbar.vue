<template>
	<view class="custom-tabbar" :class="{ 'h5-tabbar': isH5 }">
		<view 
			class="tabbar-item" 
			v-for="(item, index) in tabList" 
			:key="index"
			:class="{ 'active': currentIndex === index }"
			@click="switchTab(item, index)"
		>
			<view class="tabbar-icon">
				<text>{{ currentIndex === index ? item.selectedIcon : item.icon }}</text>
			</view>
			<text class="tabbar-text">{{ item.text }}</text>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'CustomTabbar',
		data() {
			return {
				currentIndex: 0,
				checkTimer: null,
				tabList: [
					{
						pagePath: '/pages/index/index',
						text: '首页',
						icon: '🏠',
						selectedIcon: '🏡'
					},
					{
						pagePath: '/pages/scenic/scenic',
						text: '景点',
						icon: '📍',
						selectedIcon: '🎯'
					},
					{
						pagePath: '/pages/restaurant/restaurant',
						text: '餐厅',
						icon: '🍽️',
						selectedIcon: '🍴'
					},
					{
						pagePath: '/pages/parking/parking',
						text: '停车场',
						icon: '🅿️',
						selectedIcon: '🚗'
					},
					{
						pagePath: '/pages/ai/ai',
						text: 'AI助手',
						icon: '🤖',
						selectedIcon: '💬'
					},
					{
						pagePath: '/pages/profile/profile',
						text: '个人中心',
						icon: '👤',
						selectedIcon: '👨'
					}
				]
			}
		},
		computed: {
			isH5() {
				// #ifdef H5
				return true
				// #endif
				// #ifndef H5
				return false
				// #endif
			}
		},
		mounted() {
			this.setCurrentIndex()
			// 监听页面显示事件
			uni.$on('tabbar-update', () => {
				this.setCurrentIndex()
			})
			// 定时检查当前页面（作为备用方案）
			this.checkTimer = setInterval(() => {
				this.setCurrentIndex()
			}, 500)
		},
		updated() {
			// 组件更新时重新计算当前索引
			this.$nextTick(() => {
				this.setCurrentIndex()
			})
		},
		beforeDestroy() {
			// 清除定时器
			if (this.checkTimer) {
				clearInterval(this.checkTimer)
				this.checkTimer = null
			}
			// 移除事件监听
			uni.$off('tabbar-update')
		},
		methods: {
			setCurrentIndex() {
				try {
					const pages = getCurrentPages()
					if (!pages || pages.length === 0) {
						return
					}
					const currentPage = pages[pages.length - 1]
					if (!currentPage || !currentPage.route) {
						return
					}
					// 标准化路径格式
					let currentRoute = currentPage.route
					if (!currentRoute.startsWith('/')) {
						currentRoute = '/' + currentRoute
					}
					// 移除可能的查询参数
					currentRoute = currentRoute.split('?')[0]
					
					// 查找匹配的 tab
					const index = this.tabList.findIndex(item => {
						const itemPath = item.pagePath.split('?')[0]
						return itemPath === currentRoute
					})
					
					if (index !== -1 && this.currentIndex !== index) {
						this.currentIndex = index
					}
				} catch (e) {
					console.error('设置当前索引失败:', e)
				}
			},
			switchTab(item, index) {
				if (this.currentIndex === index) {
					return
				}
				
				// 先更新索引，提供即时反馈
				this.currentIndex = index
				
				// H5端使用 reLaunch 模拟 switchTab 行为，App端和小程序端使用 switchTab
				// #ifdef H5
				uni.reLaunch({
					url: item.pagePath,
					success: () => {
						// 页面切换成功后延迟确认索引，确保页面已加载
						setTimeout(() => {
							this.setCurrentIndex()
							uni.$emit('tabbar-update')
						}, 100)
					},
					fail: () => {
						// 如果失败，恢复之前的索引
						this.setCurrentIndex()
					}
				})
				// #endif
				// #ifndef H5
				uni.switchTab({
					url: item.pagePath,
					success: () => {
						// 页面切换成功后延迟确认索引
						setTimeout(() => {
							this.setCurrentIndex()
						}, 100)
					},
					fail: () => {
						// 如果失败，恢复之前的索引
						this.setCurrentIndex()
					}
				})
				// #endif
			}
		}
	}
</script>

<style scoped>
	.custom-tabbar {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		display: flex;
		align-items: center;
		justify-content: space-around;
		background: #ffffff;
		border-top: 1rpx solid #e5e5e5;
		padding: 10rpx 0;
		padding-bottom: calc(10rpx + env(safe-area-inset-bottom));
		z-index: 999;
		box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.05);
	}

	/* H5 端适配 */
	.h5-tabbar {
		padding-bottom: 10rpx;
	}

	.tabbar-item {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 10rpx 0;
		transition: all 0.3s;
	}

	.tabbar-icon {
		font-size: 44rpx;
		margin-bottom: 6rpx;
		line-height: 1;
		transition: transform 0.3s;
	}

	.tabbar-item.active .tabbar-icon {
		transform: scale(1.1);
	}

	.tabbar-text {
		font-size: 22rpx;
		color: #7f8c8d;
		transition: color 0.3s;
	}

	.tabbar-item.active .tabbar-text {
		color: #3498db;
		font-weight: 500;
	}

	/* 响应式设计 - 6个tab时缩小字体 */
	@media screen and (max-width: 375px) {
		.tabbar-text {
			font-size: 20rpx;
		}
		
		.tabbar-icon {
			font-size: 40rpx;
		}
	}
</style>
