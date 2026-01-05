<template>
<view class="allcontainer">
  <view class="page">
    <!-- 顶部渐变卡片 -->
    <view class="top-card">
      <view class="tc-tit">今日预约</view>
      <view class="tc-num">{{ todayTotal }}</view>
      <view class="tc-txt">停车 · 景点 · 餐馆</view>
    </view>

    <!-- 三大功能入口 -->
    <view class="grid">
      <view class="item" @click="goList('parking')">
        <view class="item-icon 🅿️"></view>
        <view class="item-name">停车预约</view>
        <view v-if="count.parking>0" class="item-badge">{{count.parking}}</view>
      </view>

      <view class="item" @click="goList('scenic')">
        <view class="item-icon 🏞️"></view>
        <view class="item-name">景点预约</view>
        <view v-if="count.scenic>0" class="item-badge">{{count.scenic}}</view>
      </view>

      <view class="item" @click="goList('restaurant')">
        <view class="item-icon 🍽️"></view>
        <view class="item-name">餐馆预约</view>
        <view v-if="count.restaurant>0" class="item-badge">{{count.restaurant}}</view>
      </view>
    </view>

    <!-- 最近预约记录 -->
    <view class="recent-bar">
      <text class="recent-tit">最近预约</text>
      <view class="bar-right">
        <text class="plan-btn" @click="planRoute">一键规划路线</text>
        <text v-if="recentList.length" class="clear" @click="clearAll">清空</text>
      </view>
    </view>

    <view v-if="recentList.length" class="recent-list">
      <view v-for="item in recentList" :key="item.id" class="recent-item">
        <view class="ri-left">
          <view class="ri-name">{{item.shopName}}</view>
          <view class="ri-time">{{item.date}}</view>
        </view>
        <view class="ri-right">
          <text v-if="!item.isCancel" class="ri-status ok">已预约</text>
          <text v-else class="ri-status cancel">已取消</text>
          <text v-if="!item.isCancel" class="ri-cancel" @click="cancelOne(item)">取消</text>
        </view>
      </view>
    </view>

    <view v-else class="empty">
      <text class="empty-txt">暂无预约记录</text>
    </view>

    <!-- H5 自定义 Tabbar -->
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
  components:{CustomTabbar},
  // #endif
  data(){
    return {
      count:{parking:0,scenic:0,restaurant:0},
      recentList:[]
    }
  },
  onShow(){
    this.loadBookings()
    /* #ifdef H5 */ uni.$emit('tabbar-update') /* #endif */
  },
  computed:{
    todayTotal(){
      const today=new Date().toLocaleDateString('zh-CN')
      return this.recentList.filter(i=>!i.isCancel&&i.date.startsWith(today)).length
    }
  },
  methods:{
    loadBookings(){
      const list=uni.getStorageSync('BOOKINGS')||[]
      this.count = {
        parking: list.filter(i => i.type === 'parking' && !i.isCancel).length,
        scenic:  list.filter(i => i.type === 'scenic'  && !i.isCancel).length,
        restaurant: list.filter(i => i.type === 'restaurant' && !i.isCancel).length
      }
      this.recentList = list.slice(0, 20)
    },
    goList(type){
      uni.navigateTo({url: `/pages/booking-list/booking-list?type=${type}`});
    },
    cancelOne(item){
      const list=uni.getStorageSync('BOOKINGS')||[]
      const tar=list.find(i=>i.id===item.id)
      if(tar){ tar.isCancel=true;uni.setStorageSync('BOOKINGS',list);this.loadBookings() }
    },
    clearAll(){
      uni.showModal({
        title:'提示',
        content:'确定清空所有预约记录？',
        success:(res)=>{
          if(res.confirm){
            uni.setStorageSync('BOOKINGS',[]);
            this.loadBookings();
          }
        }
      })
    },

    /* 一键规划路线 */
    planRoute() {
      const valid = uni.getStorageSync('BOOKINGS').filter(i => !i.isCancel);
      if (!valid.length) {
        uni.showToast({ title: '暂无有效预约', icon: 'none' });
        return;
      }
      // 只保留店铺名，去掉 (日期)
      const content = valid.map(v => `「${v.shopName}」`).join('、');
      const prompt = `我想去五道营胡同旅游，选择自驾过去，我预约了${content}，请帮我规划合理的路线和行程`;
    
      uni.setStorageSync('AI_PROMPT', prompt);
      uni.switchTab({ url: '/pages/ai/ai' });
    }
  }
}
</script>

<style lang="scss" scoped>
$pagePad:32rpx;
$radius:24rpx;
$shadow:0 8rpx 24rpx rgba(102,126,234,.15);
	.allcontainer{
			height: 100%;
			width: 100%;
			background-size: 100% 100%;
			background-image: url('/static/ocean.jpg');   /* 路径换成你的 */
		}
.page{max-width: 700px;
		margin: 0 auto;
	min-height:100vh;background:#f4f6fc;padding:$pagePad;padding-bottom:calc(120rpx + 40rpx + env(safe-area-inset-bottom));}
/* 顶部卡片 */
.top-card{
  background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:#fff;border-radius:$radius;padding:40rpx 32rpx;margin-bottom:40rpx;text-align:center;box-shadow:$shadow;
  .tc-tit{font-size:28rpx;opacity:.9;}
  .tc-num{font-size:72rpx;font-weight:600;margin:12rpx 0;}
  .tc-txt{font-size:26rpx;opacity:.8;}
}
/* 三大入口 */
.grid{display:flex;justify-content:space-between;margin-bottom:40rpx;
  .item{flex:1;background:#fff;border-radius:$radius;padding:40rpx 20rpx;text-align:center;margin:0 8rpx;position:relative;box-shadow:0 4rpx 16rpx rgba(0,0,0,.05);transition:transform .2s;
    &:active{transform:scale(.96);}
    .item-icon{font-size:48rpx;margin-bottom:16rpx;}
    .item-name{font-size:28rpx;color:#2c3e50;}
    .item-badge{position:absolute;right:20rpx;top:20rpx;background:#f5576c;color:#fff;font-size:22rpx;padding:4rpx 12rpx;border-radius:50%;}
  }
}
/* 最近预约 */
.recent-bar{display:flex;justify-content:space-between;align-items:center;margin-bottom:20rpx;
  .recent-tit{font-size:32rpx;font-weight:600;color:#2c3e50;}
  .bar-right{display:flex;gap:20rpx;align-items:center;}
  .plan-btn{
    font-size:26rpx;color:#fff;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);padding:8rpx 16rpx;border-radius:20rpx;
  }
  .clear{font-size:26rpx;color:#667eea;}
}
.recent-list{background:#fff;border-radius:$radius;padding:0 24rpx;box-shadow:0 4rpx 16rpx rgba(0,0,0,.05);
  .recent-item{display:flex;justify-content:space-between;align-items:center;padding:28rpx 0;border-bottom:1px solid #f0f0f0;
    &:last-child{border-bottom:none;}
    .ri-left{flex:1;margin-right:20rpx;
      .ri-name{font-size:30rpx;color:#2c3e50;margin-bottom:8rpx;}
      .ri-time{font-size:24rpx;color:#95a5a6;}
    }
    .ri-right{display:flex;align-items:center;gap:16rpx;
      .ri-status{font-size:24rpx;padding:4rpx 10rpx;border-radius:8rpx;
        &.ok{background:#e8f5e9;color:#43a047;}
        &.cancel{background:#ffebee;color:#e53935;}
      }
      .ri-cancel{font-size:24rpx;color:#667eea;}
    }
  }
}
.empty{text-align:center;padding:120rpx 0;color:#95a5a6;font-size:28rpx;}
</style>