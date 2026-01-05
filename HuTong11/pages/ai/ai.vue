<template>
  <view class="page">
    <!-- 顶部渐变标题 -->
    <view class="head-bar">
      <text class="head-title">旅行规划助手</text>
    </view>

    <!-- 聊天历史 -->
    <scroll-view class="chat-panel" scroll-y :scroll-top="scrollTop" scroll-with-animation>
      <!-- 欢迎语 -->
      <view class="msg system">
        <image class="avatar" src="/static/robot.png" mode="aspectFill" />
        <view class="bubble">
          <text class="txt">
            您好！我是您的旅游规划助手，会用热情和温柔为您打造一次难忘的旅行。告诉我时间、预算、想去的地方和特别需求，我会为您量身定制完美计划～
          </text>
        </view>
      </view>

      <!-- 历史消息 -->
      <view v-for="(m,i) in messages" :key="i" class="msg" :class="{user:m.isUser}">
        <image class="avatar" :src="m.isUser?'/static/user.png':'/static/robot.png'" mode="aspectFill" />
        <view class="bubble" :class="{userBubble:m.isUser}">
          <text class="txt">{{m.content}}</text>
          <!-- 机器人消息附加操作 -->
          <view v-if="!m.isUser" class="bubble-tool">
            <text class="tool-btn" @click="copyAns(m.content)">复制</text>
            <text class="tool-btn" @click="regenerate">重新生成</text>
          </view>
        </view>
      </view>

      <!-- 加载中 -->
      <view v-if="isLoading" class="msg system">
        <image class="avatar" src="/static/robot.png" mode="aspectFill" />
        <view class="bubble">
          <text class="txt">正在为您规划... ⏳</text>
        </view>
      </view>

      <!-- 错误提示 -->
      <view v-if="error" class="error-bar">❌ {{error}}</view>
    </scroll-view>

    <!-- 底部输入区 -->
    <view class="input-bar">
      <input v-model="userInput" class="input" placeholder="请输入或按住说话…" confirm-type="send"
             @confirm="sendMessage" :disabled="isLoading" />
      <!-- 语音按钮 - PC端隐藏 -->
      <view class="mic-wrap" :class="{recing}" @touchstart="onTouchStart" @touchend="onTouchEnd"
            @mousedown="onMouseDown" @mouseup="onMouseUp" @mouseleave="onMouseUp" v-if="!isPC">
        <view class="mic-core">{{recing?'🎤':'🎙'}}</view>
        <view class="mic-ring" :class="{pulse:recing}"></view>
      </view>
      <button class="send-btn" :disabled="!userInput.trim()||isLoading" @click="sendMessage">发送</button>
    </view>

    <!-- 自动朗读开关 -->
    <view class="switch-bar">
      <switch :checked="autoSpeak" @change="e=>autoSpeak=e.detail.value" color="#667eea" />
      <text class="switch-txt">自动朗读</text>
    </view>

    <!-- 录音提示 -->
    <view class="rec-tip" :class="{show:showRecTip}" v-if="!isPC">
      <text class="rec-tip-txt">🎤 松开发送</text>
    </view>

    <!-- H5 自定义 Tabbar -->
    <!-- #ifdef H5 -->
    <custom-tabbar v-if="!isPC"></custom-tabbar>
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
      userInput:'',
      messages:[],
      isLoading:false,
      error:'',
      scrollTop:0,
      autoSpeak:true,
      recing:false,
      showRecTip:false,
      useMouse:false,
      iflyPlugin:null,
      isPC:false, // 新增：判断是否为PC端
      // 请换成你自己的 key
      apiKey:'sk-NgLe1sjMOBVltWoYqlS02g2LFTbMbCb4YbMKFytLsp9CDhlI'
    }
  },
  onLoad(){
    // 检测设备类型
    this.checkDeviceType()
    
    // #ifdef APP-PLUS||MP-WEIXIN||MP-ALIPAY
    this.iflyPlugin=uni.requireNativePlugin('XFYun-Speech')
    this.iflyPlugin.init({appid:'f16425e2',secret:'NWYxNmQxYzRjNDBiMjJhMTY3YWM3Mzc2',key:'129a744cdabc4fce253751c83f7ce4e2'},()=>{})
    // #endif
  },
  onShow(){ 
    /* #ifdef H5 */ 
    uni.$emit('tabbar-update') 
    // 窗口大小变化时重新检测设备
    window.addEventListener('resize', this.checkDeviceType)
    /* #endif */ 
  },
  onHide(){
    /* #ifdef H5 */
    window.removeEventListener('resize', this.checkDeviceType)
    /* #endif */
  },
  methods:{
    // 检测设备类型（PC/移动）
    checkDeviceType(){
      /* #ifdef H5 */
      const isDesktop = window.innerWidth >= 768
      this.isPC = isDesktop
      /* #endif */
      /* #ifndef H5 */
      this.isPC = false
      /* #endif */
    },
    
    /* -------------- 语音 -------------- */
    onMouseDown(){this.useMouse=true;this.holdRec()},
    onMouseUp(){if(this.useMouse){this.stopRec();this.useMouse=false}},
    onTouchStart(){this.useMouse=false;this.holdRec()},
    onTouchEnd(){if(!this.useMouse)this.stopRec()},
    holdRec(){
      if(this.recing)return
      this.recing=true;this.showRecTip=true
      // #ifdef APP-PLUS||MP-WEIXIN||MP-ALIPAY
      this.iflyPlugin.startASR({language:'zh_cn',vadEnable:true},res=>{
        if(res.result?.length){this.userInput=res.result[0].words;this.sendMessage()}
      })
      // #endif
      // #ifdef H5
      if(!this.isPC && 'webkitSpeechRecognition' in window){
        const rec=new webkitSpeechRecognition();rec.lang='zh-CN';rec.interimResults=false;rec.maxAlternatives=1
        rec.onresult=e=>{this.userInput=e.results[0][0].transcript;this.sendMessage()}
        rec.onerror=()=>{uni.showToast({title:'识别失败',icon:'none'});this.recing=false;this.showRecTip=false}
        rec.onend=()=>{this.recing=false;this.showRecTip=false};rec.start()
      } else if(this.isPC) {
        uni.showToast({title:'PC端暂不支持语音输入',icon:'none'})
        this.recing=false;this.showRecTip=false
      } else {
        uni.showToast({title:'浏览器不支持语音识别',icon:'none'});this.recing=false;this.showRecTip=false
      }
      // #endif
    },
    stopRec(){
      // #ifdef APP-PLUS||MP-WEIXIN||MP-ALIPAY
      this.iflyPlugin.stopASR()
      // #endif
      this.recing=false;this.showRecTip=false
    },
    
    /* -------------- 发送 -------------- */
    sendMessage(){
      const input=this.userInput.trim()
      if(!input||this.isLoading)return
      this.messages.push({content:input,isUser:true})
      this.userInput='';this.isLoading=true;this.error='';this.scrollToBottom()
      uni.request({
        url:'https://api.moonshot.cn/v1/chat/completions',
        method:'POST',
        header:{'Content-Type':'application/json','Authorization':`Bearer ${this.apiKey}`},
        data:{
          model:'moonshot-v1-8k',
          messages:[
            {role:'system',content:'你是一位专业旅游规划助手，回答亲切、简洁、有条理，优先给出可执行的路线/预算/Tips。'},
            ...this.messages.map(m=>({role:m.isUser?'user':'assistant',content:m.content})),
            {role:'user',content:input}
          ],
          temperature:0.7,
          max_tokens:1200,
          stream:false
        },
        success:res=>{
          if(res.statusCode!==200){this.error='生成失败：HTTP '+res.statusCode;return}
          try{
            const ans=res.data.choices[0].message.content
            this.messages.push({content:ans,isUser:false})
            this.speak(ans)
          }catch(e){this.error='解析失败'}
        },
        fail:()=>{this.error='网络失败'},
        complete:()=>{this.isLoading=false;this.scrollToBottom()}
      })
    },
    
    /* -------------- 工具 -------------- */
    copyAns(txt){
      uni.setClipboardData({data:txt,success:()=>uni.showToast({title:'已复制',icon:'none'})})
    },
    regenerate(){
      const last=this.messages.filter(m=>m.isUser).pop()
      if(!last)return
      this.messages.splice(this.messages.indexOf(last)+1) // 删除后续机器人消息
      this.userInput=last.content
      this.sendMessage()
    },
    speak(text){
      if(!this.autoSpeak)return
      // #ifdef APP-PLUS||MP-WEIXIN||MP-ALIPAY
      this.iflyPlugin.startTTS({text,speed:50,pitch:50,volume:100})
      // #endif
      // #ifdef H5
      const u=new SpeechSynthesisUtterance(text);u.lang='zh-CN';speechSynthesis.speak(u)
      // #endif
    },
    scrollToBottom(){
      this.$nextTick(()=>this.scrollTop+=9999)
    }
  }
}
</script>

<style lang="scss" scoped>
$pagePad:32rpx;
$radius:24rpx;
$shadow:0 8rpx 24rpx rgba(102,126,234,.15);

// 响应式变量
$pc-min-width:768px;
$pc-max-bubble-width:60%;
$mobile-max-bubble-width:70%;

.page{
  display:flex;flex-direction:column;height:100vh;background:#f4f6fc;
  
  // 适配PC端居中显示
  @media (min-width: $pc-min-width) {
    max-width: 1200px;
    margin: 0 auto;
    box-shadow: 0 0 40rpx rgba(0,0,0,0.05);
  }

  // 顶部渐变头栏
  .head-bar{
    height:128rpx;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);
    display:flex;align-items:center;justify-content:center;
    
    // PC端标题栏优化
    @media (min-width: $pc-min-width) {
      height: 100rpx;
      border-radius: $radius $radius 0 0;
    }
    
    .head-title{
      font-size:36rpx;color:#fff;font-weight:600;letter-spacing:1rpx;
      
      @media (min-width: $pc-min-width) {
        font-size: 40rpx;
      }
    }
  }
  
  // 聊天面板
  .chat-panel{
    flex:1;padding:$pagePad;background:#f4f6fc;
    
    // PC端聊天面板优化
    @media (min-width: $pc-min-width) {
      padding: 40rpx 60rpx;
    }
    
    .msg{
      display:flex;margin-bottom:30rpx;align-items:flex-start;
      
      &.user{flex-direction:row-reverse;margin-right: 40rpx;}
      
      .avatar{
        width:72rpx;height:72rpx;border-radius:50%;background:#fff;flex-shrink:0;
        
        // PC端头像放大
        @media (min-width: $pc-min-width) {
          width: 80rpx;
          height: 80rpx;
          margin: 0 20rpx;
        }
      }
      
      .bubble{
        max-width: $mobile-max-bubble-width;
        background:#fff;border-radius:$radius;padding:24rpx 28rpx;
        font-size:30rpx;color:#2c3e50;line-height:1.6;
        box-shadow:$shadow;position:relative;
        
        // PC端气泡优化
        @media (min-width: $pc-min-width) {
          max-width: $pc-max-bubble-width;
          padding: 30rpx 36rpx;
          font-size: 32rpx;
          line-height: 1.7;
        }
        
        &.userBubble{
          background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:#fff;
        }
        
        .txt{word-break:break-all;}
        
        .bubble-tool{
          margin-top:16rpx;display:flex;font-size:24rpx;color:#a2a8c3;
          
          @media (min-width: $pc-min-width) {
            font-size: 26rpx;
            margin-top: 20rpx;
          }
          
          .tool-btn{
            margin-right:20rpx;cursor:pointer;
            
            &:hover{color:#667eea;}
            
            // PC端hover效果增强
            @media (min-width: $pc-min-width) {
              margin-right: 24rpx;
              transition: color 0.2s ease;
            }
          }
        }
      }
    }
    
    .system .bubble{
      background:linear-gradient(135deg,#e0e7ff 0%,#d8e0ff 100%);color:#2c3e50;
    }
    
    .error-bar{
      margin:0 32rpx 20rpx;background:#ffe5e5;color:#e53935;
      border-radius:12rpx;padding:16rpx 24rpx;font-size:28rpx;text-align:center;
      
      @media (min-width: $pc-min-width) {
        padding: 20rpx 30rpx;
        font-size: 30rpx;
        margin: 0 40rpx 24rpx;
      }
    }
  }
  
  // 底部输入栏
  .input-bar{
    position:fixed;left:0;right:0;
    bottom:calc(120rpx + env(safe-area-inset-bottom));
    bottom:calc(120rpx + constant(safe-area-inset-bottom));
    display:flex;align-items:center;gap:16rpx;
    padding:20rpx 32rpx;background:#fff;box-shadow:$shadow;
    border-radius:$radius $radius 0 0;
    
    // PC端输入栏优化
    @media (min-width: $pc-min-width) {
      position: static;
      margin: 0 $pagePad $pagePad;
      border-radius: $radius;
      padding: 24rpx 40rpx;
      bottom: auto;
      box-shadow: 0 4rpx 20rpx rgba(102,126,234,.1);
    }
    
    .input{
      flex:1;height:80rpx;background:#f4f6fc;border-radius:40rpx;
      padding:0 32rpx;font-size:30rpx;color:#2c3e50;
      
      // PC端输入框优化
      @media (min-width: $pc-min-width) {
        height: 90rpx;
        font-size: 32rpx;
        padding: 0 40rpx;
        border-radius: 45rpx;
      }
    }
    
    .send-btn{
      width:120rpx;height:80rpx;border-radius:40rpx;
      background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);
      color:#fff;font-size:30rpx;display:flex;
      align-items:center;justify-content:center;border:none;
      
      // PC端发送按钮优化
      @media (min-width: $pc-min-width) {
        width: 140rpx;
        height: 90rpx;
        border-radius: 45rpx;
        font-size: 32rpx;
        cursor: pointer;
        transition: opacity 0.2s ease;
      }
      
      &:disabled{opacity:.5;}
    }
    
    // 语音按钮
    .mic-wrap{
      position:relative;width:80rpx;height:80rpx;margin-left:8rpx;
      
      .mic-core{
        width:100%;height:100%;border-radius:50%;
        background:#ff7043;color:#fff;font-size:32rpx;
        display:flex;align-items:center;justify-content:center;
      }
      
      .mic-ring{
        position:absolute;inset:0;border-radius:50%;
        border:4rpx solid #ff7043;opacity:0;
      }
      
      &.recing .mic-ring{animation:pulse 1.2s infinite;}
    }
    
    @keyframes pulse{
      0%{transform:scale(1);opacity:.6;}
      70%{transform:scale(1.5);opacity:0;}
      100%{transform:scale(1);opacity:0;}
    }
  }
  
  // 自动朗读开关
  .switch-bar{
    display:flex;align-items:center;justify-content:center;
    padding:16rpx 0;font-size:28rpx;color:#a2a8c3;
    
    @media (min-width: $pc-min-width) {
      padding: 20rpx 0 10rpx;
      font-size: 30rpx;
    }
    
    .switch-txt{margin-left:8rpx;}
  }
  
  // 录音提示
  .rec-tip{
    position:fixed;left:50%;bottom:calc(300rpx + 50rpx);
    transform:translate(-50%,20rpx);opacity:0;
    background:rgba(0,0,0,.7);color:#fff;font-size:28rpx;
    padding:16rpx 32rpx;border-radius:30rpx;transition:all .3s;
    
    &.show{
      transform:translate(-50%,0);opacity:1;
    }
  }
}

// PC端整体样式优化
@media (min-width: $pc-min-width) {
  page {
    background-color: #f0f2f5;
  }
  
  ::v-deep .uni-scroll-view {
    overflow-y: auto;
    &::-webkit-scrollbar {
      width: 8rpx;
    }
    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      border-radius: 4rpx;
    }
    &::-webkit-scrollbar-thumb {
      background: #d1d9e6;
      border-radius: 4rpx;
      &:hover {
        background: #b1b9c6;
      }
    }
  }
}
</style>