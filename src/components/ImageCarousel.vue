<template>
  <div class="carousel-container">
    <swiper
      effect="cards"
      :grab-cursor="true"
      :modules="modules"
      class="mySwiper"
    >
      <swiper-slide v-for="(img, index) in images" :key="index">
        <img :src="img" :alt="'Slide ' + (index + 1)" />
        <div class="slide-number">{{ index + 1 }}</div>
      </swiper-slide>
    </swiper>
  </div>
</template>

<script setup>
// 1. 引入核心組件
import { Swiper, SwiperSlide } from 'swiper/vue';

// 2. 引入 Cards 模組 (這是關鍵！)
import { EffectCards } from 'swiper/modules';

// 3. 引入必要的 CSS
import 'swiper/css';
import 'swiper/css/effect-cards'; // 👈 這行一定要有，不然卡片會亂飛

// 圖片資料
const images = [
  'https://picsum.photos/id/10/600/800',
  'https://picsum.photos/id/20/600/800',
  'https://picsum.photos/id/30/600/800',
  'https://picsum.photos/id/40/600/800',
  'https://picsum.photos/id/50/600/800',
];

// 設定使用的模組
const modules = [EffectCards];
</script>

<style scoped>
.carousel-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh; /* 讓輪播垂直置中 */
  background-color: #f3f3f3; /* 淺灰背景讓白色卡片更明顯 */
  overflow: hidden;
}

/* ⚠️ 關鍵設定：Cards 特效的 Swiper 必須有固定寬度 */
.swiper {
  width: 280px;  /* 手機版寬度 */
  height: 420px;
}

/* 針對平板 (iPad) 調整尺寸，讓它更大張 */
@media (min-width: 768px) {
  .swiper {
    width: 500px;
    height: 700px;
  }
}

.swiper-slide {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 18px; /* 圓角 */
  font-size: 22px;
  font-weight: bold;
  color: #fff;
  background-color: #fff;
  box-shadow: 0 8px 30px rgba(0,0,0,0.12); /* 加點陰影增加立體感 */
}

.swiper-slide img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 18px;
}

.slide-number {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 5px 12px;
  border-radius: 20px;
  font-size: 14px;
}
</style>
