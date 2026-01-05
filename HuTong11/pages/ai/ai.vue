<template>
  <view class="container">
    <!-- PC 端背景装饰 -->
    <view class="pc-bg" v-if="isPC"></view>

    <view class="page-content">
      <!-- 顶部标题栏 -->
      <view class="head-bar">
        <text class="head-title">旅行规划助手</text>
        <view class="status-dot" :class="{processing: isLoading}"></view>
      </view>

      <!-- 聊天滚动区 -->
      <scroll-view
        class="chat-scroll"
        scroll-y
        :scroll-top="scrollTop"
        :scroll-into-view="scrollIntoViewId"
        scroll-with-animation
      >
        <view class="chat-inner">
          <!-- 欢迎语 -->
          <view class="msg system">
            <image class="avatar" src="/static/robot.png" mode="aspectFill" />
            <view class="bubble">
              <text class="txt">您好！我是您的 AI 向导。告诉我时间、预算和目的地，我为您规划完美旅程～ ✈️</text>
            </view>
          </view>

          <!-- 消息列表 -->
          <view v-for="(m,i) in messages" :key="i" class="msg" :class="{user: m.isUser}" :id="'msg-'+i">
            <image class="avatar" :src="m.isUser?'/static/user.png':'/static/robot.png'" mode="aspectFill" />
            <view class="bubble" :class="{userBubble: m.isUser}">
              <text class="txt" user-select>{{m.content}}</text>
              <!-- 机器人工具栏 -->
              <view v-if="!m.isUser" class="bubble-tool">
                <text class="tool-icon" @click="copyAns(m.content)">📄 复制</text>
                <text class="tool-icon" @click="regenerate">🔄 重试</text>
              </view>
            </view>
          </view>

          <!-- 加载状态 -->
          <view v-if="isLoading" class="msg system loading-msg">
            <image class="avatar" src="/static/robot.png" mode="aspectFill" />
            <view class="bubble typing">
              <view class="dot"></view><view class="dot"></view><view class="dot"></view>
            </view>
          </view>

          <!-- 错误提示 -->
          <view v-if="error" class="error-tip" @click="regenerate">{{error}} (点击重试)</view>

          <!-- 底部占位 -->
          <view class="bottom-spacer" id="bottom-anchor"></view>
        </view>
      </scroll-view>

      <!-- 底部输入区 -->
      <view class="footer-area">
        <view class="rec-toast" :class="{show: recing}">
          <view class="wave-box">
            <view class="wave" v-for="i in 5" :key="i"></view>
          </view>
          <text>{{isPC ? '正在聆听...' : '松开结束'}}</text>
        </view>

        <view class="input-bar">
          <view
            class="action-btn mic-btn"
            :class="{active: recing}"
            @mousedown="onMouseDown"
            @mouseup="onMouseUp"
            @mouseleave="onMouseUp"
            @touchstart.stop.prevent="onTouchStart"
            @touchend.stop.prevent="onTouchEnd"
          >
            <text class="icon">{{recing ? '🎙️' : '🎤'}}</text>
          </view>

          <input
            v-model="userInput"
            class="main-input"
            :placeholder="recing ? '正在说话...' : '输入目的地、预算...'"
            confirm-type="send"
            @confirm="sendMessage"
            :disabled="isLoading"
          />

          <button
            class="action-btn send-btn"
            :disabled="!userInput.trim() || isLoading"
            @click="sendMessage"
          >
            <text class="icon">🚀</text>
          </button>
        </view>
      </view>

      <!-- 原生 TabBar 安全区占位 -->
      <!-- #ifdef H5 -->
      <view class="tabbar-placeholder" v-if="!isPC"></view>
      <!-- #endif -->
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInput: '',
      messages: [],
      isLoading: false,
      error: '',
      scrollTop: 0,
      scrollIntoViewId: '',
      recing: false,
      useMouse: false,
      isPC: false,
      iflyPlugin: null,
      apiKey: 'sk-NgLe1sjMOBVltWoYqlS02g2LFTbMbCb4YbMKFytLsp9CDhlI'
    }
  },
  onLoad() {
    this.checkDevice();
    // #ifdef APP-PLUS
    this.initAppSpeech();
    // #endif
  
    /* 读取一键规划带来的提示句 */
    const prompt = uni.getStorageSync('AI_PROMPT');
    if (prompt) {
      this.userInput = prompt;
      uni.removeStorageSync('AI_PROMPT'); // 用完即删
      this.$nextTick(() => this.sendMessage());
    }
  },
  mounted() {
    // #ifdef H5
    window.addEventListener('resize', this.checkDevice);
    // #endif
  },
  beforeDestroy() {
    // #ifdef H5
    window.removeEventListener('resize', this.checkDevice);
    // #endif
  },
  methods: {
    checkDevice() {
      // #ifdef H5
      this.isPC = window.innerWidth >= 768;
      // #endif
    },
    initAppSpeech() {
      // #ifdef APP-PLUS
      const plugin = uni.requireNativePlugin('XFYun-Speech');
      if (plugin) {
        this.iflyPlugin = plugin;
        this.iflyPlugin.init({
          appid: 'f16425e2',
          secret: 'NWYxNmQxYzRjNDBiMjJhMTY3YWM3Mzc2',
          key: '129a744cdabc4fce253751c83f7ce4e2'
        }, () => {});
      }
      // #endif
    },
    onMouseDown() { this.useMouse = true; this.startRec(); },
    onMouseUp() { if (this.useMouse) { this.stopRec(); this.useMouse = false; } },
    onTouchStart() { this.useMouse = false; this.startRec(); },
    onTouchEnd() { if (!this.useMouse) this.stopRec(); },
    startRec() {
      if (this.recing) return;
      this.recing = true;
      // #ifdef APP-PLUS
      if (this.iflyPlugin) {
        this.iflyPlugin.startASR({ language: 'zh_cn', vadEnable: true }, res => {
          if (res.result?.length) { this.userInput = res.result[0].words; this.sendMessage(); }
        });
        return;
      }
      // #endif
      // #ifdef H5
      if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        this.recognition = new SpeechRecognition();
        this.recognition.lang = 'zh-CN';
        this.recognition.interimResults = false;
        this.recognition.onresult = (event) => {
          const transcript = event.results[0][0].transcript;
          this.userInput = transcript;
          this.sendMessage();
        };
        this.recognition.onerror = () => {
          uni.showToast({ title: '语音识别需HTTPS或本地环境', icon: 'none' });
          this.recing = false;
        };
        this.recognition.onend = () => { this.recing = false; };
        this.recognition.start();
      } else {
        uni.showToast({ title: '当前浏览器不支持语音', icon: 'none' });
        this.recing = false;
      }
      // #endif
    },
    stopRec() {
      // #ifdef APP-PLUS
      if (this.iflyPlugin) this.iflyPlugin.stopASR();
      // #endif
      // #ifdef H5
      if (this.recognition) this.recognition.stop();
      // #endif
    },
    sendMessage() {
      const content = this.userInput.trim();
      if (!content || this.isLoading) return;
      this.messages.push({ content, isUser: true });
      this.userInput = '';
      this.isLoading = true;
      this.error = '';
      this.scrollToBottom();
      uni.request({
        url: 'https://api.moonshot.cn/v1/chat/completions',
        method: 'POST',
        header: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.apiKey}`
        },
        data: {
          model: 'moonshot-v1-8k',
          messages: [
            { role: 'system', content:
              `你是一位专业、热情、排版精美的旅游规划师。
               回答要求：
               1. 禁止使用任何 Markdown 符号（如 # * _ ~ [] () 等）。
               2. 可用 emoji、数字、常见标点。
               3. 分点回答请用“①②③”或“1. 2. 3.”。
               4. 给出可执行的路线/预算/交通建议。
			   5. 用户问道距离优先顺序时，请自行编他们的距离地址位置，并给出合理的最短路径规划
			   6. 描述尽量简单明白`
            },
            ...this.messages.map(m => ({ role: m.isUser ? 'user' : 'assistant', content: m.content })),
            { role: 'user', content }
          ],
          temperature: 0.7
        },
        success: (res) => {
          if (res.statusCode !== 200) { this.error = '服务繁忙，请稍后再试'; return; }
          const replyRaw = res.data.choices?.[0]?.message?.content ?? '';
          const reply = this.stripMarkdown(replyRaw);
          if (reply) {
            this.messages.push({ content: reply, isUser: false });
            this.speak(reply);
          } else {
            this.error = '返回数据异常';
          }
        },
        fail: () => { this.error = '网络连接失败'; },
        complete: () => { this.isLoading = false; this.scrollToBottom(); }
      });
    },
    regenerate() {
      const lastUserIdx = this.messages.map(m => m.isUser).lastIndexOf(true);
      if (lastUserIdx === -1) return;
      const lastMsg = this.messages[lastUserIdx].content;
      this.messages = this.messages.slice(0, lastUserIdx);
      this.userInput = lastMsg;
      this.sendMessage();
    },
    copyAns(text) {
      uni.setClipboardData({ data: text, success: () => uni.showToast({ title: '已复制', icon: 'none' }) });
    },
    speak(text) {
      const shortText = text.substring(0, 100).replace(/[#*`~_\[\]()]/g, '');
      // #ifdef H5
      if ('speechSynthesis' in window) {
        window.speechSynthesis.cancel();
        const u = new SpeechSynthesisUtterance(shortText);
        u.lang = 'zh-CN';
        window.speechSynthesis.speak(u);
      }
      // #endif
      // #ifdef APP-PLUS
      if (this.iflyPlugin) this.iflyPlugin.startTTS({ text: shortText });
      // #endif
    },
    scrollToBottom() {
      this.$nextTick(() => {
        this.scrollIntoViewId = 'bottom-anchor';
        this.scrollTop += 9999;
      });
    },
    stripMarkdown(str) {
      return str
        .replace(/[#*`~_\[\]()]/g, '')
        .replace(/^(\s*)[\d①②③④⑤⑥⑦⑧⑨⑩]+[\.、]\s*/gm, '$1')
        .trim();
    }
  }
};
</script>

<style lang="scss" scoped>
/* 核心变量 */
$primary: #667eea;
$primary-grad: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
$bg-color: #f4f7fc;
$chat-bg: #37d8ea;
$radius-box: 24rpx;
$radius-bubble: 20rpx;

/* 全局容器：使用Flex Column实现不遮挡布局 */
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
  background-color: $bg-color;
  position: relative;
  overflow: hidden;
}

/* PC背景装饰 */
.pc-bg {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  background: linear-gradient(180deg, #dce4f7 0%, #f4f7fc 50%);
  z-index: 0;
}

/* 页面主体内容区 */
.page-content {
  position: relative; z-index: 1;
  flex: 1; display: flex; flex-direction: column; height: 100%;
  @media (min-width: 768px) {
    max-width: 900px; height: 90vh; margin: 5vh auto;
    background: #fff; border-radius: 20px; box-shadow: 0 10px 40px rgba(0,0,0,0.1);
    overflow: hidden;
  }
}

/* 1. 头部 */
.head-bar {
  height: 100rpx; background: $chat-bg;
  border-bottom: 1rpx solid rgba(0,0,0,0.05);
  display: flex; align-items: center; justify-content: center; flex-shrink: 0;
  position: relative;
  @media (min-width: 768px) { background: $primary-grad; .head-title { color: #fff !important; } }
  .head-title { font-size: 34rpx; font-weight: 600; color: #333; }
  .status-dot {
    position: absolute; right: 40rpx; width: 16rpx; height: 16rpx; border-radius: 50%; background: #4caf50;
    &.processing { background: #ff9800; animation: blink 1s infinite; }
  }
}

/* 2. 聊天区域 */
.chat-scroll {
  flex: 1; height: 0;
  background: $bg-color;
  @media (min-width: 768px) { background: #fff; }
  .chat-inner { padding: 30rpx; display: flex; flex-direction: column; }
}

/* 消息气泡 */
.msg {
  display: flex; margin-bottom: 40rpx; align-items: flex-start;
  &.user { flex-direction: row-reverse; }
  .avatar { width: 80rpx; height: 80rpx; border-radius: 50%; background: #fff; margin: 0 20rpx; box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.05); }
  .bubble {
    max-width: 70%; padding: 24rpx 30rpx; background: #fff; border-radius: 4rpx $radius-bubble $radius-bubble $radius-bubble;
    box-shadow: 0 4rpx 16rpx rgba(0,0,0,0.04); font-size: 30rpx; line-height: 1.6; color: #333;
    &.userBubble { background: $primary-grad; color: #fff; border-radius: $radius-bubble 4rpx $radius-bubble $radius-bubble; }
    .tool-icon { font-size: 24rpx; color: #999; margin-right: 20rpx; display: inline-block; margin-top: 10rpx; cursor: pointer; }
  }
}

/* 加载动画 */
.loading-msg .bubble.typing {
  padding: 20rpx 30rpx; display: flex; align-items: center; gap: 8rpx;
  .dot { width: 12rpx; height: 12rpx; background: #ccc; border-radius: 50%; animation: bounce 1.4s infinite ease-in-out both; }
}

/* 3. 底部操作区 */
.footer-area { flex-shrink: 0; background: #fff; box-shadow: 0 -4rpx 20rpx rgba(0,0,0,0.03); position: relative; z-index: 10; }
.input-bar {
  display: flex; align-items: center; padding: 20rpx 30rpx; gap: 20rpx;
  @media (min-width: 768px) { padding: 30rpx 40rpx; }
  .main-input {
    flex: 1; height: 80rpx; background: #f4f6f8; border-radius: 40rpx; padding: 0 32rpx; font-size: 30rpx;
    &:focus { background: #fff; box-shadow: 0 0 0 2rpx $primary; }
  }
  .action-btn { width: 80rpx; height: 80rpx; border-radius: 50%; display: flex; align-items: center; justify-content: center; border: none; outline: none; padding: 0; }
  .mic-btn { background: #f0f2f5; color: #666; font-size: 36rpx; cursor: pointer; user-select: none; &.active { background: #ffebee; color: #f44336; animation: pulse 1.5s infinite; } }
  .send-btn { background: $primary-grad; color: #fff; font-size: 32rpx; &[disabled] { opacity: 0.5; filter: grayscale(1); } }
}

/* 录音波纹提示 */
.rec-toast {
  position: absolute; bottom: 100%; left: 0; right: 0; height: 120rpx;
  background: rgba(255,255,255,0.95); backdrop-filter: blur(5px);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  transform: translateY(20rpx); opacity: 0; visibility: hidden; transition: all 0.3s; border-top: 1rpx solid #eee;
  &.show { transform: translateY(0); opacity: 1; visibility: visible; }
  .wave-box { display: flex; align-items: center; height: 40rpx; gap: 6rpx; margin-bottom: 10rpx;
    .wave { width: 6rpx; height: 100%; background: #f44336; border-radius: 4rpx; animation: wave 0.5s infinite ease-in-out alternate; }
  }
  text { font-size: 24rpx; color: #666; }
}

/* H5导航栏占位 */
.tabbar-placeholder { height: 100rpx; flex-shrink: 0; background: #fff; @media (min-width: 768px) { display: none; } }
.bottom-spacer { height: 20rpx; width: 100%; }
.error-tip { text-align: center; color: #f44336; font-size: 24rpx; padding: 20rpx; cursor: pointer; }

/* 动画 */
@keyframes blink { 50% { opacity: 0.5; } }
@keyframes bounce { 0%, 80%, 100% { transform: scale(0); } 40% { transform: scale(1); } }
@keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(244, 67, 54, 0.4); } 70% { box-shadow: 0 0 0 20rpx rgba(244, 67, 54, 0); } 100% { box-shadow: 0 0 0 0 rgba(244, 67, 54, 0); } }
@keyframes wave { 0% { height: 20%; } 100% { height: 100%; } }
</style>