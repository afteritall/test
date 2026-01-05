<template>
  <view class="page">
    <view class="bar">
      <text class="tit">{{title}}</text>
      <text class="clear" v-if="list.length" @click="clearType">清空</text>
    </view>

    <view v-if="list.length" class="list">
      <view class="item" v-for="i in list" :key="i.id">
        <view class="left">
          <view class="name">{{i.shopName}}</view>
          <view class="time">{{i.date}}</view>
        </view>
        <view class="right">
          <text class="status ok">已预约</text>
          <text class="cancel" @click="cancelOne(i)">取消</text>
        </view>
      </view>
    </view>

    <view v-else class="empty">
      <text class="txt">暂无{{title}}</text>
    </view>

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
  components:{CustomTabbar},
  // #endif
  data(){
    return {
      type:'',   // parking / scenic / restaurant
      list:[]
    }
  },
  onLoad({type}){
    this.type = type;
    this.loadList();
  },
  onShow(){
    /* #ifdef H5 */ uni.$emit('tabbar-update'); /* #endif */
  },
  computed:{
    title(){
      const map={parking:'停车预约',scenic:'景点预约',restaurant:'餐馆预约'};
      return map[this.type]||'预约列表';
    }
  },
  methods:{
    loadList(){
      const all = uni.getStorageSync('BOOKINGS')||[];
      this.list = all.filter(i=>i.type===this.type && !i.isCancel);
    },
    cancelOne(item){
      const all = uni.getStorageSync('BOOKINGS')||[];
      const tar = all.find(i=>i.id===item.id);
      if(tar){
        tar.isCancel = true;
        uni.setStorageSync('BOOKINGS',all);
        this.loadList();          // 刷新当前页
        uni.showToast({title:'已取消',icon:'none'});
      }
    },
    clearType(){
      uni.showModal({
        title:'提示',
        content:`确定清空所有${this.title}记录？`,
        success:(res)=>{
          if(res.confirm){
            const all = uni.getStorageSync('BOOKINGS')||[];
            const rest = all.filter(i=>i.type!==this.type || i.isCancel); // 只留非当前类型或已取消
            uni.setStorageSync('BOOKINGS',rest);
            this.loadList();
          }
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
$pagePad:32rpx;
.bar{display:flex;justify-content:space-between;align-items:center;padding:$pagePad;}
.tit{font-size:34rpx;font-weight:600;color:#2c3e50;}
.clear{font-size:26rpx;color:#667eea;}
.list{padding:0 $pagePad;}
.item{display:flex;justify-content:space-between;align-items:center;padding:28rpx 0;border-bottom:1rpx solid #f0f0f0;}
.item:last-child{border-bottom:none;}
.name{font-size:30rpx;color:#2c3e50;margin-bottom:6rpx;}
.time{font-size:24rpx;color:#95a5a6;}
.status{padding:4rpx 10rpx;border-radius:8rpx;font-size:24rpx;}
.status.ok{background:#e8f5e9;color:#43a047;}
.cancel{font-size:24rpx;color:#667eea;margin-left:16rpx;}
.empty{text-align:center;padding:120rpx 0;color:#95a5a6;font-size:28rpx;}
/* 底部留安全区 */
.page{padding-bottom:calc(120rpx + env(safe-area-inset-bottom));}
</style>