<template>
  <div id="app">
    <main>
      <BirthdayCard />
    </main>
  </div>
</template>

<script setup>
// 引入你的輪播組件
import BirthdayCard from './components/BirthdayCard.vue'
</script>

<style>
/* 1. 清除所有元素的預設邊距 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 2. 強制 html, body 和 #app 佔滿整個螢幕 */
html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden; /* 禁止出現捲軸 */
  background-color: #d68c9f; /* 溫暖的背景色 */
}

/* 3. 確保 main 標籤也是滿版 */
main {
  width: 100%;
  height: 100%;
  display: block;
}
</style>
