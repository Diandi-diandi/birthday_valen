<template>
  <div class="birthday-container" @click="createHeart">
    <audio ref="audioPlayer1" loop autoplay>
      <source src="/birthday_song.wav" type="audio/wav" />
    </audio>

    <audio ref="audioPlayer2" loop>
      <source src="/happy_birthday_jiang.mp3" type="audio/mpeg" />
    </audio>

    <div class="floating-bg">
      <div v-for="(icon, index) in bgIcons" :key="index" class="bg-icon" :style="icon.style">
        {{ icon.emoji }}
      </div>
    </div>

    <div class="flags-container">
      <div v-for="n in 12" :key="n" class="flag" :style="{ animationDelay: `${n * 0.1}s` }"></div>
    </div>

    <transition name="fade">
      <div v-if="!isOpened" class="envelope-stage" @click="openEnvelope">
        <div class="intro-glass-box">
          <div class="intro-container">
            <div class="typing-line">
              {{ introLine1 }}
              <span v-if="introLine1.length < text1.length" class="cursor">|</span>
            </div>
            <div class="typing-line">
              {{ introLine2 }}
              <span
                v-if="introLine1.length === text1.length && introLine2.length < text2.length"
                class="cursor"
                >|</span
              >
            </div>
            <transition name="fade">
              <div v-if="showArrow" class="arrow">⬇️</div>
            </transition>
          </div>
        </div>

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
        <swiper effect="cards" :grab-cursor="true" :modules="modules" class="mySwiper">
          <swiper-slide class="glass-card cover-card">
            <h1>Happy Birthday!</h1>
            <p>for 永遠的歌唱小甜心 ❤️</p>
            <!-- <div class="days-counter">
              我們已經認識了 <span class="highlight">{{ daysCount }}</span> 天
            </div> -->
          </swiper-slide>

          <swiper-slide
            v-for="(item, index) in memories"
            :key="index"
            class="glass-card photo-card"
          >
            <div class="polaroid">
              <div
                v-if="item.img2"
                :class="['double-photo-wrapper', { 'vertical-layout': item.isVertical }]"
              >
                <img :src="item.img" class="half-img" @click.stop="openModal(item.img)" />
                <img :src="item.img2" class="half-img" @click.stop="openModal(item.img2)" />
              </div>

              <img v-else :src="item.img" class="single-img" @click.stop="openModal(item.img)" />
              <div class="caption">{{ item.text }}</div>
            </div>
          </swiper-slide>

          <swiper-slide class="glass-card end-teaser">
            <div class="envelope-icon">💌</div>
            <h3>給 Valen 的一封信</h3>
            <p>還有些話，想親口對你說...</p>
            <button class="open-letter-btn" @click="openLetter">開啟信件 ✨</button>
          </swiper-slide>
        </swiper>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="isShowLetter" class="letter-overlay">
        <div class="letter-paper">
          <div class="paper-content">
            <!-- <div class="cake-icon">🎂</div> -->
            <p class="typed-text">{{ typedText }}<span class="cursor">|</span></p>

            <!-- <button class="wish-btn" @click="fireMoreConfetti">再撒一次花 🎉</button>

            <p class="close-hint" style="color: #999; margin-top: 20px; font-size: 0.8rem">
              (重新整理網頁可重看)
            </p> -->
          </div>
        </div>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="showModal" class="image-modal" @click.stop="showModal = false">
        <div class="modal-content">
          <img :src="modalImage" />
          <p class="close-hint">點擊任意處關閉</p>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { Swiper, SwiperSlide } from 'swiper/vue'
import { EffectCards } from 'swiper/modules'
import confetti from 'canvas-confetti'
import 'swiper/css'
import 'swiper/css/effect-cards'

// ================= 資料設定區 (保留你的內容) =================
const startDate = new Date('2020-05-20')
const text1 = '今天是一個很特別的日子...'
const text2 = '點擊開啟這份屬於Valen的專屬祝福'
const fullText = `To Valen:

  不知不覺，已經在螢幕前陪了你 7 年!!
  從校園走入職場的這7年
  不只陪伴著你直播
  也陪伴了你人生的進展
  不論是拿到永久居留證還是遇見了人生的另一半
  對你來說都是非常重要的人生歷程

  也謝謝你，用你的聲音陪伴了我無數個讀書與工作的夜晚。
  未來的日子，祝福你在表演的路上繼續成長❤️

  生日快樂 Happy Birthday~🎂

  by 點滴Diandi`

const memories = [
  {
    img: 'diandi.jpg',
    img2: 'valen.png',
    text: '原本在不同世界的你和我...',
  },
  {
    img: 'photo1.jpg',
    text: '故事的開始...\n2019.12.1我們的第一次相遇~\nSirens十週年【拾年一起】演唱會',
  },
  { img: 'photo2.jpg', text: '2020.11.15 【海嘯唱歌BAR】專場' },
  { img: 'photo3_1.jpg', img2: 'photo3_2.jpg', text: '2021 UP live直播' },
  { img: 'photo_wave.png', text: '2021.07.04 wave 聲播 feat.ctwo' },
  { img: 'photo4.jpg', text: '2021.10.03 疫情後的第一場演出\n也是wave的線下見面會' },
  { img: 'photo6.jpg', text: '2022.02.04 MICO直播\nSUPER家族的實力派女歌手ヾ(´︶`*)ﾉ♬' },
  { img: 'photo7.jpg', text: '2022.06.27 首次在直播上慶生的點滴\nfeat.叭噗' },
  { img: 'photo8.jpg', text: '2022.10.09 17 live直播 第一次派對播\nfeat.衣喬Aki、小安啾' },
  { img: 'photo9.jpg', text: '2022.12.25 線上交換聖誕禮物1' },
  { img: 'photo10.jpg', text: '2022.12.25 線上交換聖誕禮物2' },
  { img: 'photo12.jpg', text: '2023.02.28 庭三歲線下慶生聚會@金色三麥' },
  { img: 'photo13.jpg', text: '2023.06.24 鳳山體育場演出\n難得在高雄的表演' },
  { img: 'photo14.jpg', text: '2023.12.21 線下交換聖誕禮物' },
  { img: 'photo15.jpg', text: '2024.02.18 線下慶生聚會 feat. ctwo' },
  { img: 'photo16.jpg', text: '2024.02.18 線下慶生聚會 feat. Mars、blue' },
  { img: 'photo17.jpg', text: '2024.09.15 微風松高街唱' },
  { img: 'photo18.jpg', text: '2025.08.22 Att Valley 駐唱' },
]
// ===============================================================

// 變數宣告
const introLine1 = ref('')
const introLine2 = ref('')
const showArrow = ref(false)
const isOpened = ref(false)
const isShowLetter = ref(false) // 新增：控制信紙顯示
const isAnimating = ref(false)
const audioPlayer1 = ref(null) // 改名：第一首歌
const audioPlayer2 = ref(null) // 新增：第二首歌
const typedText = ref('')
const isTyping = ref(false)
const showModal = ref(false)
const modalImage = ref('')
const modules = [EffectCards]
const bgIcons = ref([])

// 音樂淡入淡出邏輯
const crossFadeAudio = (fromPlayer, toPlayer) => {
  if (!fromPlayer || !toPlayer) return

  const maxVolume = 0.3 // 目標音量
  toPlayer.volume = 0
  toPlayer.play()

  let vol = 0
  const step = 0.02
  const fadeInterval = setInterval(() => {
    if (vol < maxVolume) {
      vol += step
      toPlayer.volume = Math.min(vol, maxVolume)
      fromPlayer.volume = Math.max(maxVolume - vol, 0)
    } else {
      clearInterval(fadeInterval)
      fromPlayer.pause()
    }
  }, 100)
}

// 開啟信件 (觸發音樂切換 + 打字)
const openLetter = () => {
  isShowLetter.value = true
  // 1. 切換音樂
  crossFadeAudio(audioPlayer1.value, audioPlayer2.value)
  if (audioPlayer2.value) {
    audioPlayer2.value.currentTime = 95
  }
  // 2. 延遲一點點開始打字
  setTimeout(() => {
    startTyping()
    fireMoreConfetti()
  }, 500)
}

// 初始化動態背景
const initBackground = () => {
  const icons = ['🎈', '✨', '🎂', '🎵', '🎁', '🌸', '💖']
  for (let i = 0; i < 20; i++) {
    bgIcons.value.push({
      emoji: icons[Math.floor(Math.random() * icons.length)],
      style: {
        left: `${Math.random() * 100}vw`,
        animationDuration: `${Math.random() * 10 + 10}s`,
        animationDelay: `-${Math.random() * 10}s`,
        fontSize: `${Math.random() * 1.5 + 1}rem`,
        opacity: Math.random() * 0.5 + 0.1,
      },
    })
  }
}

// 開場打字
const runOpeningTyping = async () => {
  for (const char of text1) {
    introLine1.value += char
    await new Promise((r) => setTimeout(r, Math.random() * 100 + 50))
  }
  await new Promise((r) => setTimeout(r, 500))
  for (const char of text2) {
    introLine2.value += char
    await new Promise((r) => setTimeout(r, Math.random() * 100 + 50))
  }
  await new Promise((r) => setTimeout(r, 300))
  showArrow.value = true
}

// 結尾打字
const startTyping = () => {
  if (isTyping.value) return
  isTyping.value = true
  let i = 0
  const speed = Math.random() * 100 + 50 // 稍微調快一點點
  const typeWriter = () => {
    if (i < fullText.length) {
      typedText.value += fullText.charAt(i)
      i++
      setTimeout(typeWriter, speed)
    }
  }
  typeWriter()
}

// 移除原本的 onSlideChange 自動觸發打字，改由按鈕觸發
// const onSlideChange = (swiper) => { ... }

const openEnvelope = () => {
  if (isAnimating.value) return
  isAnimating.value = true
  // 播放第一首歌
  if (audioPlayer1.value && audioPlayer1.value.paused) {
    audioPlayer1.value.volume = 0.2
    audioPlayer1.value.play()
  }
  setTimeout(() => {
    isOpened.value = true
    fireConfetti()
  }, 800)
}

const openModal = (imgSrc) => {
  modalImage.value = imgSrc
  showModal.value = true
}

const fireConfetti = () => {
  const duration = 2000
  const end = Date.now() + duration
  ;(function frame() {
    confetti({
      particleCount: 5,
      angle: 60,
      spread: 55,
      origin: { x: 0 },
      colors: ['#ffafcc', '#bde0fe'],
    })
    confetti({
      particleCount: 5,
      angle: 120,
      spread: 55,
      origin: { x: 1 },
      colors: ['#ffafcc', '#bde0fe'],
    })
    if (Date.now() < end) requestAnimationFrame(frame)
  })()
}

const fireMoreConfetti = () => {
  confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } })
}

const createHeart = (e) => {
  const heart = document.createElement('div')
  heart.innerHTML = '❤️'
  heart.className = 'floating-heart'
  heart.style.left = `${e.clientX}px`
  heart.style.top = `${e.clientY}px`
  const size = Math.random() * 20 + 20
  heart.style.fontSize = `${size}px`
  heart.style.transform = `rotate(${Math.random() * 360}deg)`
  document.body.appendChild(heart)
  setTimeout(() => {
    heart.remove()
  }, 1000)
}

onMounted(() => {
  initBackground()
  runOpeningTyping()

  // 嘗試自動播放第一首歌
  if (audioPlayer1.value) {
    audioPlayer1.value.volume = 0.2
    const playPromise = audioPlayer1.value.play()
    if (playPromise !== undefined) {
      playPromise.catch(() => {
        const playOnInteraction = () => {
          audioPlayer1.value.play()
          document.removeEventListener('click', playOnInteraction)
          document.removeEventListener('touchstart', playOnInteraction)
        }
        document.addEventListener('click', playOnInteraction)
        document.addEventListener('touchstart', playOnInteraction)
      })
    }
  }
})
</script>

<style>
/* 全域樣式：飄浮愛心 */
.floating-heart {
  position: fixed;
  pointer-events: none;
  animation: floatUp 1s ease-out forwards;
  z-index: 9999;
}
@keyframes floatUp {
  0% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
  100% {
    opacity: 0;
    transform: translateY(-100px) scale(1.5);
  }
}
</style>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Pacifico&family=Zen+Maru+Gothic:wght@500&display=swap');

/* --- 📝 新增：信紙視窗樣式 --- */
.letter-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6); /* 深色遮罩 */
  backdrop-filter: blur(5px);
  z-index: 100;
  display: flex;
  justify-content: center;
  align-items: center;
}

.letter-paper {
  width: 90%;
  max-width: 500px;
  min-height: 60vh;

  /* 👇 1. 改成暖米色背景 */
  background-color: #fffdf9;

  /* 👇 2. 加入可愛的暖粉色格子底紋 (取代原本的灰點點) */
  background-image:
    linear-gradient(rgba(255, 175, 204, 0.15) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 175, 204, 0.15) 1px, transparent 1px);
  background-size: 25px 25px; /* 格子稍微大一點 */

  /* 👇 3. 加入可愛的虛線邊框 */
  border: 3px dashed #ffc8dd;

  padding: 40px 30px;
  border-radius: 15px; /* 圓角大一點 */

  /* 加深陰影，更有立體感 */
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);

  position: relative;
  overflow-y: auto;
  max-height: 90vh;
  overflow-x: hidden; /* 防止膠帶超出 */
  animation: slideUp 0.8s ease-out;
}

/* 👇 4. 新增：利用偽元素製作角落的「紙膠帶」裝飾 */
.letter-paper::before,
.letter-paper::after {
  content: '';
  position: absolute;
  width: 50px;
  height: 12px;
  background-color: rgba(255, 158, 187, 0.7); /* 半透明粉色膠帶 */
  box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
}

/* 左上角的膠帶 */
.letter-paper::before {
  top: 15px;
  left: -12px;
  transform: rotate(-45deg);
}

/* 右下角的膠帶 */
.letter-paper::after {
  bottom: 15px;
  right: -12px;
  transform: rotate(-45deg);
}
/* ----------------------------------------- */

@keyframes slideUp {
  from {
    transform: translateY(100px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.paper-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.cake-icon {
  font-size: 4rem;
  margin-bottom: 20px;
  filter: drop-shadow(2px 2px 2px rgba(0, 0, 0, 0.1));
}

/* 修改打字機文字顏色 */
.typed-text {
  font-family: 'Zen Maru Gothic', monospace;
  white-space: pre-line;
  text-align: left;
  line-height: 2;
  font-size: 1.1rem;

  /* 👇 改成暖棕色，比純黑更溫柔 */
  color: #5a4a42;

  width: 100%;
  margin-bottom: 30px;
  /* 加入一點文字陰影增加質感 */
  text-shadow: 1px 1px 0px rgba(255, 255, 255, 0.5);
}
/* --- 🚪 新增：最後一頁入口 (End Teaser) --- */
.end-teaser {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: rgba(255, 255, 255, 0.9);
}
.envelope-icon {
  font-size: 5rem;
  margin-bottom: 20px;
  animation: float 3s ease-in-out infinite;
}
.open-letter-btn {
  margin-top: 30px;
  padding: 12px 30px;
  background: linear-gradient(45deg, #ff9ebb, #ffafcc);
  color: white;
  font-size: 1.2rem;
  border: none;
  border-radius: 50px;
  box-shadow: 0 5px 15px rgba(255, 158, 187, 0.4);
  cursor: pointer;
  transition: transform 0.2s;
}
.open-letter-btn:active {
  transform: scale(0.95);
}

/* --- 💎 全局容器與動態背景 --- */
.birthday-container {
  height: 100vh;
  width: 100vw;
  background: linear-gradient(180deg, #ffe6f0 0%, #ffc2d1 100%);
  font-family: 'Zen Maru Gothic', sans-serif;
  overflow: hidden;
  position: relative;
}

/* 飄浮圖示容器 */
.floating-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

.bg-icon {
  position: absolute;
  bottom: -50px;
  animation: floatUpBg linear infinite;
}

@keyframes floatUpBg {
  0% {
    transform: translateY(0) rotate(0deg);
  }
  100% {
    transform: translateY(-110vh) rotate(360deg);
  }
}

/* --- 🚩 派對掛旗 --- */
.flags-container {
  position: absolute;
  top: -10px;
  left: 0;
  width: 100%;
  height: 60px;
  display: flex;
  justify-content: space-around;
  z-index: 10;
  pointer-events: none;
}

.flag {
  width: 30px;
  height: 40px;
  background: #ff9ebb;
  clip-path: polygon(0 0, 100% 0, 50% 100%);
  transform-origin: top;
  animation: swing 2s ease-in-out infinite alternate;
}
.flag:nth-child(even) {
  background: #bde0fe;
}
.flag:nth-child(3n) {
  background: #ffdfba;
}

@keyframes swing {
  from {
    transform: rotate(-10deg);
  }
  to {
    transform: rotate(10deg);
  }
}

/* --- 💌 開場區塊 --- */
.envelope-stage {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(255, 227, 236, 0.6);
  backdrop-filter: blur(5px);
  z-index: 50;
  cursor: pointer;
}

/* 文字玻璃框 */
.intro-glass-box {
  background: rgba(255, 255, 255, 0.4);
  padding: 20px 40px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  margin-bottom: 30px;
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.1);
}

.intro-container {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.typing-line {
  font-size: 1.2rem;
  color: #c95c7e;
  font-weight: bold;
  min-height: 1.5rem;
  text-shadow: 0 1px 2px rgba(255, 255, 255, 0.8);
}

.cursor {
  display: inline-block;
  margin-left: 2px;
  animation: blink 1s infinite;
  color: #c95c7e;
}
.arrow {
  font-size: 2rem;
  margin-top: 10px;
  animation: bounce 1.5s infinite;
}
@keyframes bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(10px);
  }
}
@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

/* 信封樣式 */
.envelope-wrapper {
  position: relative;
  width: 300px;
  height: 200px;
  transition: transform 0.5s;
}
.envelope {
  position: relative;
  width: 100%;
  height: 100%;
}
.envelope .front {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 0;
  border-left: 150px solid transparent;
  border-right: 150px solid transparent;
  border-bottom: 120px solid #ffafcc;
  z-index: 3;
}
.envelope .top {
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 0;
  border-left: 150px solid transparent;
  border-right: 150px solid transparent;
  border-top: 110px solid #ff9ebb;
  transform-origin: top;
  transition:
    transform 0.6s ease-in-out,
    z-index 0.6s;
  z-index: 4;
}
.card-insert {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 260px;
  height: 180px;
  background: white;
  z-index: 2;
  transition: transform 0.6s ease-in-out;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
}
.heart {
  font-size: 3rem;
}
.shadow {
  position: absolute;
  bottom: -20px;
  left: 50%;
  transform: translateX(-50%);
  width: 280px;
  height: 20px;
  background: rgba(0, 0, 0, 0.1);
  border-radius: 50%;
  z-index: 1;
}
.open-anim .envelope .top {
  transform: rotateX(180deg);
  z-index: 1;
}
.open-anim .card-insert {
  transform: translateX(-50%) translateY(-80px);
  transition-delay: 0.2s;
}
.open-anim {
  transform: scale(1.1);
}

/* --- 卡片輪播 --- */
.carousel-stage {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 20;
}
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

.glass-card {
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.cover-card {
  background: linear-gradient(135deg, rgba(255, 175, 204, 0.9), rgba(189, 224, 254, 0.9));
  color: white;
}
.cover-card h1 {
  font-family: 'Pacifico', cursive;
  font-size: 3rem;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.photo-card {
  padding: 20px;
}
.polaroid {
  width: 90%;
  height: 85%;
  background: white;
  padding: 10px 10px 40px 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transform: rotate(-2deg);
  transition: transform 0.3s;
}
.caption {
  margin-top: 10px;
  font-size: 0.9rem;
  color: #555;
  white-space: pre-line;
  line-height: 1.6;
}
.polaroid:active {
  transform: rotate(0deg) scale(1.02);
}

.single-img {
  width: 100%;
  height: 80%;
  object-fit: contain;
  background-color: #f8f8f8;
  border: 1px solid #eee;
}

/* 雙圖樣式 */
.double-photo-wrapper {
  width: 100%;
  height: 80%;
  display: flex;
  gap: 5px;
}
.half-img {
  width: 50%;
  height: 100%;
  object-fit: cover;
  border: 1px solid #eee;
  border-radius: 4px;
}
.vertical-layout {
  flex-direction: column;
}
.vertical-layout .half-img {
  width: 100%;
  height: 50%;
}

.wish-btn {
  margin-top: 20px;
  padding: 10px 25px;
  background: linear-gradient(45deg, #ffafcc, #ff9ebb);
  border: none;
  border-radius: 30px;
  color: white;
  cursor: pointer;
  font-size: 1.1rem;
  box-shadow: 0 4px 15px rgba(255, 158, 187, 0.4);
  transition: transform 0.2s;
}
.wish-btn:active {
  transform: scale(0.95);
}

/* 計數器 */
.days-counter {
  margin: 20px 0;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.25);
  padding: 10px 20px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.4);
}
.highlight {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* 圖片放大視窗 */
.image-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.9);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: zoom-out;
}
.modal-content img {
  max-width: 95%;
  max-height: 80vh;
  object-fit: contain;
  border: 5px solid white;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
}
.close-hint {
  color: white;
  margin-top: 15px;
  text-align: center;
  font-size: 0.9rem;
  opacity: 0.7;
}
@keyframes float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.zoom-enter-active {
  transition: all 0.8s ease-out;
}
.zoom-enter-from {
  opacity: 0;
  transform: scale(0.8);
}
</style>
