<template>
  <view class="content">
    <div class="g-container">
      <div class="g-number">{{ chargeInfo.level }} %</div>
      <div class="g-contrast">
        <div class="g-circle"></div>
        <ul class="g-bubbles">
          <li v-for="i in 15"></li>
        </ul>
      </div>
    </div>
  </view>
</template>

<script setup lang="ts">
import { watch, type Ref } from 'vue'
import { onBeforeUnmount, onMounted, ref } from 'vue'

const chargeInfo = ref({
  errMsg: 'getBatteryInfo:ok',
  isCharging: true,
  level: 100
})

const lastLevel = ref(0)

const interValId = ref()
/* 获取电池信息 */
const getBatteryInfo = () => {
  uni.getBatteryInfo({
    success: res => {
      chargeInfo.value = res
    }
  })
}

/* 发送信息 */
const postMessage = () => {
  const params = {
    appToken: 'AT_WWE4AafjqlQHgFVV24NigV1yjX8OUFAk',
    content: `当前充电状态：${chargeInfo.value.isCharging ? '充电中' : '未充电'}\n当前电量：${chargeInfo.value.level}%`,
    summary: `当前电量：${chargeInfo.value.level}%`, //消息摘要，显示在微信聊天页面或者模版消息卡片上
    contentType: 1, //内容类型 1表示文字  2表示html(只发送body标签内部的数据即可，不包括body标签) 3表示markdown
    uids: [
      //发送目标的UID，是一个数组。注意uids和topicIds可以同时填写，也可以只填写一个。
      'UID_Pvc2cOcPOljmRhZk7mdXO3xbIiiI'
    ],
    url: 'https://wxpusher.zjiecode.com', //原文链接，可选参数
    verifyPay: false //是否验证订阅时间，true表示只推送给付费订阅用户，false表示推送的时候，不验证付费，不验证用户订阅到期时间，用户订阅过期了，也能收到。
  }
  uni.request({
    url: 'https://wxpusher.zjiecode.com/api/send/message',
    method: 'POST',
    data: params
  })
}

/* 监听电池信息 */
watch(chargeInfo, (newValue, oldValue) => {
  const { level } = newValue
  if (level < 80 && level - lastLevel.value >= 10) {
    postMessage()
    lastLevel.value = level
    return
  }
  if (level >= 80 && lastLevel.value < 80) {
    postMessage()
    lastLevel.value = level
    return
  }
  if (level >= 80 && level - lastLevel.value >= 5) {
    postMessage()
    lastLevel.value = level
  }
})

/* 钩子函数 */
onMounted(() => {
  getBatteryInfo()
  if (interValId.value) {
    clearInterval(interValId.value)
    interValId.value = ''
  }
  interValId.value = setInterval(getBatteryInfo, 60000)
})

// onBeforeUnmount(() => {
//   clearInterval(interValId.value)
// })
</script>

<style lang="less">
.content {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  background: #000;
  height: calc(100vh);
}

.g-container {
  position: relative;
  width: 300px;
  height: 400px;
}

.g-number {
  position: absolute;
  width: 300px;
  top: 27%;
  text-align: center;
  font-size: 32px;
  z-index: 10;
  color: #fff;
}

.g-contrast {
  filter: contrast(10) hue-rotate(0);
  width: 300px;
  height: 400px;
  background-color: #000;
  overflow: hidden;
  animation: hueRotate 10s infinite linear;
}

.g-circle {
  position: relative;
  width: 300px;
  height: 300px;
  box-sizing: border-box;
  filter: blur(8px);
  &::after {
    content: '';
    position: absolute;
    top: 40%;
    left: 50%;
    transform: translate(-50%, -50%) rotate(0);
    width: 200px;
    height: 200px;
    background-color: #00ff6f;
    border-radius: 42% 38% 62% 49% / 45%;
    animation: rotate 10s infinite linear;
  }

  &::before {
    content: '';
    position: absolute;
    width: 176px;
    height: 176px;
    top: 40%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 50%;
    background-color: #000;
    z-index: 10;
  }
}

.g-bubbles {
  position: absolute;
  left: 50%;
  bottom: 0;
  width: 100px;
  height: 40px;
  transform: translate(-50%, 0);
  border-radius: 100px 100px 0 0;
  background-color: #00ff6f;
  filter: blur(5px);
  li {
    position: absolute;
    border-radius: 50%;
    background: #00ff6f;
  }
}

.loop(@n) when (@n > 0) {
  .loop(@n - 1);
  li:nth-child(@{n}) {
    left: ~`Math.floor(Math.random() * 95) + 15+'px' `;
    top: 50%;
    transform: translate(-50%, -50%);
    @width: ~`Math.floor(Math.random() * 15) + 15+'px' `;
    width: @width;
    height: @width;
    @s: ~`Math.random() `;
    animation: moveToTop @s*6+3s ease-in-out @s*5s infinite;
  }
}
.loop(15);

@keyframes rotate {
  50% {
    border-radius: 45% / 42% 38% 58% 49%;
  }
  100% {
    transform: translate(-50%, -50%) rotate(720deg);
  }
}

@keyframes moveToTop {
  90% {
    opacity: 1;
  }
  100% {
    opacity: 0.1;
    transform: translate(-50%, -180px);
  }
}

@keyframes hueRotate {
  100% {
    filter: contrast(15) hue-rotate(360deg);
  }
}
</style>
