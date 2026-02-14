<template>
  <div class="birthday-container" @click="createHeart">
    <audio ref="audioPlayer" loop autoplay>
      <source src="/birthday_song.wav" type="audio/mpeg" />
    </audio>

    <transition name="fade">
      <div v-if="!isOpened" class="envelope-stage" @click="openEnvelope">
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
          @slideChange="onSlideChange"
          effect="cards"
          :grab-cursor="true"
          :modules="modules"
          class="mySwiper"
        >
          <swiper-slide class="cover-card">
            <h1>
              Happy <br />
              Birthday!
            </h1>
            <p>致 最特別的你 ✨</p>

            <div class="days-counter">
              我們已經一起度過了 <span class="highlight">{{ daysCount }}</span> 天
            </div>

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
            <div class="typing-container">
              <p class="typed-text">{{ typedText }}<span class="cursor">|</span></p>
            </div>
            <button class="wish-btn" @click="fireMoreConfetti">再撒一次花 🎉</button>
          </swiper-slide>
        </swiper>
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

// --- 資料設定區 (請修改這裡) ---
const startDate = new Date('2020-05-20') // 在一起的日子或生日
const text1 = '今天是一個很特別的日子...'
const text2 = '點擊開啟這份屬於Valen的專屬祝福'
const fullText =
  '親愛的，生日快樂。\n這一年有你真好，\n未來的路，我們也要一起慢慢走。\n願你眼裡有光，心中有愛。❤️'

// 照片資料
const memories = [
  { img: 'https://picsum.photos/id/65/600/800', text: '第一次去海邊' },
  { img: 'https://picsum.photos/id/102/600/800', text: '那是超級開心的一天' },
  { img: 'https://picsum.photos/id/177/600/800', text: '不管過了多久...' },
  { img: 'https://picsum.photos/id/237/600/800', text: '這裡永遠有個位置留給你 🐶' },
]

// --- 變數宣告 ---
const introLine1 = ref('')
const introLine2 = ref('')
const showArrow = ref(false)
const isOpened = ref(false) // 是否進入輪播
const isAnimating = ref(false) // 信封動畫狀態
const audioPlayer = ref(null)
const typedText = ref('')
const isTyping = ref(false)
const modules = [EffectCards]

// --- 計算屬性 ---
const daysCount = computed(() => {
  const now = new Date()
  const diff = now - startDate
  return Math.floor(diff / (1000 * 60 * 60 * 24))
})

// --- 邏輯函式 ---

// 1. 開場打字特效
const runOpeningTyping = async () => {
  for (const char of text1) {
    introLine1.value += char
    await new Promise((r) => setTimeout(r, 150))
  }
  await new Promise((r) => setTimeout(r, 500))

  for (const char of text2) {
    introLine2.value += char
    await new Promise((r) => setTimeout(r, 150))
  }
  await new Promise((r) => setTimeout(r, 300))
  showArrow.value = true
}

// 2. 結尾卡片打字特效
const startTyping = () => {
  if (isTyping.value) return
  isTyping.value = true
  let i = 0
  const speed = 150
  const typeWriter = () => {
    if (i < fullText.length) {
      typedText.value += fullText.charAt(i)
      i++
      setTimeout(typeWriter, speed)
    }
  }
  typeWriter()
}

// 3. 監聽 Swiper 切換 (滑到最後一張時觸發打字)
const onSlideChange = (swiper) => {
  if (swiper.activeIndex === memories.length + 1) {
    startTyping()
  }
}

// 4. 開啟信封
const openEnvelope = () => {
  if (isAnimating.value) return
  isAnimating.value = true

  // 播放音樂 (雙重保險)
  if (audioPlayer.value && audioPlayer.value.paused) {
    audioPlayer.value.play()
  }

  // 延遲後切換畫面並撒花
  setTimeout(() => {
    isOpened.value = true
    fireConfetti()
  }, 800)
}

// 5. 撒花特效
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

// 6. 點擊冒愛心
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

// --- 生命週期 ---
onMounted(() => {
  runOpeningTyping()

  // 嘗試自動播放音樂
  if (audioPlayer.value) {
    audioPlayer.value.volume = 0.2
    const playPromise = audioPlayer.value.play()
    if (playPromise !== undefined) {
      playPromise.catch(() => {
        console.log('自動播放被擋，改為點擊播放')
        const playOnInteraction = () => {
          audioPlayer.value.play()
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

/* --- 全局容器 --- */
.birthday-container {
  height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #ffe3ec 0%, #ffc0cb 100%);
  font-family: 'Zen Maru Gothic', sans-serif;
  overflow: hidden;
  position: relative;
}

/* --- 開場打字區 --- */
.intro-container {
  text-align: center;
  margin-bottom: 40px;
  min-height: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.typing-line {
  font-size: 1.2rem;
  color: #d68c9f;
  font-weight: bold;
  font-family: 'Zen Maru Gothic', sans-serif;
  min-height: 1.5rem;
}

/* 共用的游標動畫 */
.cursor {
  display: inline-block;
  margin-left: 2px;
  animation: blink 1s infinite;
  color: #d68c9f;
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

/* --- 箭頭動畫 --- */
.arrow {
  font-size: 2rem;
  margin-bottom: 30px;
  animation: bounce 1.5s infinite;
  cursor: pointer;
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

/* --- 信封樣式 --- */
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
  background: #ffe3ec;
  z-index: 50;
  cursor: pointer;
}

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

/* 信封開啟動畫 */
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

/* --- 輪播樣式 --- */
.carousel-stage {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
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

.swiper-slide {
  border-radius: 20px;
  background: white;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

/* 封面與拍立得 */
.cover-card {
  background: linear-gradient(to bottom right, #ffafcc, #bde0fe);
  color: white;
}
.cover-card h1 {
  font-family: 'Pacifico', cursive;
  font-size: 3rem;
  margin: 0;
}
.photo-card {
  padding: 20px;
  background: #fff;
}
.polaroid {
  width: 90%;
  height: 85%;
  background: white;
  padding: 10px 10px 40px 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transform: rotate(-2deg);
}
.polaroid img {
  width: 100%;
  height: 80%;
  object-fit: cover;
  border: 1px solid #eee;
}

/* 結尾卡片 */
.end-card {
  background: #fff0f5;
}
.cake {
  font-size: 5rem;
}
.typed-text {
  font-family: 'Zen Maru Gothic', monospace;
  white-space: pre-line;
  text-align: left;
  line-height: 1.8;
  font-size: 1.1rem;
  color: #555;
  min-height: 150px;
}
.wish-btn {
  margin-top: 20px;
  padding: 10px 20px;
  background: #ffafcc;
  border: none;
  border-radius: 30px;
  color: white;
  cursor: pointer;
}

/* --- 天數計數器 --- */
.days-counter {
  margin: 20px 0;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.2);
  padding: 10px 20px;
  border-radius: 20px;
}
.highlight {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* --- Vue Transitions --- */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
} /* 重複定義合併 */

.zoom-enter-active {
  transition: all 0.8s ease-out;
}
.zoom-enter-from {
  opacity: 0;
  transform: scale(0.8);
}
</style>
