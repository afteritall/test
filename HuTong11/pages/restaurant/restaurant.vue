<template>
<view class="allcontainer">
  <view class="container">
    <!-- 搜索框 -->
    <view class="search-section">
      <view class="search-box">
        <text class="search-icon">🔍</text>
        <input
          class="search-input"
          type="text"
          placeholder="搜索餐厅名称..."
          v-model="searchKeyword"
          @input="handleSearch"
          confirm-type="search"
        />
        <text v-if="searchKeyword" class="clear-icon" @click="clearSearch">✕</text>
      </view>
    </view>

    <!-- 餐厅列表 -->
    <view class="restaurant-list">
      <view
        v-for="(item, index) in filteredRestaurantList"
        :key="index"
        class="restaurant-card"
        @click="goToDetail(item.name)"
      >
        <!-- 卡片图片 -->
        <view class="card-image-wrapper">
          <image class="card-image" :src="item.image" mode="aspectFill"></image>

          <!-- 预约状态小标签 -->
          <view
            :class="['book-badge', bookedSet.has(item.name) ? 'booked' : 'can-book']"
          >
            <text class="book-badge-text">
              {{ bookedSet.has(item.name) ? '已预约' : '可预约' }}
            </text>
          </view>

          <view class="price-badge">
            <text class="price-badge-text">{{ item.price }}</text>
          </view>
        </view>

        <!-- 卡片内容 -->
        <view class="card-body">
          <view class="card-header">
            <text class="restaurant-name">{{ item.name }}</text>
            <view class="arrow-wrapper">
              <text class="arrow-icon">›</text>
            </view>
          </view>
          <view class="card-content">
            <text class="intro-text">{{ item.intro }}</text>
            <view class="card-footer">
              <view class="tag-wrapper">
                <text class="tag" v-if="item.tag">{{ item.tag }}</text>
              </view>
            </view>
          </view>
        </view>
      </view>

      <!-- 无搜索结果提示 -->
      <view v-if="filteredRestaurantList.length === 0" class="empty-tip">
        <text class="empty-text">未找到相关餐厅</text>
      </view>
    </view>

    <!-- H5端自定义Tabbar -->
    <!-- #ifdef H5 -->
    <custom-tabbar></custom-tabbar>
    <!-- #endif -->
  </view>
</view>
</template>

<script>
// #ifdef H5
import CustomTabbar from '@/components/custom-tabbar/custom-tabbar.vue'
// #endif

export default {
  // #ifdef H5
  components: { CustomTabbar },
  // #endif
  data() {
    return {
      searchKeyword: '',
      restaurantList: [
        {
          name: '蜜雪冰城',
          intro: '胡同西口打卡级「平价快乐水」，3 元甜筒、5 元果茶，学生游客第一站。',
          price: '¥3-15',
          tag: '饮品',
          image: 'https://images.unsplash.com/photo-1559056199-641a0ac8b55e?w=800'
        },
        {
          name: '一拙茶研所',
          intro: '老平房改成的极简茶空间，主打手冲原叶与茶拿铁，可安静办公一下午。',
          price: '¥35-80',
          tag: '茶饮',
          image: 'https://images.unsplash.com/photo-1544787219-7f47ccb76574?w=800'
        },
        {
          name: '敦敦斋',
          intro: '清真家常菜小馆，招牌"敦敦包子"一天只出 3 屉，皮薄汁多，11 点前排不上。',
          price: '¥25-50',
          tag: '中餐',
          image: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800'
        },
        {
          name: '兰·泰餐',
          intro: '胡同里最便宜的泰式小馆，冬阴功 28 元、泰式奶茶 12 元，口味改良偏甜。',
          price: '¥30-60',
          tag: '东南亚',
          image: 'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?w=800'
        },
        {
          name: '王欻欻火锅',
          intro: '一人食小火锅，锅底 9.9 元，菜量可按"半份"点，适合独自探店。',
          price: '¥40-80',
          tag: '火锅',
          image: 'https://images.unsplash.com/photo-1556911220-e15b29be8c8f?w=800'
        },
        {
          name: '站点披萨',
          intro: '18 寸纽约薄底，128 元够 4 人吃；窗边位能望雍和宫红墙，拍照出片。',
          price: '¥30-50',
          tag: '西餐',
          image: 'https://images.unsplash.com/photo-1513104890138-7c749659a591?w=800'
        },
        {
          name: '松和 MATSUWA',
          intro: '胡同唯一日料正餐，午市定食 58 元起，晚市有炙烤和牛，清酒种类多。',
          price: '¥80-200',
          tag: '日料',
          image: 'https://images.unsplash.com/photo-1579952363873-27f3bade9f55?w=800'
        },
        {
          name: '串门儿串串火锅',
          intro: '成都冷锅串串+热卤，牛肉串 1 元/签，干碟香而不辣，夜宵营业到 24:00。',
          price: '¥50-100',
          tag: '川菜',
          image: 'https://images.unsplash.com/photo-1556911220-e15b29be8c8f?w=800'
        },
        {
          name: '胡同肉饼坊',
          intro: '现烙牛肉肉饼 8 元/张，外酥里汁，加一碗小米粥是周边上班族的"隐藏套餐"。',
          price: '¥15-30',
          tag: '小吃',
          image: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800'
        },
        {
          name: '满香馄饨',
          intro: '夫妻老店，鲜肉馄饨 12 元/碗，汤底用猪骨熬，可加免费韭菜花。',
          price: '¥12-25',
          tag: '小吃',
          image: 'https://images.unsplash.com/photo-1559056199-641a0ac8b55e?w=800'
        },
        {
          name: '涮火捞石锅鱼',
          intro: '酸菜黑鱼片石锅，先喝鱼汤再涮菜，团购 88 元双人餐含饮料。',
          price: '¥50-80',
          tag: '火锅',
          image: 'https://images.unsplash.com/photo-1529692236671-f1f6cf9683ba?w=800'
        },
        {
          name: '董记粗粮',
          intro: '健康轻食路线：莜面鱼鱼、玉米面煎饼、杂粮豆浆，适合素食/控糖人群。',
          price: '¥20-40',
          tag: '轻食',
          image: 'https://images.unsplash.com/photo-1542838132-92c53300491e?w=800'
        },
		{
		  name: '野馄饨·夜酒',
		  intro: '晚上 8 点才开门的“深夜食堂”，鸡汤馄饨+自酿米酒，胡同口的小黄灯亮了 15 年。',
		  price: '¥18-35',
		  tag: '夜宵',
		  image: 'https://images.unsplash.com/photo-1528732263441-1482b1f1a845?w=800'
		},
		{
		  name: '柿子树法甜',
		  intro: '老四合院里吃法式甜品，主厨师从 Pierre Hermé，柿子蒙布朗是秋季限定。',
		  price: '¥68-120',
		  tag: '甜品',
		  image: 'https://images.unsplash.com/photo-1511381939415-e44015466834?w=800'
		},
		{
		  name: '巷尾咖喱',
		  intro: '日本人开的微辣咖喱专门店，每日只卖 80 份，加料芝士要抢。',
		  price: '¥42-68',
		  tag: '日式',
		  image: 'https://images.unsplash.com/photo-1585937421612-70a05835626b?w=800'
		},
		{
		  name: '糖渍山楂',
		  intro: '老北京炒红果+创意冰粉，山楂每天手工去核，酸甜解腻。',
		  price: '¥12-22',
		  tag: '小吃',
		  image: 'https://images.unsplash.com/photo-1599599810694-b5b37307298b?w=800'
		},
		{
		  name: 'Planet 轻食星球',
		  intro: '健身党福音：每道菜标卡路里，藜麦饭+低温鸡胸，酱热量也写出来。',
		  price: '¥38-65',
		  tag: '轻食',
		  image: 'https://images.unsplash.com/photo-1547592166-23ac45744acd?w=800'
		},
		{
		  name: '串府·小腰精',
		  intro: '主打“小腰子”烤串，羊腰提前用牛奶泡，不腥不腻，夜宵王。',
		  price: '¥3-10/串',
		  tag: '烧烤',
		  image: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800'
		},
		{
		  name: '妈妈手馍馍',
		  intro: '陕西人开的纯手工馍馍店，现烙白吉馍+腊汁肉，10 元一套管饱。',
		  price: '¥10-25',
		  tag: '西北',
		  image: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800'
		},
		{
		  name: '茶泡同学',
		  intro: '把“茶+奶盖”做成实验课，自己倒氮气茶，看云雾翻涌，中二又出片。',
		  price: '¥22-40',
		  tag: '茶饮',
		  image: 'https://images.unsplash.com/photo-1544787219-7f47ccb76574?w=800'
		}
      ],
      bookedSet: new Set()
    }
  },
  onShow() {
    this.refreshBookStatus()
    // #ifdef H5
    uni.$emit('tabbar-update')
    // #endif
  },
  computed: {
    filteredRestaurantList() {
      const kw = this.searchKeyword.trim().toLowerCase()
      if (!kw) return this.restaurantList
      return this.restaurantList.filter(
        i =>
          i.name.toLowerCase().includes(kw) ||
          i.intro.toLowerCase().includes(kw)
      )
    }
  },
  methods: {
    refreshBookStatus() {
      const list = uni.getStorageSync('BOOKINGS') || []
      const set = new Set(list.filter(i => !i.isCancel).map(i => i.shopName))
      this.bookedSet = set
    },
    handleSearch() {},
    clearSearch() {
      this.searchKeyword = ''
    },
    goToDetail(name) {
      uni.navigateTo({
        url: `/pages/restaurant/detail?name=${encodeURIComponent(name)}`
      })
    }
  }
}
</script>

<style scoped>
/* 整体背景 */
	.allcontainer{
			height: 100%;
			width: 100%;
			background-size: 100% 100%;
			background-image: url('/static/ocean.jpg');   /* 路径换成你的 */
		}
.container {
	max-width: 700px;
	margin: 0 auto;
  min-height: 100vh;
  background: #f5f5f5;
  /* #ifdef H5 */
  padding-bottom: calc(40rpx + 120rpx);
  /* #endif */
  /* #ifndef H5 */
  padding-bottom: 40rpx;
  /* #endif */
}

/* 搜索区域 */
.search-section {
  background: linear-gradient(135deg, #54e8ea 0%, #764ba2 100%);
  padding: 30rpx 40rpx 40rpx;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 4rpx 20rpx rgba(102, 126, 234, 0.3);
}
.search-box {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10rpx);
  border-radius: 50rpx;
  padding: 24rpx 30rpx;
  box-shadow: 0 4rpx 15rpx rgba(0, 0, 0, 0.1);
}
.search-icon {
  font-size: 32rpx;
  margin-right: 20rpx;
  color: #667eea;
}
.search-input {
  flex: 1;
  font-size: 28rpx;
  color: #2c3e50;
}
.search-input::placeholder {
  color: #95a5a6;
}
.clear-icon {
  font-size: 28rpx;
  color: #95a5a6;
  padding: 10rpx;
  margin-left: 10rpx;
}

/* 餐厅列表 */
.restaurant-list {
  padding: 30rpx 30rpx 40rpx;
}
.restaurant-card {
  background: #ffffff;
  border-radius: 24rpx;
  margin-bottom: 30rpx;
  overflow: hidden;
  box-shadow: 0 8rpx 30rpx rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  position: relative;
}
.restaurant-card:active {
  transform: translateY(-4rpx) scale(0.98);
  box-shadow: 0 12rpx 40rpx rgba(0, 0, 0, 0.15);
}

/* 卡片图片区域 */
.card-image-wrapper {
  position: relative;
  width: 100%;
  height: 300rpx;
  overflow: hidden;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
.card-image {
  width: 100%;
  height: 100%;
  transition: transform 0.3s ease;
}
.restaurant-card:active .card-image {
  transform: scale(1.05);
}

/* 预约状态小标签 */
.book-badge {
  position: absolute;
  left: 20rpx;
  top: 20rpx;
  padding: 6rpx 16rpx;
  border-radius: 20rpx;
  font-size: 22rpx;
  font-weight: 600;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.2);
}
.book-badge.booked {
  background: #f5576c;
  color: #fff;
}
.book-badge.can-book {
  background: #43e97b;
  color: #fff;
}
.book-badge-text {
  letter-spacing: 1rpx;
}

/* 价格标签 */
.price-badge {
  position: absolute;
  right: 20rpx;
  top: 20rpx;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  padding: 12rpx 24rpx;
  border-radius: 50rpx;
  box-shadow: 0 4rpx 15rpx rgba(245, 87, 108, 0.4);
  backdrop-filter: blur(10rpx);
}
.price-badge-text {
  color: #ffffff;
  font-size: 26rpx;
  font-weight: 600;
  letter-spacing: 1rpx;
}

/* 卡片内容 */
.card-body {
  padding: 30rpx;
}
.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20rpx;
}
.restaurant-name {
  font-size: 38rpx;
  font-weight: 700;
  color: #2c3e50;
  line-height: 1.3;
  flex: 1;
  letter-spacing: 1rpx;
}
.arrow-wrapper {
  width: 60rpx;
  height: 60rpx;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 20rpx;
  box-shadow: 0 4rpx 15rpx rgba(102, 126, 234, 0.3);
}
.arrow-icon {
  font-size: 40rpx;
  color: #ffffff;
  font-weight: bold;
}
.card-content {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}
.intro-text {
  font-size: 28rpx;
  color: #5a6c7d;
  line-height: 1.8;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10rpx;
}
.tag-wrapper {
  display: flex;
  gap: 12rpx;
}
.tag {
  display: inline-block;
  padding: 8rpx 20rpx;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: #ffffff;
  font-size: 22rpx;
  border-radius: 20rpx;
  font-weight: 500;
  box-shadow: 0 2rpx 8rpx rgba(245, 87, 108, 0.3);
}

/* 空状态 */
.empty-tip {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 200rpx 0;
}
.empty-text {
  font-size: 28rpx;
  color: #95a5a6;
}
</style>