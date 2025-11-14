<template>
  <div class="excuse-screen">
    <h2>{{ card.name }}</h2>

    <!-- 理由文本框 -->
    <div class="reason-box">
      <p>{{ currentReason }}</p>
    </div>

    <!-- 語氣選擇器 -->
    <div class="tone-selector">
      <button
        v-for="t in tones"
        :key="t"
        class="tone-btn"
        :class="{ active: tone === t }"
        @click="$emit('tone-change', t)"
      >
        {{ tonEmoji[t] }} {{ t }}
      </button>
    </div>

    <!-- 複製按鈕 -->
    <button
      class="copy-btn"
      @click="handleCopy"
      :class="{ copied: isCopied }"
    >
      {{ isCopied ? '已複製！✨' : '複製 ✂️' }}
    </button>

    <!-- 導航按鈕 -->
    <div class="nav-buttons">
      <button class="nav-btn" @click="$emit('back')">回首頁 🏠</button>
      <button class="nav-btn" @click="$emit('again')">再抽一次 🔁</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { getReason } from '../data/reasons'

const props = defineProps({
  card: {
    type: Object,
    required: true,
  },
  tone: {
    type: String,
    required: true,
  },
})

const emit = defineEmits(['tone-change', 'back', 'again'])

const tones = ['平靜', '中二', '社恐']
const tonEmoji = {
  '平靜': '😌',
  '中二': '🎭',
  '社恐': '😅',
}

const isCopied = ref(false)

const currentReason = computed(() => {
  return getReason(props.card.cardId, props.tone)
})

const handleCopy = async () => {
  try {
    await navigator.clipboard.writeText(currentReason.value)
    isCopied.value = true
    setTimeout(() => {
      isCopied.value = false
    }, 2000)
  } catch (err) {
    console.error('複製失敗:', err)
    // 備選方案：使用 document.execCommand
    const textarea = document.createElement('textarea')
    textarea.value = currentReason.value
    document.body.appendChild(textarea)
    textarea.select()
    document.execCommand('copy')
    document.body.removeChild(textarea)

    isCopied.value = true
    setTimeout(() => {
      isCopied.value = false
    }, 2000)
  }
}
</script>

<style scoped>
.excuse-screen {
  padding: 20px 0;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

h2 {
  text-align: center;
  color: #6b4ce6;
  margin-bottom: 30px;
  font-size: 28px;
  font-weight: 800;
}

.reason-box {
  background: #fafafa;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 30px;
  min-height: 100px;
  display: flex;
  align-items: center;
  border-left: 4px solid #6b4ce6;
}

.reason-box p {
  font-size: 16px;
  line-height: 1.6;
  color: #222;
}

.tone-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
}

.tone-btn {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  background: white;
  color: #666;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
  font-size: 14px;
}

.tone-btn:hover {
  border-color: #6b4ce6;
  color: #6b4ce6;
}

.tone-btn.active {
  background: #6b4ce6;
  color: white;
  border-color: #6b4ce6;
  box-shadow: 0 4px 12px rgba(107, 76, 230, 0.3);
}

.copy-btn {
  width: 100%;
  padding: 14px;
  font-size: 16px;
  background: linear-gradient(135deg, #6b4ce6, #f0a500);
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-bottom: 20px;
  font-weight: bold;
}

.copy-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(107, 76, 230, 0.3);
}

.copy-btn:active {
  transform: translateY(0);
}

.copy-btn.copied {
  background: #4caf50;
}

.nav-buttons {
  display: flex;
  gap: 10px;
}

.nav-btn {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  background: white;
  color: #666;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
}

.nav-btn:hover {
  border-color: #6b4ce6;
  color: #6b4ce6;
}
</style>
