<template>
  <div class="carousel-container">
    <swiper
      :modules="modules"
      :slides-per-view="1"
      :space-between="10"
      :pagination="{ clickable: true }"
      :navigation="true"
      :loop="true"
      class="mySwiper"
    >
      <swiper-slide v-for="(img, index) in images" :key="index">
        <img :src="img" :alt="'Slide ' + (index + 1)" />
      </swiper-slide>
    </swiper>
  </div>
</template>

<script>
// 引入 Swiper Vue 組件
import { Swiper, SwiperSlide } from 'swiper/vue';
// 引入 Swiper 核心樣式與模組樣式
import 'swiper/css';
import 'swiper/css/pagination';
import 'swiper/css/navigation';

// 引入需要的功能模組
import { Pagination, Navigation } from 'swiper/modules';

export default {
  components: {
    Swiper,
    SwiperSlide,
  },
  setup() {
    // 這裡放置圖片網址，建議使用適合平板比例的圖片
    const images = [
      'https://picsum.photos/id/10/1200/800',
      'https://picsum.photos/id/20/1200/800',
      'https://picsum.photos/id/30/1200/800',
    ];

    return {
      images,
      modules: [Pagination, Navigation],
    };
  },
};
</script>

<style scoped>
.carousel-container {
  width: 100%;
  max-width: 1024px; /* 適合平板的寬度 */
  margin: 0 auto;
  height: 500px;
}

.swiper {
  width: 100%;
  height: 100%;
  border-radius: 12px;
}

.swiper-slide {
  display: flex;
  justify-content: center;
  align-items: center;
}

.swiper-slide img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* 確保圖片填滿不變形 */
}
</style>
