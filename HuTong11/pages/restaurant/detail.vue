<template>
  <view class="container">
    <!-- 轮播图 -->
    <view class="image-section">
      <swiper class="restaurant-swiper" indicator-dots autoplay interval="3000" duration="500" circular>
        <swiper-item v-for="(img,idx) in restaurantDetail.images" :key="idx">
          <image class="restaurant-image" :src="img" mode="aspectFill" />
        </swiper-item>
      </swiper>
    </view>

    <!-- 餐厅信息 -->
    <view class="info-container">
      <view class="title-section">
        <text class="restaurant-title">{{ restaurantDetail.name }}</text>
      </view>

      <view class="detail-card">
        <view class="detail-item">
          <text class="detail-label">📖 简介</text>
          <text class="detail-content">{{ restaurantDetail.intro }}</text>
        </view>

        <view class="detail-item" v-if="restaurantDetail.features">
          <text class="detail-label">⭐ 特色</text>
          <text class="detail-content">{{ restaurantDetail.features }}</text>
        </view>

        <view class="detail-item" v-if="restaurantDetail.price">
          <text class="detail-label">💰 人均消费</text>
          <text class="detail-content price-text">{{ restaurantDetail.price }}</text>
        </view>

        <view class="detail-item" v-if="restaurantDetail.recommend">
          <text class="detail-label">🍽️ 推荐菜品</text>
          <text class="detail-content">{{ restaurantDetail.recommend }}</text>
        </view>

        <view class="detail-item" v-if="restaurantDetail.hours">
          <text class="detail-label">🕐 营业时间</text>
          <text class="detail-content">{{ restaurantDetail.hours }}</text>
        </view>
      </view>

      <!-- 个人中心入口（临时） -->
      <view class="mine-btn" @click="goMine">查看我的预约 →</view>
    </view>

    <!-- 底部预约栏 -->
    <view class="footer-bar">
      <button v-if="!booked" class="btn-primary" @click="showPop">立即预约</button>
      <button v-else class="btn-cancel" @click="cancelBook">已预约 · 点击取消</button>
    </view>

    <!-- 自制弹窗（不依赖 uni-popup） -->
    <view v-if="showModal" class="modal-mask" @click="hidePop">
      <view class="modal-box" @click.stop>
        <view class="modal-title">输入手机号预约</view>
        <input v-model="mobile" class="modal-input" type="number" maxlength="11" placeholder="请输入11位手机号" />
        <view class="modal-footer">
          <text class="modal-btn" @click="hidePop">取消</text>
          <text class="modal-btn primary" @click="doBook">确定</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      restaurantDetail: {},
      mobile: '',
      booked: false,
      showModal: false,
      // ====== 12 家餐厅完整数据 ======
      allRestaurantDetails: {
        '蜜雪冰城': {
          name: '蜜雪冰城',
          images: [
            'https://images.unsplash.com/photo-1559056199-641a0ac8b55e?w=800',
            'https://images.unsplash.com/photo-1525385133512-2f3bdd039054?w=800'
          ],
          intro: '胡同西口打卡级「平价快乐水」，3 元甜筒、5 元果茶，学生游客第一站。地点位于东城区安定门内大街16号',
          features: '超高性价比的饮品店，价格亲民，是学生和游客的最爱。店内环境简洁明亮，服务快速，是胡同里最受欢迎的平价饮品店。',
          price: '人均 3-15 元',
          recommend: '3元甜筒、5元果茶、柠檬水、珍珠奶茶',
          hours: '09:00 - 22:00'
        },
        '一拙茶研所': {
          name: '一拙茶研所',
          images: [
            'https://images.unsplash.com/photo-1544787219-7f47ccb76574?w=800',
            'https://images.unsplash.com/photo-1511920170033-83939cdc2da7?w=800'
          ],
          intro: '老平房改成的极简茶空间，主打手冲原叶与茶拿铁，可安静办公一下午。地点位于东城区五道营胡同84号',
          features: '由老北京平房改造的极简茶空间，保留了胡同的原始韵味。店内环境安静舒适，适合办公、阅读或与朋友小聚。主打手冲原叶茶和创意茶拿铁，是文艺青年的理想去处。',
          price: '人均 35-80 元',
          recommend: '手冲原叶茶、茶拿铁、茶点套餐',
          hours: '10:00 - 20:00'
        },
        '敦敦斋': {
          name: '敦敦斋',
          images: [
            'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800',
            'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?w=800'
          ],
          intro: '清真家常菜小馆，招牌"敦敦包子"一天只出 3 屉，皮薄汁多，11 点前排不上。地点为五道口胡同80号',
          features: '胡同里的清真家常菜馆，以招牌"敦敦包子"闻名。包子每天限量供应，皮薄汁多，深受食客喜爱。店内环境朴实，但味道地道，是体验老北京家常菜的好去处。',
          price: '人均 25-50 元',
          recommend: '敦敦包子、牛肉面、羊肉串、家常小炒',
          hours: '07:00 - 14:00, 17:00 - 21:00'
        },
        '兰·泰餐': {
          name: '兰·泰餐',
          images: [
            'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?w=800',
            'https://images.unsplash.com/photo-1414235077428-338989a2e8c0?w=800'
          ],
          intro: '胡同里最便宜的泰式小馆，冬阴功 28 元、泰式奶茶 12 元，口味改良偏甜。地点为五道口胡同78号',
          features: '胡同里性价比最高的泰式餐厅，价格亲民，口味经过改良更适合中国人口味。店内装修充满东南亚风情，是体验异域美食的平价选择。',
          price: '人均 30-60 元',
          recommend: '冬阴功汤、泰式奶茶、绿咖喱、芒果糯米饭',
          hours: '11:00 - 21:30'
        },
        '王欻欻火锅': {
          name: '王欻欻火锅',
          images: [
            'https://images.unsplash.com/photo-1556911220-e15b29be8c8f?w=800',
            'https://images.unsplash.com/photo-1529692236671-f1f6cf9683ba?w=800'
          ],
          intro: '一人食小火锅，锅底 9.9 元，菜量可按"半份"点，适合独自探店。地点为五道营胡同70-3号',
          features: '专为一人食设计的小火锅店，锅底价格实惠，菜品可按半份点单，非常适合独自用餐。店内环境温馨，是独自探店的好选择。',
          price: '人均 40-80 元',
          recommend: '番茄锅底、麻辣锅底、半份菜品、小料自助',
          hours: '11:00 - 22:00'
        },
        '站点披萨': {
          name: '站点披萨',
          images: [
            'https://images.unsplash.com/photo-1513104890138-7c749659a591?w=800',
            'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?w=800'
          ],
          intro: '18 寸纽约薄底，128 元够 4 人吃；窗边位能望雍和宫红墙，拍照出片。地点位于五道营胡同68号',
          features: '美式披萨店，主打18寸纽约薄底披萨，分量十足。窗边位置可以远眺雍和宫红墙，是拍照打卡的绝佳位置。店内氛围轻松，适合朋友聚餐。',
          price: '人均 30-50 元',
          recommend: '18寸纽约薄底披萨、鸡翅、沙拉、精酿啤酒',
          hours: '11:00 - 22:00'
        },
        '松和 MATSUWA': {
          name: '松和 MATSUWA',
          images: [
            'https://images.unsplash.com/photo-1579952363873-27f3bade9f55?w=800',
            'https://images.unsplash.com/photo-1579584425555-c3ce17fd4351?w=800'
          ],
          intro: '胡同唯一日料正餐，午市定食 58 元起，晚市有炙烤和牛，清酒种类多。地点为五道营胡同75号',
          features: '胡同里唯一的日料正餐厅，午市提供性价比高的定食套餐，晚市则有高端的炙烤和牛。店内清酒种类丰富，是日料爱好者的好去处。',
          price: '人均 80-200 元',
          recommend: '午市定食、炙烤和牛、刺身拼盘、清酒',
          hours: '11:30 - 14:00, 17:30 - 22:00'
        },
        '串门儿串串火锅': {
          name: '串门儿串串火锅',
          images: [
            'https://images.unsplash.com/photo-1556911220-e15b29be8c8f?w=800',
            'https://images.unsplash.com/photo-1529692236671-f1f6cf9683ba?w=800'
          ],
          intro: '成都冷锅串串+热卤，牛肉串 1 元/签，干碟香而不辣，夜宵营业到 24:00。地点位于五道营胡同73号院后面',
          features: '正宗的成都串串店，提供冷锅串串和热卤。价格实惠，牛肉串仅1元/签。干碟调料香而不辣，适合不太能吃辣的食客。营业至深夜，是夜宵的好选择。',
          price: '人均 50-100 元',
          recommend: '牛肉串、冷锅串串、热卤、干碟、冰粉',
          hours: '17:00 - 24:00'
        },
        '胡同肉饼坊': {
          name: '胡同肉饼坊',
          images: [
            'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=800',
            'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?w=800'
          ],
          intro: '现烙牛肉肉饼 8 元/张，外酥里汁，加一碗小米粥是周边上班族的"隐藏套餐"。地点位于东城区安定门内大街66号',
          features: '胡同里的老字号肉饼店，现烙现卖，外酥里嫩，汁水丰富。搭配小米粥是周边上班族的经典套餐，价格实惠，味道地道。',
          price: '人均 15-30 元',
          recommend: '牛肉肉饼、小米粥、小菜、豆浆',
          hours: '06:30 - 20:00'
        },
        '满香馄饨': {
          name: '满香馄饨',
          images: [
            'https://images.unsplash.com/photo-1559056199-641a0ac8b55e?w=800',
            'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?w=800'
          ],
          intro: '夫妻老店，鲜肉馄饨 12 元/碗，汤底用猪骨熬，可加免费韭菜花。地点为东城区安定门内大街77号',
          features: '胡同里的夫妻老店，传承了传统馄饨的制作工艺。汤底用猪骨精心熬制，味道鲜美。可以免费加韭菜花，是老北京的传统吃法。',
          price: '人均 12-25 元',
          recommend: '鲜肉馄饨、韭菜花、小菜、茶叶蛋',
          hours: '07:00 - 21:00'
        },
        '涮火捞石锅鱼': {
          name: '涮火捞石锅鱼',
          images: [
            'https://images.unsplash.com/photo-1529692236671-f1f6cf9683ba?w=800',
            'https://images.unsplash.com/photo-1556911220-e15b29be8c8f?w=800'
          ],
          intro: '酸菜黑鱼片石锅，先喝鱼汤再涮菜，团购 88 元双人餐含饮料。地点位于东城区安定门内大街93号',
          features: '特色石锅鱼店，使用酸菜和黑鱼片，先喝汤再涮菜，是独特的用餐体验。团购套餐性价比高，适合情侣或朋友聚餐。',
          price: '人均 50-80 元',
          recommend: '酸菜黑鱼片、石锅鱼、配菜、团购双人餐',
          hours: '11:00 - 22:00'
        },
        '董记粗粮': {
          name: '董记粗粮',
          images: [
            'https://images.unsplash.com/photo-1542838132-92c53300491e?w=800',
            'https://images.unsplash.com/photo-1466692476868-aef1dfb1e735?w=800'
          ],
          intro: '健康轻食路线：莜面鱼鱼、玉米面煎饼、杂粮豆浆，适合素食/控糖人群。地点位于东城区安定门内大街119号鲜果味超市进门直走到头',
          features: '主打健康轻食的餐厅，提供各种粗粮制品，适合追求健康饮食的人群。菜品适合素食者和控糖人群，是胡同里的健康之选。',
          price: '人均 20-40 元',
          recommend: '莜面鱼鱼、玉米面煎饼、杂粮豆浆、粗粮小菜',
          hours: '07:00 - 20:00'
        }
      }
    }
  },
  onLoad(options) {
    const name = decodeURIComponent(options.name || '')
    if (name && this.allRestaurantDetails[name]) {
      this.restaurantDetail = this.allRestaurantDetails[name]
      uni.setNavigationBarTitle({ title: name })
      this.checkBookStatus(name)
    } else {
      uni.showToast({ title: '餐厅信息不存在', icon: 'none' })
      setTimeout(() => uni.navigateBack(), 1500)
    }
  },
  methods: {
    checkBookStatus(name) {
      const arr = uni.getStorageSync('BOOKINGS') || []
      this.booked = arr.some(i => i.shopName === name && !i.isCancel)
    },
    showPop() {
      this.showModal = true
    },
    hidePop() {
      this.showModal = false
      this.mobile = ''
    },
    doBook() {
      if (!/^1[3-9]\d{9}$/.test(this.mobile)) {
        uni.showToast({ title: '手机号格式错误', icon: 'none' })
        return
      }
      const arr = uni.getStorageSync('BOOKINGS') || []
      if (arr.some(i => i.shopName === this.restaurantDetail.name && !i.isCancel)) {
        uni.showToast({ title: '您已预约过本店', icon: 'none' })
        this.hidePop()
        return
      }
      arr.unshift({
        id: Date.now(),
        shopName: this.restaurantDetail.name,
        mobile: this.mobile,
        date: new Date().toLocaleString('zh-CN'),
        isCancel: false
      })
      uni.setStorageSync('BOOKINGS', arr)
      this.booked = true
      this.hidePop()
      uni.showToast({ title: '预约成功' })
    },
    cancelBook() {
      const arr = uni.getStorageSync('BOOKINGS') || []
      const tar = arr.find(i => i.shopName === this.restaurantDetail.name && !i.isCancel)
      if (tar) {
        tar.isCancel = true
        uni.setStorageSync('BOOKINGS', arr)
        this.booked = false
        uni.showToast({ title: '已取消预约', icon: 'none' })
      }
    },
    goMine() {
      uni.navigateTo({ url: '/pages/mine/mine' })
    }
  }
}
</script>

<style scoped>
/* 基础结构 */
.container{min-height:100vh;background:#f5f5f5;}
.image-section{width:100%;height:500rpx;overflow:hidden;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);position:relative;}
.image-section::after{content:'';position:absolute;bottom:0;left:0;right:0;height:100rpx;background:linear-gradient(to top,rgba(255,255,255,1),transparent);z-index:1;}
.restaurant-swiper{width:100%;height:100%;}
.restaurant-image{width:100%;height:100%;}
.info-container{padding:0 30rpx 40rpx;margin-top:-40rpx;position:relative;z-index:2;}
.title-section{margin-bottom:30rpx;padding:30rpx;background:#ffffff;border-radius:24rpx 24rpx 0 0;box-shadow:0 -4rpx 20rpx rgba(0,0,0,.05);}
.restaurant-title{font-size:52rpx;font-weight:700;color:#2c3e50;line-height:1.3;letter-spacing:1rpx;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;}
.detail-card{background:#ffffff;border-radius:0 0 24rpx 24rpx;padding:0 30rpx 40rpx;box-shadow:0 8rpx 30rpx rgba(0,0,0,.1);}
.detail-item{margin-bottom:40rpx;padding:30rpx;background:linear-gradient(135deg,#f5f7fa 0%,#ffffff 100%);border-radius:20rpx;border-left:6rpx solid;box-shadow:0 2rpx 10rpx rgba(0,0,0,.05);transition:all .3s ease;}
.detail-item:nth-child(1){border-left-color:#667eea;}
.detail-item:nth-child(2){border-left-color:#f093fb;}
.detail-item:nth-child(3){border-left-color:#f5576c;}
.detail-item:nth-child(4){border-left-color:#4facfe;}
.detail-item:nth-child(5){border-left-color:#43e97b;}
.detail-item:last-child{margin-bottom:0;}
.detail-label{display:flex;align-items:center;font-size:32rpx;font-weight:700;margin-bottom:20rpx;letter-spacing:1rpx;}
.detail-item:nth-child(1) .detail-label{color:#667eea;}
.detail-item:nth-child(2) .detail-label{color:#f093fb;}
.detail-item:nth-child(3) .detail-label{color:#f5576c;}
.detail-item:nth-child(4) .detail-label{color:#4facfe;}
.detail-item:nth-child(5) .detail-label{color:#43e97b;}
.detail-content{display:block;font-size:28rpx;color:#5a6c7d;line-height:1.9;text-align:justify;}
.price-text{color:#f5576c;font-weight:700;font-size:32rpx;letter-spacing:1rpx;}

/* 底部预约栏 */
.footer-bar{position:fixed;left:0;right:0;bottom:0;background:#fff;padding:20rpx 30rpx;box-shadow:0 -2rpx 10rpx rgba(0,0,0,.08);z-index:9;}
.btn-primary{width:100%;height:80rpx;line-height:80rpx;text-align:center;color:#fff;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);border-radius:40rpx;font-size:30rpx;border:none;}
.btn-cancel{width:100%;height:80rpx;line-height:80rpx;text-align:center;color:#999;background:#f5f5f5;border-radius:40rpx;font-size:30rpx;border:none;}

/* 个人中心入口 */
.mine-btn{margin:40rpx 0;font-size:30rpx;color:#667eea;text-align:center;}

/* 自制弹窗 */
.modal-mask{position:fixed;left:0;top:0;right:0;bottom:0;background:rgba(0,0,0,.5);z-index:999;display:flex;align-items:center;justify-content:center;}
.modal-box{width:600rpx;background:#fff;border-radius:20rpx;padding:40rpx;}
.modal-title{font-size:32rpx;font-weight:bold;margin-bottom:30rpx;text-align:center;}
.modal-input{height:80rpx;border:1px solid #ddd;border-radius:10rpx;padding:0 20rpx;font-size:30rpx;}
.modal-footer{margin-top:40rpx;display:flex;justify-content:space-between;}
.modal-btn{flex:1;text-align:center;line-height:70rpx;font-size:30rpx;color:#666;}
.modal-btn.primary{color:#667eea;}
</style>