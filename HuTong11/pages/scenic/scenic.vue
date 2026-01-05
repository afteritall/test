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
          placeholder="搜索景点名称..."
          v-model="searchKeyword"
          @input="handleSearch"
          confirm-type="search"
        />
        <text v-if="searchKeyword" class="clear-icon" @click="clearSearch">✕</text>
      </view>
    </view>

    <!-- 景点列表 -->
    <view class="scenic-list">
      <view
        v-for="(item, index) in filteredScenicList"
        :key="index"
        class="scenic-card"
        @click="goToDetail(item.name)"
      >
        <!-- 预约状态小标签 -->
        <view
          :class="['book-badge', bookedSet.has(item.name) ? 'booked' : 'can-book']"
        >
          <text class="book-badge-text">
            {{ bookedSet.has(item.name) ? '已预约' : '可预约' }}
          </text>
        </view>

        <view class="card-header">
          <text class="scenic-name">{{ item.name }}</text>
          <text class="arrow-icon">›</text>
        </view>
        <view class="card-content">
          <view class="info-section">
            <text class="info-label">简介：</text>
            <text class="info-text">{{ item.intro }}</text>
          </view>
          <view class="info-section">
            <text class="info-label">地址：</text>
            <text class="info-text">{{ item.parking }}</text>
          </view>
        </view>
      </view>

      <!-- 无搜索结果提示 -->
      <view v-if="filteredScenicList.length === 0" class="empty-tip">
        <text class="empty-text">未找到相关景点</text>
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
      scenicList: [
        {
          name: '五道营胡同78号「多巴胺涂鸦墙」',
          intro: '整条胡同最炸的"露天摄影棚"，整面粉刷墙被涂成高饱和黄/粉/蓝几何块，站在中间摆夸张pose，广角一拍就是杂志封面；早晚人少，侧光更显色。',
          parking: '五道营胡同78号。'
        },
        {
          name: '观夏（五道营店）',
          intro: '帝都香薰顶流，纯白极简+留白走廊。建议穿黑/红/克莱因蓝，手持白色瓷杯或香砖，对墙一拍就是清冷高级感；香砖可刻字，买完当道具继续出片。',
          parking: '东城区国子监街23号'
        },
        {
          name: 'Wake Me Up Café',
          intro: '韩系奶油系小店，木质长桌+拱形窗。选豆吧台是黄金机位，侧窗自然光自带柔焦。上午10点前没人，点一杯"自制冷萃"当道具，安静办公也OK。',
          parking: '五道营胡同38号'
        },
        {
          name: '福顶咖啡「屋顶银杏机位」',
          intro: '入门直接上露台，红瓦+灰墙+银杏王（11月全黄）。想独占C位得9:00前到，下午逆光易"死亡高光"；手机开0.5倍广角，脚贴栏杆，秒变"故宫同款屋顶"。',
          parking: '五道营胡同26号'
        },
        {
          name: 'ICEDRIC LOMRE 香水实验室',
          intro: '门口巨型紫花框是"五道营最柔背景"，穿白裙站在框里，人像居中，一键柔紫滤镜。店内太空舱香墙，关灯拍剪影也酷。',
          parking: '五道营胡同48号'
        },
        {
          name: '北冰洋制冰厂／童年小卖铺',
          intro: '复刻80-90后记忆：北冰洋汽水纸箱子、玻璃瓶装酸奶、铁皮青蛙。穿海魂衫+红领巾，拿塑料小风扇，开胶片滤镜直接穿越。',
          parking: '五道营胡同66号'
        },
        {
          name: '松和日料居酒屋（Hello Kitty 门头）',
          intro: '全胡同最粉嫩——整扇Kitty浮雕门+日式红灯笼。建议傍晚灯亮后拍，和服或JK制服更搭；推门进去还有Kitty餐盘，可边吃边二次取景。',
          parking: '五道营胡同75号'
        },
        {
          name: '胡同二楼取景处',
          intro: '适合拍照打卡，风景优美，俯瞰整个北京城。',
          parking: '永康胡同18号院'
        },
            {
              name: 'Rainbow 彩虹楼梯',
              intro: '胡同里藏着的 7 色渐变楼梯，从上往下拍腿长 2 米，晴天 10:00-14:00 光影最绝。',
              parking: '北锣鼓巷 45 号旁（地铁北锣 A 口步行 3 分钟）'
            },
            {
              name: '银河汽水实验室',
              intro: '自制“星系气泡水”，把 edible glitter 灌进汽水里，关灯瞬间星河爆炸，超治愈。',
              parking: '交道口南大街 88 号'
            },
            {
              name: '时光打字机咖啡馆',
              intro: '店里摆着 20 台 1930s 打字机，可敲一张“旧时代情书”当明信片，文艺值 +100。',
              parking: '方家胡同 12 号'
            },
            {
              name: '月亮邮筒',
              intro: '巨型银色月牙形邮筒，夜里内置灯自动亮起，把明信片投进去 7 天后会收到“来自月亮的回信”。',
              parking: '国子监街 40 号门口'
            },
            {
              name: '像素胡同',
              intro: '整条巷子地面贴满 8-bit 马赛克，踩在上面像跳“马里奥”，无人机俯拍效果炸裂。',
              parking: '柴棒胡同 3-9 号'
            },
            {
              name: '风铃长廊',
              intro: '百米长廊挂 3000+ 只手工风铃，四季风向不同铃声也不同，闭眼拍 vlog 自带 BGM。',
              parking: '鼓楼东大街 190 号后院'
            },
            {
              name: '云朵制造机',
              intro: '门口机器 30 秒生成“可食用云朵”棉花糖，拍照完一口吃掉，零卡代糖不怕胖。',
              parking: '南锣鼓巷 128 号'
            },
            {
              name: '镜面对称屋',
              intro: '屋内 360° 镜面+对称灯光，站在中间仿佛掉进盗梦空间，手机广角一键出科幻大片。',
              parking: '菊儿胡同 73 号地下一层'
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
    filteredScenicList() {
      const kw = this.searchKeyword.trim().toLowerCase()
      if (!kw) return this.scenicList
      return this.scenicList.filter(
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
        url: `/pages/scenic/detail?name=${encodeURIComponent(name)}`
      })
    }
  }
}
</script>

<style scoped>
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
  background: #f5f5f5;
  border-radius: 50rpx;
  padding: 20rpx 30rpx;
}
.search-icon {
  font-size: 32rpx;
  margin-right: 20rpx;
  color: #7f8c8d;
}
.search-input {
  flex: 1;
  font-size: 28rpx;
  color: #2c3e50;
}
.clear-icon {
  font-size: 28rpx;
  color: #95a5a6;
  padding: 10rpx;
  margin-left: 10rpx;
}

/* 景点列表 */
.scenic-list {
  padding: 20rpx 40rpx 40rpx;
}
.scenic-card {
  background: #ffffff;
  border-radius: 20rpx;
  margin-bottom: 30rpx;
  padding: 40rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
  transition: all 0.3s;
  position: relative;
}
.scenic-card:active {
  transform: scale(0.98);
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.12);
}

/* 预约状态小标签 */
.book-badge {
  position: absolute;
  right: 3rpx;
  top: 7rpx;
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

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 30rpx;
  padding-bottom: 20rpx;
  border-bottom: 2rpx solid #f0f0f0;
}
.scenic-name {
  font-size: 36rpx;
  font-weight: bold;
  color: #2c3e50;
  line-height: 1.4;
  flex: 1;
}
.arrow-icon {
  font-size: 48rpx;
  color: #95a5a6;
  margin-left: 20rpx;
  flex-shrink: 0;
}
.card-content {
  display: flex;
  flex-direction: column;
  gap: 24rpx;
}
.info-section {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  line-height: 1.8;
}
.info-label {
  font-size: 28rpx;
  color: #3498db;
  font-weight: 500;
  flex-shrink: 0;
  margin-right: 10rpx;
}
.info-text {
  font-size: 28rpx;
  color: #5a6c7d;
  flex: 1;
  line-height: 1.8;
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