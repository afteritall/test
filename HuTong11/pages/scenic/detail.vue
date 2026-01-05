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
        },
      'Rainbow 彩虹楼梯': {
          name: 'Rainbow 彩虹楼梯',
          images: [
            'https://images.unsplash.com/photo-1559181567-c3190ca9959b?w=800',
            'https://images.unsplash.com/photo-1504215680853-026ed2a45def?w=800'
          ],
          intro: '胡同里藏着的 7 色渐变楼梯，从上往下拍腿长 2 米，晴天 10:00-14:00 光影最绝。',
          geography: '藏在北锣鼓巷支巷的彩虹楼梯，由附近美院学生自发涂鸦完成。7 色渐变从薄荷绿到蜜桃粉，每一步都是马卡龙。因为巷子窄，阳光直射时形成天然“影棚光”，手机原片就能直出发圈。',
          price: '免费',
          parking: '北锣鼓巷 45 号旁（地铁北锣 A 口步行 3 分钟）'
        },
        '银河汽水实验室': {
          name: '银河汽水实验室',
          images: [
            'https://images.unsplash.com/photo-1556909114-75a47b5a7f5f?w=800',
            'https://images.unsplash.com/photo-1504215680853-026ed2a45def?w=800'
          ],
          intro: '自制“星系气泡水”，把 edible glitter 灌进汽水里，关灯瞬间星河爆炸，超治愈。',
          geography: '帝都首家“可食用银河”主题实验室，利用食品级珠光粉与二氧化碳分层，制造出“星云翻滚”的视觉。店内每天只开放 30 个体验名额，调好的汽水在黑暗中呈现蓝紫渐变，号称“喝下一口银河”。',
          price: '人均 28-48 元',
          parking: '交道口南大街 88 号（地铁张自忠路站 B 口步行 5 分钟）'
        },
        '时光打字机咖啡馆': {
          name: '时光打字机咖啡馆',
          images: [
            'https://images.unsplash.com/photo-1507048331197-7d4ac70811cf?w=800',
            'https://images.unsplash.com/photo-1481627834876-b7833e8f5570?w=800'
          ],
          intro: '店里摆着 20 台 1930s 打字机，可敲一张“旧时代情书”当明信片，文艺值 +100。',
          geography: '咖啡馆收藏了 20 台 1930-1950 年欧美古董打字机，每台均可正常工作。客人可任选一台，用店内提供的亚麻信纸敲一封 140 字以内的“情书”，盖上火漆章，由店家 7 天后寄出。因为纸张做旧，墨迹带一点脱色，完美复刻“慢时代”浪漫。',
          price: '人均 35-65 元',
          parking: '方家胡同 12 号（地铁安定门站 B 口步行 6 分钟）'
        },
        '月亮邮筒': {
          name: '月亮邮筒',
          images: [
            'https://images.unsplash.com/photo-1481627834876-b7833e8f5570?w=800',
            'https://images.unsplash.com/photo-1518709268805-4e9042af9f23?w=800'
          ],
          intro: '巨型银色月牙形邮筒，夜里内置灯自动亮起，把明信片投进去 7 天后会收到“来自月亮的回信”。',
          geography: '由独立插画师团队打造的公共艺术装置，月牙本体用 3D 打印航空铝，表面做磨砂银处理，内置 2700K 暖光感应灯。投递的明信片会统一盖上“月面”纪念戳，并由志愿者在 7 天后手写回信寄还，形成“地月往返”仪式感。',
          price: '免费（明信片 10 元/张可选）',
          parking: '国子监街 40 号门口（地铁雍和宫站 F 口步行 3 分钟）'
        },
        '像素胡同': {
          name: '像素胡同',
          images: [
            'https://images.unsplash.com/photo-1558221639-130a5f44e3f4?w=800',
            'https://images.unsplash.com/photo-1520637836862-4d197d17c93a?w=800'
          ],
          intro: '整条巷子地面贴满 8-bit 马赛克，踩在上面像跳“马里奥”，无人机俯拍效果炸裂。',
          geography: '地面采用防滑环氧地坪漆，手工绘制 30×30 cm 的 8-bit 像素格，共 12 种经典游戏角色隐藏其中（吃豆人、马里奥、星星等）。巷子仅 1.8 米宽，手机垂直俯拍即可得到“满屏像素”效果，是游戏爱好者的隐藏打卡地。',
          price: '免费',
          parking: '柴棒胡同 3-9 号（地铁南锣鼓巷站 E 口步行 5 分钟）'
        },
        '风铃长廊': {
          name: '风铃长廊',
          images: [
            'https://images.unsplash.com/photo-1478827539354-8eb3f729c3a7?w=800',
            'https://images.unsplash.com/photo-1520637836862-4d197d17c93a?w=800'
          ],
          intro: '百米长廊挂 3000+ 只手工风铃，四季风向不同铃声也不同，闭眼拍 vlog 自带 BGM。',
          geography: '长廊为传统木结构，顶部悬挂 3000+ 只手绘玻璃风铃，材质分铜/竹/玻璃三种，对应高中低三频。春季东南风多，铃声清脆；秋季西北风强劲，低音厚重。长廊尽头设录音区，可现场采样做 vlog BGM，号称“会唱歌的胡同”。',
          price: '免费',
          parking: '鼓楼东大街 190 号后院（地铁鼓楼大街站 G 口步行 4 分钟）'
        },
        '云朵制造机': {
          name: '云朵制造机',
          images: [
            'https://images.unsplash.com/photo-1518709268805-4e9042af9f23?w=800',
            'https://images.unsplash.com/photo-1481627834876-b7833e8f5570?w=800'
          ],
          intro: '门口机器 30 秒生成“可食用云朵”棉花糖，拍照完一口吃掉，零卡代糖不怕胖。',
          geography: '店内主打“分子糖果”概念，使用零卡赤藓糖醇 + 植物色素，通过高速离心仪在 30 秒内生成“云朵”状棉花糖，口感比普通棉花糖更蓬松。提供 5 种口味：原味、草莓、抹茶、蝶豆花、黑可可，拍照 30 秒后开始萎缩，建议即拍即吃。',
          price: '人均 18-30 元',
          parking: '南锣鼓巷 128 号（地铁南锣鼓巷站 A 口步行 2 分钟）'
        },
        '镜面对称屋': {
          name: '镜面对称屋',
          images: [
            'https://images.unsplash.com/photo-1520637836862-4d197d17c93a?w=800',
            'https://images.unsplash.com/photo-1558221639-130a5f44e3f4?w=800'
          ],
          intro: '屋内 360° 镜面+对称灯光，站在中间仿佛掉进盗梦空间，手机广角一键出科幻大片。',
          geography: '房间六面墙全部贴进口银镜，地面为高光黑玻，顶部隐藏式 LED 灯带形成无限反射。中央“悬浮”圆台仅供 1 人站立，手机开 0.5 倍广角，即可拍出“盗梦空间”式无限延伸画面。每次仅允许 3 人进入，避免镜面穿帮。',
          price: '体验票 48 元/人（含 10 分钟拍摄时段）',
          parking: '菊儿胡同 73 号地下一层（地铁南锣鼓巷站 D 口步行 4 分钟）'
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
      /* ===== 新增：统一字段 + 格式化时间 ===== */
        const dateTime = new Date().toLocaleString('zh-CN', {
          year: 'numeric', month: 'numeric', day: 'numeric',
          hour: '2-digit', minute: '2-digit', second: '2-digit',
          hour12: true
        }).replace(/年|月/g, '/').replace(/日/, '');   // 2026/1/5 上午10:58:19
      
        list.unshift({
          id: Date.now(),
          type: 'scenic',                    // ① 类型
          shopName: this.scenicDetail.name,  // ② 统一字段
          date: dateTime,                    // ③ 带时间
          mobile: this.mobile,
          isCancel: false
        });
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