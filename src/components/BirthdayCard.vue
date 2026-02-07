<template>
  <div class="birthday-container">

    <div v-if="!isStarted" class="start-overlay" @click="startCelebration">
      <div class="envelope">
        💌 點擊開啟這份專屬祝福
      </div>
    </div>

    <audio ref="audioPlayer" loop>
      <source src="/birthday-song.wav" type="audio/mpeg">
    </audio>

    <swiper
      effect="cards"
      :grab-cursor="true"
      :modules="modules"
      class="mySwiper"
      @slideChange="onSlideChange"
    >
      <swiper-slide class="cover-card">
        <h1>Happy Birthday!</h1>
        <p>致 最特別的你 ✨</p>
        <div class="hint">👉 向左滑動拆開禮物</div>
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
        <p>願你所有夢想都成真</p>
        <button class="wish-btn" @click="fireMoreConfetti">再撒一次花 🎉</button>
      </swiper-slide>
    </swiper>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { Swiper, SwiperSlide } from 'swiper/vue';
import { EffectCards } from 'swiper/modules';
import confetti from 'canvas-confetti'; // 引入撒花套件

// 引入樣式
import 'swiper/css';
import 'swiper/css/effect-cards';

const modules = [EffectCards];
const isStarted = ref(false);
const audioPlayer = ref(null);

// 定義回憶內容 (你可以隨意修改這裡的文字)
const memories = [
  { img: 'https://picsum.photos/id/65/600/800', text: '還記得我們第一次去海邊嗎？' },
  { img: 'https://picsum.photos/id/102/600/800', text: '那是超級開心的一天' },
  { img: 'https://picsum.photos/id/177/600/800', text: '不管過了多久...' },
  { img: 'https://picsum.photos/id/237/600/800', text: '這裡永遠有個位置留給你 🐶' },
];

// 開始慶祝 (播放音樂 + 撒花)
const startCelebration = () => {
  isStarted.value = true;
  // 嘗試播放音樂
  if (audioPlayer.value) {
    audioPlayer.value.volume = 0.5; // 音量 50%
    audioPlayer.value.play().catch(e => console.log("瀏覽器阻擋自動播放", e));
  }
  // 發射第一波碎紙花
  fireConfetti();
};

// 撒花特效函式
const fireConfetti = () => {
  const duration = 3000;
  const end = Date.now() + duration;

  (function frame() {
    confetti({
      particleCount: 5,
      angle: 60,
      spread: 55,
      origin: { x: 0 },
      colors: ['#ffafcc', '#bde0fe', '#a2d2ff'] // 馬卡龍色系
    });
    confetti({
      particleCount: 5,
      angle: 120,
      spread: 55,
      origin: { x: 1 },
      colors: ['#ffafcc', '#bde0fe', '#a2d2ff']
    });

    if (Date.now() < end) {
      requestAnimationFrame(frame);
    }
  }());
};

// 手動再撒一次
const fireMoreConfetti = () => {
  confetti({
    particleCount: 100,
    spread: 70,
    origin: { y: 0.6 }
  });
};

// 滑動時的小驚喜 (可選)
const onSlideChange = () => {
  // 可以在這裡加入切換時的音效
};
</script>

<style scoped>
/* 匯入好看的手寫字體 */
@import url('https://fonts.googleapis.com/css2?family=Pacifico&family=Zen+Maru+Gothic:wght@500&display=swap');

.birthday-container {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #fdfbfb 0%, #ebedee 100%);
  font-family: 'Zen Maru Gothic', sans-serif;
  overflow: hidden;
}

/* 開場遮罩 */
.start-overlay {
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.8);
  z-index: 999;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.envelope {
  font-size: 2rem;
  color: white;
  animation: bounce 1s infinite;
}

/* Swiper 卡片樣式 */
.swiper {
  width: 300px;
  height: 480px;
}

@media (min-width: 768px) {
  .swiper {
    width: 450px;
    height: 600px;
  }
}

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

/* 封面卡片 */
.cover-card {
  background: linear-gradient(to bottom right, #ffafcc, #bde0fe);
  color: white;
}
.cover-card h1 {
  font-family: 'Pacifico', cursive;
  font-size: 3rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}
.hint {
  margin-top: 50px;
  font-size: 0.9rem;
  opacity: 0.8;
  animation: pulse 1.5s infinite;
}

/* 拍立得照片樣式 */
.photo-card {
  padding: 20px;
  background: #fff;
}
.polaroid {
  width: 90%;
  height: 85%;
  background: white;
  padding: 10px 10px 40px 10px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  transform: rotate(-2deg); /* 微微傾斜更有感覺 */
}
.polaroid img {
  width: 100%;
  height: 80%;
  object-fit: cover;
  border: 1px solid #eee;
}
.caption {
  margin-top: 15px;
  font-family: 'Zen Maru Gothic', sans-serif;
  color: #555;
  font-size: 1.2rem;
}

/* 結尾卡片 */
.end-card {
  background: #fff0f5;
}
.cake {
  font-size: 5rem;
  margin-bottom: 20px;
}
.wish-btn {
  margin-top: 20px;
  padding: 10px 20px;
  background: #ffafcc;
  border: none;
  border-radius: 30px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 4px 10px rgba(255, 175, 204, 0.5);
}

/* 動畫 */
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}
@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}
</style>
