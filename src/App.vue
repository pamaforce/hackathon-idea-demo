<template>
  <main class="main">
    <img src="./assets/qq_top.png" class="top" draggable="false">
    <img src="./assets/qq_message.png" class="message" v-if="showMessage">
    <img src="./assets/qq_pat.png" class="pat" v-if="showPat" />
    <img src="./assets/qq_bottom.png" class="bottom">
    <img src="./assets/sdk_tip.png" class="tip" v-if="showTip">
    <canvas v-if="showCanvas" ref="canvas" class="canvas"></canvas>
  </main>
</template>

<script setup>
import { nextTick, onMounted, ref, watch } from 'vue';

let showMessage = ref(false);
let showPat = ref(false);
let showTip = ref(false);
let showCanvas = ref(false);
let step = ref(0);

const handleClick = () => {
  step.value++;
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
};

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

onMounted(() => {
  window.addEventListener('click', handleClick);
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
  width: 100%;
  position: absolute;
  bottom: 0;
  left: 0;
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

@keyframes fadeIn {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
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

@media screen and (max-width: 475px) {
  .main {
    --width: 100vw;
  }
}
</style>