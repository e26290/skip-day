<template>
  <div class="draw-cards">
    <div class="cards-grid">
      <div
        v-for="card in visibleCards"
        :key="card.cardId"
        class="card"
        :class="{ flipped: flippedCardId === card.cardId }"
        @click="handleCardClick(card)"
      >
        <!-- 卡牌背面 -->
        <div class="card-back">🔮</div>
        <!-- 卡牌正面 (顯示 emoji) -->
        <div class="card-front">{{ card.emoji }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  cards: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits(['selected'])

const visibleCards = ref([])
const flippedCardId = ref(null)
const isAnimating = ref(false)

// 隨機打亂陣列
const shuffle = (array) => {
  const arr = [...array]
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

onMounted(() => {
  // 隨機選 3-5 張卡牌
  const shuffled = shuffle(props.cards)
  const count = Math.floor(Math.random() * 3) + 3 // 3-5
  visibleCards.value = shuffled.slice(0, count)
})

const handleCardClick = (card) => {
  if (isAnimating.value) return

  isAnimating.value = true
  flippedCardId.value = card.cardId

  // 等 600ms 動畫結束後發送事件
  setTimeout(() => {
    emit('selected', card)
  }, 600)
}
</script>

<style scoped>
.draw-cards {
  padding: 30px 0;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}

.card {
  aspect-ratio: 2/3;
  position: relative;
  cursor: pointer;
  perspective: 1000px;
  height: 150px;
}

.card > div {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  font-size: 40px;
  font-weight: bold;
  transition: opacity 0.6s ease, transform 0.6s ease;
  backface-visibility: hidden;
}

.card-back {
  background: linear-gradient(135deg, #6b4ce6, #a78bfa);
  color: white;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.card-front {
  background: #fafafa;
  color: #222;
  opacity: 0;
  transform: rotateY(180deg);
  font-size: 50px;
  padding: 10px;
  text-align: center;
  line-height: 1.2;
}

.card.flipped .card-back {
  opacity: 0;
  transform: rotateY(180deg);
}

.card.flipped .card-front {
  opacity: 1;
  transform: rotateY(0deg);
}
</style>
