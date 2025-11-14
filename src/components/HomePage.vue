<template>
  <div class="home-page">
    <h1>Skip Day 🔮</h1>
    <p class="subtitle">Tap the cards. Escape reality.</p>
    <button class="start-btn" @click="$emit('start')">Start My Fate 🔮</button>
    <p class="disclaimer">此工具僅供娛樂參考 😄</p>
  </div>
</template>

<script setup>
defineEmits(['start'])
</script>

<style scoped>
.home-page {
  text-align: center;
  padding: 40px 20px;
}

h1 {
  font-size: 48px;
  color: #6b4ce6;
  margin-bottom: 10px;
  font-weight: 800;
  letter-spacing: 2px;
}

.subtitle {
  font-size: 18px;
  color: #999;
  margin-bottom: 40px;
  font-style: italic;
  letter-spacing: 1px;
}

.start-btn {
  padding: 16px 32px;
  font-size: 18px;
  background: linear-gradient(135deg, #6b4ce6, #f0a500);
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 20px rgba(107, 76, 230, 0.3);
  font-weight: bold;
  letter-spacing: 1px;
}

.start-btn:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 30px rgba(107, 76, 230, 0.4);
}

.start-btn:active {
  transform: translateY(-2px);
}

.disclaimer {
  font-size: 12px;
  color: #ccc;
  margin-top: 30px;
}
</style>
