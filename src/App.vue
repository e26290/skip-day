<template>
  <div id="app">
    <!-- US1: 首頁 -->
    <HomePage
      v-if="currentPage === 'home'"
      @start="handleStart"
    />

    <!-- US2: 抽牌 + 結果 -->
    <DrawCards
      v-else-if="currentPage === 'draw'"
      :cards="allCards"
      @selected="handleCardSelected"
    />

    <CardResult
      v-else-if="currentPage === 'result'"
      :card="selectedCard"
      @generate="currentPage = 'excuse'"
      @again="currentPage = 'draw'"
    />

    <!-- US3 + US4: 理由 + 複製 -->
    <ExcuseScreen
      v-else-if="currentPage === 'excuse'"
      :card="selectedCard"
      :tone="selectedTone"
      @tone-change="selectedTone = $event"
      @back="resetGame"
      @again="currentPage = 'draw'"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import HomePage from './components/HomePage.vue'
import DrawCards from './components/DrawCards.vue'
import CardResult from './components/CardResult.vue'
import ExcuseScreen from './components/ExcuseScreen.vue'
import { cards as allCards } from './data/cards'

// 狀態管理
const currentPage = ref('home')           // 'home' | 'draw' | 'result' | 'excuse'
const selectedCard = ref(null)            // 選中的卡牌物件
const selectedTone = ref('平靜')          // 語氣：'平靜' | '中二' | '社恐'

// 事件處理
const handleStart = () => {
  currentPage.value = 'draw'
}

const handleCardSelected = (card) => {
  selectedCard.value = card
  // 等動畫完成後顯示結果頁
  setTimeout(() => {
    currentPage.value = 'result'
  }, 600)
}

const resetGame = () => {
  currentPage.value = 'home'
  selectedCard.value = null
  selectedTone.value = '平靜'
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #6b4ce6, #a78bfa);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

#app {
  background: white;
  border-radius: 20px;
  padding: 40px;
  max-width: 600px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

@media (max-width: 480px) {
  #app {
    padding: 20px;
  }

  h1 {
    font-size: 36px;
  }

  h2 {
    font-size: 24px;
  }
}
</style>
