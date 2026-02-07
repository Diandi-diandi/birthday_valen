<template>
  <div class="carousel-wrapper">
    <swiper
      :modules="modules"
      :effect="'coverflow'"
      :grab-cursor="true"
      :centered-slides="true"
      :slides-per-view="'auto'"
      :coverflow-effect="{
        rotate: 50,       /* 旋轉角度 */
        stretch: 0,       /* 拉伸距離 */
        depth: 100,       /* 景深 (越大越有立體感) */
        modifier: 1,      /* 特效倍率 */
        slideShadows: true /* 是否開啟陰影 */
      }"
      :pagination="{ clickable: true }"
      :autoplay="{ delay: 2500, disableOnInteraction: false }"
      class="mySwiper"
    >
      <swiper-slide v-for="(img, index) in images" :key="index">
        <img :src="img" />
      </swiper-slide>
    </swiper>
  </div>
</template>

<script>
import { Swiper, SwiperSlide } from 'swiper/vue';

// ✨ 1. 引入 EffectCoverflow 模組
import { Pagination, Autoplay, EffectCoverflow } from 'swiper/modules';

import 'swiper/css';
import 'swiper/css/pagination';
// ✨ 2. 記得引入特效的 CSS
import 'swiper/css/effect-coverflow';

export default {
  components: { Swiper, SwiperSlide },
  setup() {
    const images = [
      'https://picsum.photos/id/101/600/800', // 建議找直式或方形圖效果更好
      'https://picsum.photos/id/102/600/800',
      'https://picsum.photos/id/103/600/800',
      'https://picsum.photos/id/104/600/800',
      'https://picsum.photos/id/105/600/800',
    ];

    return {
      images,
      // ✨ 3. 將 EffectCoverflow 加入模組清單
      modules: [Pagination, Autoplay, EffectCoverflow],
    };
  },
};
</script>

<style scoped>
.carousel-wrapper {
  width: 100%;
  padding-top: 50px;
  padding-bottom: 50px;
  background: #eee; /* 加個背景色讓陰影更明顯 */
}

.swiper {
  width: 100%;
  padding-top: 50px;
  padding-bottom: 50px;
}

.swiper-slide {
  background-position: center;
  background-size: cover;
  width: 300px; /* ✨ 設定固定寬度，讓左右兩邊露出來 */
  height: 400px;
  border-radius: 15px; /* 圓角看起來更像卡片 */
  overflow: hidden;    /* 確保圖片不超出圓角 */
}

.swiper-slide img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
