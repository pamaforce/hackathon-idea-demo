<template>
  <main class="main">
    <img src="./assets/qq_top.png" class="top" draggable="false">
    <img src="./assets/qq_message.png" class="message" v-if="showMessage">
    <img src="./assets/qq_pat.png" class="pat" v-if="showPat && !showOk" />
    <img src="./assets/qq_bottom.png"
      :class="{ 'bottom': true, 'bottom_move': showResult, 'bottom_move_2': showOk, 'bottom_move_3': showOver }">
    <img src="./assets/sdk_tip.png" class="tip" v-if="showTip">
    <canvas v-if="showCanvas" ref="canvas" class="canvas"></canvas>
    <div class="scan" v-if="showScan && !showOk"></div>
    <img src="./assets/left_bottom.svg" class="l_b" v-if="showScan && !showOk">
    <img src="./assets/right_top.svg" class="r_t" v-if="showScan && !showOk">
    <div :class="{ 'scan_result': true, 'scan_ok': showOk, 'scan_over': showOver }" v-if="showResult">
      <img src="./assets/siri.png" @click="changeActive" :class="{
        'siri': true,
        'siri_inactive': isInactive
      }" v-if="!hasVoice && !showOk" />
      <img src="./assets/input.png" class="input" v-if="isInactive && !hasVoice" />
      <div class="recommend" v-if="isInactive && !hasVoice">
        <div class="recommend_item" v-for="(item, i) in recommendList" :key="i" @click="setOkState"
          :style="{ animationDelay: `${1 + i * 0.3}s`, backgroundImage: `url(${getImageUrl(i)})` }">{{ item }}</div>
      </div>
      <img src="./assets/settings.png" class="settings" v-if="isInactive && !hasVoice">
      <div class="text" v-if="hasVoice">{{ text }}</div>
      <div class="bar_group" v-if="hasVoice">
        <div v-for="i in 11" :key="i" class="bar" :style="{
          height: `${heights[i - 1]}px`,
        }"></div>
      </div>
      <img src="./assets/close.png" class="close" v-if="hasVoice" />
      <img src="./assets/ok.png" class="ok" v-if="showOk && !showOver"></img>
      <div class="countdown" v-if="showOk && !showOver">{{ `${countdown}s` }}</div>
    </div>
    <button id="fullscreenButton" class="x" @click.stop="goFullScreen" v-if="showFullscreenButton">进入全屏</button>
  </main>
</template>

<script setup>
import { nextTick, onMounted, ref, watch } from 'vue';

let showMessage = ref(false);
let showPat = ref(false);
let showTip = ref(false);
let showCanvas = ref(false);
let showScan = ref(false);
let showResult = ref(false);
let isInactive = ref(false);
let hasVoice = ref(false);
let showOk = ref(false);
let showOver = ref(false);
let step = ref(0);
let finalText = '我想要关闭头像双击动作...'
let text = ref('');
let heights = ref([4, 10, 20, 10, 30, 36, 30, 10, 20, 10, 4]);
let countdown = ref(4);
let showFullscreenButton = ref(true);

let recommendList = ref(['头像双击动作', '深色模式', '字体大小', '颜色', '更多']);
const handleClick = () => {
  step.value++;
  showFullscreenButton.value = false;
  if (step.value === 1) {
    showMessage.value = true;
  } else if (step.value === 2) {
    showPat.value = true;
  } else if (step.value === 3) {
    showTip.value = true;
    showCanvas.value = true;
  }
};

const canvas = ref(null);
let isDrawing = false;
let context = null;
let lastX = 0;
let lastY = 0;

const startDrawing = (event) => {
  showTip.value = false;
  isDrawing = true;
  [lastX, lastY] = [event.clientX, event.clientY];
};

const draw = (event) => {
  if (!isDrawing) return;
  context.beginPath();
  context.moveTo(lastX, lastY);
  const [newX, newY] = [event.clientX, event.clientY];
  context.quadraticCurveTo(lastX, lastY, newX, newY);
  context.stroke();
  [lastX, lastY] = [newX, newY];
};

const stopDrawing = () => {
  isDrawing = false;
  showCanvas.value = false;
  setTimeout(() => {
    showScan.value = true;
    setTimeout(() => {
      showResult.value = true;
      setTimeout(() => {
        showVoiceResult();
      }, 2000);
    }, 500);
  }, 100);
};

const changeActive = () => {
  isInactive.value = !isInactive.value;
};

const getImageUrl = (index) => {
  switch (index) {
    case 0:
      return new URL(`./assets/text_bg.png`, import.meta.url).href;
    case 1:
    case 2:
      return new URL(`./assets/text_bg_2.png`, import.meta.url).href;
    case 3:
    case 4:
    default:
      return new URL(`./assets/text_bg_3.png`, import.meta.url).href;
  }
};
const setOkState = () => {
  hasVoice.value = false;
  isInactive.value = false;
  showOk.value = true;
  countdown.value = 3;
  let timerId = setInterval(() => {
    if (countdown.value > 0) {
      countdown.value--;
    } else {
      clearInterval(timerId);
      showOver.value = true;
    }
  }, 1000);
};
const showVoiceResult = () => {
  if (isInactive.value) return;
  text.value = '';
  hasVoice.value = true;
  intervalId = setInterval(updateHeights, 150);
  setTimeout(() => {
    const textArray = finalText.split('');
    let i = 0;
    const timer = setInterval(() => {
      text.value += textArray[i];
      i++;
      if (i === textArray.length) {
        clearInterval(timer);
        setTimeout(() => { clearInterval(intervalId) }, 1500)
        setTimeout(() => {
          setOkState();
        }, 1800);
      }
    }, 150);
  }, 300)
};

const maxCenterValue = 36;
const fluctuationRange = 14;
let intervalId;
const updateHeights = () => {
  const centerIndex = Math.floor(heights.value.length / 2);
  for (let i = 0; i < heights.value.length; i++) {
    const distanceFromCenter = Math.abs(centerIndex - i);
    const maxPossibleValue = maxCenterValue - distanceFromCenter * (maxCenterValue / centerIndex);
    heights.value[i] = Math.max(0, maxPossibleValue + randomFluctuation(fluctuationRange));
  }
}

const randomFluctuation = (range) => {
  return Math.random() * range - range / 2;
}

watch(isInactive, (newVal) => {
  if (!newVal) {
    setTimeout(() => {
      if (hasVoice.value) showVoiceResult();
    }, 2000);
  }
});
watch(showCanvas, (newVal) => {
  if (newVal) {
    nextTick(() => {
      const canvasElement = canvas.value;
      canvasElement.width = window.innerWidth;
      canvasElement.height = window.innerHeight;
      context = canvasElement.getContext('2d');
      context.strokeStyle = 'gray';
      context.lineWidth = 3;
      context.lineCap = 'round';
      context.lineJoin = 'round';
      canvasElement.addEventListener('mousedown', startDrawing);
      canvasElement.addEventListener('mousemove', draw);
      canvasElement.addEventListener('mouseup', stopDrawing);
      canvasElement.addEventListener('mouseleave', stopDrawing);
      canvasElement.addEventListener('touchstart', (e) => startDrawing(e.touches[0]));
      canvasElement.addEventListener('touchmove', (e) => draw(e.touches[0]));
      canvasElement.addEventListener('touchend', stopDrawing);
    });
  }
});
const goFullScreen = () => {
  if (document.documentElement.requestFullscreen) {
    document.documentElement.requestFullscreen().then(() => {
      showFullscreenButton.value = false;
    });
  } else if (document.documentElement.mozRequestFullScreen) { // Firefox
    document.documentElement.mozRequestFullScreen().then(() => {
      showFullscreenButton.value = false;
    });
  } else if (document.documentElement.webkitRequestFullscreen) { // Chrome, Safari and Opera
    document.documentElement.webkitRequestFullscreen().then(() => {
      showFullscreenButton.value = false;
    });
  } else if (document.documentElement.msRequestFullscreen) { // IE/Edge
    document.documentElement.msRequestFullscreen().then(() => {
      showFullscreenButton.value = false;
    });
  }
}
onMounted(() => {
  window.addEventListener('click', handleClick);
  document.addEventListener('fullscreenchange', function () {
    if (!document.fullscreenElement) {
      showFullscreenButton.value = true;
    }
  });
});
</script>

<style scoped>
.main {
  --width: 475px;
  width: var(--width);
  height: 100vh;
  background-color: #f1f1f1;
  user-select: none;
  position: relative;
  margin: 0 auto;
  overflow: hidden;
}

.top {
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

.message {
  width: 42.4%;
  position: absolute;
  right: 0;
  top: calc(0.2292 * var(--width));
  animation: rightIn 0.2s forwards;
}

.pat {
  position: absolute;
  width: 26.24%;
  top: calc(0.43 * var(--width));
  left: 50%;
  transform: translateX(-50%);
}

.bottom {
  width: var(--width);
  position: fixed;
  bottom: 0;
  transition: all 0.5s;
  left: calc(50% - var(--width) / 2);
}

.tip {
  width: var(--width);
  position: absolute;
  bottom: 0;
  left: 0;
  animation: fadeIn 0.5s forwards;
}

.canvas {
  position: fixed;
  top: 0;
  left: 0;
}

.scan {
  position: absolute;
  width: 26.24%;
  top: calc(0.43 * var(--width));
  height: 30px;
  background: radial-gradient(828.55% 122.94% at 99.91% 105.21%, rgba(70, 154, 251, 0.20) 27%, rgba(255, 255, 255, 0.00) 100%),
    radial-gradient(383.99% 127.33% at 11.2% 15.63%, rgba(155, 168, 255, 0.20) 32%, rgba(255, 255, 255, 0.00) 100%);
  left: 50%;
  transform: translateX(-50%);
}

.l_b {
  position: absolute;
  left: calc(50% - 13.12% + 4px);
  top: calc(0.43 * var(--width) + 30px - 10px);
  transform: translateX(-50%);
  width: 14px;
  height: 14px;
  transition: margin 0.5s;
  animation: leftCornerIn 0.5s forwards;
}

.r_t {
  position: absolute;
  right: calc(50% - 13.12% - 10px);
  top: calc(0.43 * var(--width) - 4px);
  transform: translateX(-50%);
  width: 14px;
  height: 14px;
  transition: margin 0.5s;
  animation: rightCornerIn 0.5s forwards;
}

.scan_result {
  position: fixed;
  width: var(--width);
  height: 150px;
  background: linear-gradient(180deg, #F6ADFD -65.48%, #D7EFFE 100%);
  left: calc(50% - var(--width) / 2);
  bottom: 0;
  animation: topIn 0.5s forwards;
  transition: all 0.5s;
}

.bottom_move {
  bottom: 150px;
}

.siri {
  position: absolute;
  width: 80px;
  height: 80px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0);
  animation: siriIn 0.5s 0.5s forwards;
  transition: all 0.5s;
}

.siri_inactive {
  width: 40px;
  height: 40px;
  left: 40px;
  top: 80px;
}

.input {
  position: absolute;
  width: 0;
  height: 32px;
  left: 70px;
  top: 64px;
  animation: inputIn 0.5s 0.5s forwards;
}

.recommend {
  display: flex;
  margin-top: 18px;
  margin-left: 20px;
}

.recommend_item {
  flex-shrink: 0;
  background-image: url('./assets/text_bg.png');
  background-size: 100% 100%;
  background-repeat: no-repeat;
  display: flex;
  padding: 5px 12px;
  justify-content: center;
  align-items: center;
  font-size: 12px;
  color: white;
  margin-right: 14px;
  transform: translateX(100%);
  opacity: 0;
  animation: itemIn 0.5s forwards;
}

.settings {
  position: absolute;
  left: 20px;
  width: calc(100% - 40px);
  bottom: 14px;
  opacity: 0;
  transform: translateY(100%);
  animation: bottomUp 0.5s 1s forwards;
}

.text {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  top: 30px;
  font-size: 17px;
  font-style: normal;
  font-weight: 500;
  line-height: normal;
  background: radial-gradient(138.54% 115.36% at 40.42% 17.84%, #1F3DFF 0%, rgba(255, 255, 255, 0.00) 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-wrap: nowrap;
}

.bar_group {
  position: absolute;
  width: 100%;
  top: 72px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bar {
  width: 4px;
  background: linear-gradient(75deg, rgba(73, 122, 237, 0.70) 0%, rgba(96, 176, 255, 0.70) 100%);
  margin: 0 5px;
  transition: height 0.1s;
}

.close {
  position: absolute;
  right: 20px;
  top: 20px;
  width: 20px;
  height: 20px;
}

.scan_ok {
  height: 80px;
}

.bottom_move_2 {
  bottom: 80px;
}

.scan_over {
  height: 0;
}

.bottom_move_3 {
  bottom: 0;
}

.ok {
  width: 200px;
  position: absolute;
  left: calc(50% - 100px);
  top: 30px;
  opacity: 0;
  animation: fadeIn 0.5s 0.5s forwards;
}

.countdown {
  position: absolute;
  right: 15px;
  top: 12px;
  color: #B9B9B9;
  font-size: 14px;
  opacity: 0;
  animation: fadeIn 0.5s 0.5s forwards;
}

.x {
  position: fixed;
  top: 100px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
  font-size: 20px;
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}

@keyframes leftCornerIn {
  0% {
    margin-left: -20px;
    margin-top: 14px;
  }

  100% {
    margin-left: 0px;
    margin-top: 0px;
  }
}

@keyframes rightCornerIn {
  0% {
    margin-right: -20px;
    margin-top: -14px;
  }

  100% {
    margin-left: 0px;
    margin-top: 0px;
  }
}

@keyframes rightIn {
  0% {
    transform: translateX(100%);
  }

  100% {
    transform: translateX(0);
  }
}

@keyframes topIn {
  0% {
    transform: translateY(100%);
  }

  100% {
    transform: translateY(0);
  }
}

@keyframes siriIn {
  0% {
    transform: translate(-50%, -50%) scale(0);
  }

  100% {
    transform: translate(-50%, -50%) scale(1);
  }
}

@keyframes inputIn {
  0% {
    width: 0;
  }

  100% {
    width: calc(100% - 100px);
  }
}

@keyframes itemIn {
  0% {
    opacity: 0;
    transform: translateX(100%);
  }

  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes bottomUp {
  0% {
    opacity: 0;
    transform: translateY(100%);
  }

  100% {
    opacity: 1;
    transform: translateY(0);
  }

}

@media screen and (max-width: 475px) {
  .main {
    --width: 100vw;
  }
}
</style>