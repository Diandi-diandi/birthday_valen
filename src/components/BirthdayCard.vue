<template>
  <div class="birthday-container">

    <audio ref="audioPlayer" loop>
      <source src="/birthday-song.wav" type="audio/mpeg">
    </audio>

    <transition name="fade">
      <div v-if="!isOpened" class="envelope-stage" @click="openEnvelope">
        <div class="instruction">✨ 點擊開啟這份屬於Valen的專屬祝福 ✨</div>
        <div class="envelope-wrapper" :class="{ 'open-anim': isAnimating }">
          <div class="envelope">
            <div class="front"></div>
            <div class="card-insert">
              <div class="heart">❤️</div>
            </div>
            <div class="top"></div>
            <div class="shadow"></div>
          </div>
        </div>
      </div>
    </transition>

    <transition name="zoom">
      <div v-if="isOpened" class="carousel-stage">
        <swiper
          effect="cards"
          :grab-cursor="true"
          :modules="modules"
          class="mySwiper"
        >
          <swiper-slide class="cover-card">
            <h1>Happy Birthday!</h1>
            <p>致 最特別的你 ✨</p>
            <div class="hint">👉 向左滑動看照片</div>
          </swiper-slide>

          <swiper-slide v-for="(item, index) in memories" :key="index" class="photo-card">
            <div class="polaroid">
              <img :src="item.img" />
              <div class="caption">{{ item.text }}</div>
            </div>
          </swiper-slide>

          <swiper-slide class="end-card">
            <div class="cake">🎂</div>
            <h2>許個願吧！</h2>
            <button class="wish-btn" @click="fireMoreConfetti">再撒一次花 🎉</button>
          </swiper-slide>
        </swiper>
      </div>
    </transition>

  </div>
</template>

<script setup>
import { ref } from 'vue';
import { Swiper, SwiperSlide } from 'swiper/vue';
import { EffectCards } from 'swiper/modules';
import confetti from 'canvas-confetti';

import 'swiper/css';
import 'swiper/css/effect-cards';

// 狀態控制
const isOpened = ref(false);     // 是否已經進入輪播畫面
const isAnimating = ref(false);  // 是否正在播放開信動畫
const audioPlayer = ref(null);
const modules = [EffectCards];

// 你的回憶資料
const memories = [
  { img: 'https://picsum.photos/id/65/600/800', text: '第一次去海邊' },
  { img: 'https://picsum.photos/id/102/600/800', text: '那是超級開心的一天' },
  { img: 'https://picsum.photos/id/177/600/800', text: '不管過了多久...' },
  { img: 'https://picsum.photos/id/237/600/800', text: '這裡永遠有個位置留給你 🐶' },
];

// 開啟信封的邏輯
const openEnvelope = () => {
  if (isAnimating.value) return; // 防止重複點擊

  // 1. 開始播放 CSS 動畫
  isAnimating.value = true;

  // 2. 播放音樂 (瀏覽器允許在使用者點擊後播放)
  if (audioPlayer.value) {
    audioPlayer.value.volume = 0.5;
    audioPlayer.value.play().catch(e => console.log("音樂播放失敗", e));
  }

  // 3. 延遲 0.8 秒後 (等信封打開)，切換到輪播畫面並撒花
  setTimeout(() => {
    isOpened.value = true;
    fireConfetti();
  }, 800);
};

// 撒花特效
const fireConfetti = () => {
  const duration = 2000;
  const end = Date.now() + duration;

  (function frame() {
    confetti({ particleCount: 5, angle: 60, spread: 55, origin: { x: 0 }, colors: ['#ffafcc', '#bde0fe'] });
    confetti({ particleCount: 5, angle: 120, spread: 55, origin: { x: 1 }, colors: ['#ffafcc', '#bde0fe'] });
    if (Date.now() < end) requestAnimationFrame(frame);
  }());
};

const fireMoreConfetti = () => {
  confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Pacifico&family=Zen+Maru+Gothic:wght@500&display=swap');

/* --- 全局容器 --- */
.birthday-container {
  height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #ffe3ec 0%, #ffc0cb 100%);
  font-family: 'Zen Maru Gothic', sans-serif;
  overflow: hidden;
  position: relative;
}

/* --- 1. 信封樣式 (純 CSS 畫圖) --- */
.envelope-stage {
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #ffe3ec; /* 溫暖的背景色 */
  z-index: 50;
  cursor: pointer;
}

.instruction {
  margin-bottom: 40px;
  color: #d68c9f;
  font-size : 1.5rem;
  font-weight: bold;
  font-family: 'Pacifico', cursive;
  animation: float 2s ease-in-out infinite;
}

.envelope-wrapper {
  position: relative;
  width: 300px;
  height: 200px;
  transition: transform 0.5s;
}

.envelope {
  position: relative;
  width: 100%; height: 100%;
}

/* 信封身體 */
.envelope .front {
  position: absolute;
  bottom: 0; left: 0;
  width: 0; height: 0;
  border-left: 150px solid transparent;
  border-right: 150px solid transparent;
  border-bottom: 120px solid #ffafcc; /* 信封顏色 */
  z-index: 3;
}

/* 信封上蓋 (會動的部分) */
.envelope .top {
  position: absolute;
  top: 0; left: 0;
  width: 0; height: 0;
  border-left: 150px solid transparent;
  border-right: 150px solid transparent;
  border-top: 110px solid #ff9ebb; /* 上蓋顏色略深 */
  transform-origin: top;
  transition: transform 0.6s ease-in-out, z-index 0.6s; /* 加慢動畫 */
  z-index: 4;
}

/* 信紙 / 愛心 */
.card-insert {
  position: absolute;
  bottom: 0; left: 50%;
  transform: translateX(-50%);
  width: 260px; height: 180px;
  background: white;
  z-index: 2;
  transition: transform 0.6s ease-in-out;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
}
.heart { font-size: 3rem; }

/* 陰影 */
.shadow {
  position: absolute;
  bottom: -20px; left: 50%;
  transform: translateX(-50%);
  width: 280px; height: 20px;
  background: rgba(0,0,0,0.1);
  border-radius: 50%;
  z-index: 1;
}

/* --- 動畫狀態 --- */
.open-anim .envelope .top {
  transform: rotateX(180deg); /* 打開蓋子 */
  z-index: 1; /* 打開後蓋子要跑到信紙後面 */
}

.open-anim .card-insert {
  transform: translateX(-50%) translateY(-80px); /* 信紙向上浮出 */
  transition-delay: 0.2s;
}

.open-anim {
  transform: scale(1.1); /* 點擊時整個放大一點 */
}

/* --- 2. 輪播樣式 (沿用之前的) --- */
.carousel-stage {
  width: 100%; height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.swiper { width: 300px; height: 480px; }
@media (min-width: 768px) { .swiper { width: 450px; height: 600px; } }

.swiper-slide {
  border-radius: 20px;
  background: white;
  box-shadow: 0 10px 30px rgba(0,0,0,0.15);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

/* 封面與拍立得樣式 */
.cover-card { background: linear-gradient(to bottom right, #ffafcc, #bde0fe); color: white; }
.cover-card h1 { font-family: 'Pacifico', cursive; font-size: 3rem; margin: 0; }
.photo-card { padding: 20px; background: #fff; }
.polaroid { width: 90%; height: 85%; background: white; padding: 10px 10px 40px 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); transform: rotate(-2deg); }
.polaroid img { width: 100%; height: 80%; object-fit: cover; border: 1px solid #eee; }
.end-card { background: #fff0f5; }
.cake { font-size: 5rem; }
.wish-btn { margin-top: 20px; padding: 10px 20px; background: #ffafcc; border: none; border-radius: 30px; color: white; cursor: pointer; }

/* Vue Transition 動畫 (淡入淡出) */
.fade-enter-active, .fade-leave-active { transition: opacity 0.8s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.zoom-enter-active { transition: all 0.8s ease-out; }
.zoom-enter-from { opacity: 0; transform: scale(0.8); }

/* 浮動動畫 */
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}
</style>
