<template>
  <div id="app">
    <header>
      <h1>我的平板輪播展示</h1>
    </header>

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
/* 全域樣式設定，確保平板滿版顯示 */
body {
  margin: 0;
  padding: 0;
  background-color: #f5f5f5;
  font-family: Arial, sans-serif;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

header {
  margin-bottom: 20px;
}
</style>
