<template>
  <view class="container">
    <!-- 景点图片轮播 -->
    <view class="image-section">
      <swiper class="scenic-swiper" indicator-dots autoplay interval="3000" duration="500" circular>
        <swiper-item v-for="(img,idx) in scenicDetail.images" :key="idx">
          <image class="scenic-image" :src="img" mode="aspectFill" />
        </swiper-item>
      </swiper>
    </view>

    <!-- 景点信息 -->
    <view class="info-container">
      <view class="title-section">
        <text class="scenic-title">{{ scenicDetail.name }}</text>
      </view>

      <view class="detail-card">
        <view class="detail-item">
          <text class="detail-label">📖 简介</text>
          <text class="detail-content">{{ scenicDetail.intro }}</text>
        </view>

        <view class="detail-item" v-if="scenicDetail.geography">
          <text class="detail-label">🌍 人文地理</text>
          <text class="detail-content">{{ scenicDetail.geography }}</text>
        </view>

        <view class="detail-item" v-if="scenicDetail.price">
          <text class="detail-label">💰 人均消费</text>
          <text class="detail-content price-text">{{ scenicDetail.price }}</text>
        </view>

        <view class="detail-item">
          <text class="detail-label">📍 地址</text>
          <text class="detail-content">{{ scenicDetail.parking }}</text>
        </view>
      </view>
    </view>

    <!-- 底部预约栏 -->
    <view class="footer-bar">
      <button v-if="!booked" class="btn-primary" @click="showPop">立即预约</button>
      <button v-else class="btn-cancel" @click="cancelBook">已预约 · 点击取消</button>
    </view>

    <!-- 手机号弹窗（自制） -->
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
      scenicDetail: {},
      mobile: '',
      booked: false,
      showModal: false,
      // ====== 8 条景点完整数据 ======
      allScenicDetails: {
        '五道营胡同78号「多巴胺涂鸦墙」': {
          name: '五道营胡同78号「多巴胺涂鸦墙」',
          images: [
            'https://images.unsplash.com/photo-1513475382585-d06e58bcb0e0?w=800',
            'https://images.unsplash.com/photo-1547036967-23d11aacaee0?w=800'
          ],
          intro: '整条胡同最炸的"露天摄影棚"，整面粉刷墙被涂成高饱和黄/粉/蓝几何块，站在中间摆夸张pose，广角一拍就是杂志封面；早晚人少，侧光更显色。',
          geography: '位于五道营胡同78号，是整条胡同最具视觉冲击力的拍照打卡地。这面涂鸦墙由当代艺术家创作，采用高饱和度的黄、粉、蓝几何色块，展现了现代艺术与胡同文化的碰撞。这里不仅是摄影爱好者的天堂，更是年轻人表达个性的舞台，体现了五道营胡同从传统到现代的转变。',
          price: '免费',
          parking: '五道营胡同78号'
        },
        '观夏（五道营店）': {
          name: '观夏（五道营店）',
          images: [
            'https://images.unsplash.com/photo-1544787219-7f47ccb76574?w=800',
            'https://images.unsplash.com/photo-1511920170033-83939cdc2da7?w=800'
          ],
          intro: '帝都香薰顶流，纯白极简+留白走廊。建议穿黑/红/克莱因蓝，手持白色瓷杯或香砖，对墙一拍就是清冷高级感；香砖可刻字，买完当道具继续出片。',
          geography: '观夏是北京本土高端香薰品牌，五道营店延续了品牌一贯的极简美学。店内采用纯白设计，留白走廊营造出清冷高级的氛围。这里不仅是购买香薰的场所，更是体验东方美学和现代设计融合的空间。店内香砖可定制刻字，是独特的纪念品。',
          price: '人均 200-500 元（香薰产品）',
          parking: '东城区国子监街23号；胡同内无停车位。'
        },
        'Wake Me Up Café': {
          name: 'Wake Me Up Café',
          images: [
            'https://images.unsplash.com/photo-1509042239860-f550ce710b93?w=800',
            'https://images.unsplash.com/photo-1511920170033-83939cdc2da7?w=800'
          ],
          intro: '韩系奶油系小店，木质长桌+拱形窗。选豆吧台是黄金机位，侧窗自然光自带柔焦。上午10点前没人，点一杯"自制冷萃"当道具，安静办公也OK。',
          geography: '五道营胡同的韩系咖啡店，以奶油色系和温馨氛围著称。店内采用木质长桌和拱形窗设计，营造出韩式小清新的风格。选豆吧台是拍照的黄金机位，侧窗的自然光自带柔焦效果。这里不仅是咖啡爱好者的聚集地，也是自由职业者和学生的理想办公空间。',
          price: '人均 35-60 元',
          parking: '五道营胡同38号院；店内消费可免费停车 1 h。'
        },
        '福顶咖啡「屋顶银杏机位」': {
          name: '福顶咖啡「屋顶银杏机位」',
          images: [
            'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=800',
            'https://images.unsplash.com/photo-1513475382585-d06e58bcb0e0?w=800'
          ],
          intro: '入门直接上露台，红瓦+灰墙+银杏王（11月全黄）。想独占C位得9:00前到，下午逆光易"死亡高光"；手机开0.5倍广角，脚贴栏杆，秒变"故宫同款屋顶"。',
          geography: '福顶咖啡的屋顶露台是五道营胡同最独特的观景平台。这里可以俯瞰整个胡同，红瓦灰墙与巨大的银杏树形成绝美的画面。11月银杏全黄时，这里成为摄影师的必争之地。露台设计参考了传统北京四合院的屋顶结构，让人仿佛置身于故宫的屋顶，是体验老北京建筑美学的绝佳位置。',
          price: '人均 40-80 元',
          parking: '五道营胡同26号院；消费满 50 元免 2 h。'
        },
        'ICEDRIC LOMRE 香水实验室': {
          name: 'ICEDRIC LOMRE 香水实验室',
          images: [
            'https://images.unsplash.com/photo-1441986300917-64674bd600d8?w=800',
            'https://images.unsplash.com/photo-1445205170230-053b83016050?w=800'
          ],
          intro: '门口巨型紫花框是"五道营最柔背景"，穿白裙站在框里，人像居中，一键柔紫滤镜。店内太空舱香墙，关灯拍剪影也酷。',
          geography: 'ICEDRIC LOMRE 是一家创意香水实验室，将科技感与艺术美学完美结合。门口的巨型紫花框是五道营最柔美的拍照背景，店内太空舱式的香墙设计充满未来感。这里不仅是购买香水的场所，更是体验香氛艺术和科技创新的空间，体现了五道营胡同的多元文化特色。',
          price: '人均 300-800 元（香水产品）',
          parking: '五道营胡同48号院；胡同窄，建议步行进入。'
        },
        '北冰洋制冰厂／童年小卖铺': {
          name: '北冰洋制冰厂／童年小卖铺',
          images: [
            'https://images.unsplash.com/photo-1556912172-45b7abe8b7e1?w=800',
            'https://images.unsplash.com/photo-1556911220-bff31c812dba?w=800'
          ],
          intro: '复刻80-90后记忆：北冰洋汽水纸箱子、玻璃瓶装酸奶、铁皮青蛙。穿海魂衫+红领巾，拿塑料小风扇，开胶片滤镜直接穿越。',
          geography: '这是一家怀旧主题的小卖铺，完美复刻了80-90年代的童年记忆。店内陈列着北冰洋汽水纸箱子、玻璃瓶装酸奶、铁皮青蛙等经典怀旧商品。这里不仅是购物场所，更是80后、90后的时光机，让人瞬间回到那个纯真年代。店内还提供怀旧服装租赁，可以穿着海魂衫和红领巾拍照，体验完整的怀旧氛围。',
          price: '人均 20-50 元',
          parking: '五道营胡同66号院；店门口无车位，建议步行进入。'
        },
        '松和日料居酒屋（Hello Kitty 门头）': {
          name: '松和日料居酒屋（Hello Kitty 门头）',
          images: [
            'https://images.unsplash.com/photo-1579952363873-27f3bade9f55?w=800',
            'https://images.unsplash.com/photo-1579584425555-c3ce17fd4351?w=800'
          ],
          intro: '全胡同最粉嫩——整扇Kitty浮雕门+日式红灯笼。建议傍晚灯亮后拍，和服或JK制服更搭；推门进去还有Kitty餐盘，可边吃边二次取景。',
          geography: '松和日料居酒屋是五道营胡同最独特的日料店，以Hello Kitty主题门头闻名。整扇Kitty浮雕门和日式红灯笼营造出浓郁的日式氛围，是全胡同最粉嫩的打卡地。店内不仅提供正宗的日式料理，还有Hello Kitty主题的餐盘和装饰，是日料爱好者和Kitty粉丝的必访之地。傍晚时分，红灯笼亮起，这里成为拍照的绝佳背景。',
          price: '人均 100-200 元',
          parking: '五道营胡同18号院；消费满 100 元免 2 h。'
        },
        '胡同二楼取景处': {
          name: '胡同二楼取景处',
          images: [
            'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=800',
            'https://images.unsplash.com/photo-1513475382585-d06e58bcb0e0?w=800'
          ],
          intro: '适合拍照打卡，风景优美，俯瞰整个北京城。',
          geography: '二楼取景处是五道营胡同的隐藏打卡地，位于胡同内某建筑的二楼平台。这里可以俯瞰整个北京城的景色，是拍摄城市全景的绝佳位置。站在这里，可以看到传统胡同与现代都市的完美融合，感受北京这座城市的独特魅力。这里适合拍照打卡，风景优美，是摄影师和游客的必访之地。',
          price: '免费（部分时段需预约）',
          parking: '永康胡同18号院，建议提前预约。'
        }
      }
    }
  },
  onLoad(options) {
    const name = decodeURIComponent(options.name || '')
    if (name && this.allScenicDetails[name]) {
      this.scenicDetail = this.allScenicDetails[name]
      uni.setNavigationBarTitle({ title: name })
      this.checkBookStatus(name)
    } else {
      uni.showToast({ title: '景点信息不存在', icon: 'none' })
      setTimeout(() => uni.navigateBack(), 1500)
    }
  },
  methods: {
    checkBookStatus(name) {
      const list = uni.getStorageSync('BOOKINGS') || []
      this.booked = list.some(i => i.shopName === name && !i.isCancel)
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
      const list = uni.getStorageSync('BOOKINGS') || []
      if (list.some(i => i.shopName === this.scenicDetail.name && !i.isCancel)) {
        uni.showToast({ title: '您已预约过本景点', icon: 'none' })
        this.hidePop()
        return
      }
      list.unshift({
        id: Date.now(),
        shopName: this.scenicDetail.name,
        mobile: this.mobile,
        date: new Date().toLocaleString('zh-CN'),
        isCancel: false
      })
      uni.setStorageSync('BOOKINGS', list)
      this.booked = true
      this.hidePop()
      uni.showToast({ title: '预约成功' })
    },
    cancelBook() {
      const list = uni.getStorageSync('BOOKINGS') || []
      const tar = list.find(i => i.shopName === this.scenicDetail.name && !i.isCancel)
      if (tar) {
        tar.isCancel = true
        uni.setStorageSync('BOOKINGS', list)
        this.booked = false
        uni.showToast({ title: '已取消预约', icon: 'none' })
      }
    }
  }
}
</script>

<style scoped>
/* ========= 原样式保持不变，仅追加预约相关 ========= */
.container{min-height:100vh;background:#f5f5f5;}
.image-section{width:100%;height:500rpx;overflow:hidden;background:#e0e0e0;}
.scenic-swiper{width:100%;height:100%;}
.scenic-image{width:100%;height:100%;}
.info-container{padding:40rpx;}
.title-section{margin-bottom:30rpx;}
.scenic-title{font-size:48rpx;font-weight:bold;color:#2c3e50;line-height:1.4;}
.detail-card{background:#ffffff;border-radius:20rpx;padding:40rpx;box-shadow:0 4rpx 20rpx rgba(0,0,0,.08);}
.detail-item{margin-bottom:40rpx;padding-bottom:30rpx;border-bottom:1rpx solid #f0f0f0;}
.detail-item:last-child{margin-bottom:0;padding-bottom:0;border-bottom:none;}
.detail-label{display:block;font-size:32rpx;font-weight:600;color:#3498db;margin-bottom:20rpx;}
.detail-content{display:block;font-size:28rpx;color:#5a6c7d;line-height:1.8;text-align:justify;}
.price-text{color:#e74c3c;font-weight:500;font-size:30rpx;}

/* ===== 底部预约栏 ===== */
.footer-bar{position:fixed;left:0;right:0;bottom:0;background:#fff;padding:20rpx 30rpx;box-shadow:0 -2rpx 10rpx rgba(0,0,0,.08);z-index:9;}
.btn-primary{width:100%;height:80rpx;line-height:80rpx;text-align:center;color:#fff;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);border-radius:40rpx;font-size:30rpx;border:none;}
.btn-cancel{width:100%;height:80rpx;line-height:80rpx;text-align:center;color:#999;background:#f5f5f5;border-radius:40rpx;font-size:30rpx;border:none;}

/* ===== 自制弹窗 ===== */
.modal-mask{position:fixed;left:0;top:0;right:0;bottom:0;background:rgba(0,0,0,.5);z-index:999;display:flex;align-items:center;justify-content:center;}
.modal-box{width:600rpx;background:#fff;border-radius:20rpx;padding:40rpx;}
.modal-title{font-size:32rpx;font-weight:bold;margin-bottom:30rpx;text-align:center;}
.modal-input{height:80rpx;border:1px solid #ddd;border-radius:10rpx;padding:0 20rpx;font-size:30rpx;}
.modal-footer{margin-top:40rpx;display:flex;justify-content:space-between;}
.modal-btn{flex:1;text-align:center;line-height:70rpx;font-size:30rpx;color:#666;}
.modal-btn.primary{color:#667eea;}
</style>