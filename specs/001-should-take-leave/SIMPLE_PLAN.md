# Skip Day - 極簡開發計畫

直接看這個檔案開始編碼。5 分鐘就能懂。

---

## 📁 檔案結構

```
src/
├── App.vue                    # 唯一的容器 + 狀態管理
├── components/
│   ├── HomePage.vue           # 首頁「Start My Fate 🔮」
│   ├── DrawCards.vue          # 抽牌畫面（3-5 張卡牌）
│   ├── CardResult.vue         # 卡牌結果（名稱 + 訓語）
│   └── ExcuseScreen.vue       # 理由顯示 + 語氣選擇 + 複製
├── data/
│   ├── cards.js               # 5 張卡牌資料
│   └── reasons.js             # 每張卡牌的 3 種語氣理由
└── assets/
    └── styles/
        └── main.scss          # 全局樣式

```

---

## 🎯 Component 架構

### App.vue（主容器 + 狀態）

**狀態變數**:
```javascript
const currentPage = ref('home')        // 'home' | 'draw' | 'result' | 'excuse'
const selectedCard = ref(null)         // 選中的卡牌物件
const selectedTone = ref('平靜')       // 當前語氣：'平靜' | '中二' | '社恐'
```

**邏輯**:
- 根據 `currentPage` 來切換顯示哪個組件
- 管理狀態傳給子組件
- 接收子組件的事件來更新狀態

**模板**:
```vue
<template>
  <div id="app">
    <!-- 根據 currentPage 顯示不同組件 -->
    <HomePage v-if="currentPage === 'home'" @start="currentPage = 'draw'" />
    <DrawCards v-else-if="currentPage === 'draw'" 
               :cards="allCards"
               @selected="handleCardSelected" />
    <CardResult v-else-if="currentPage === 'result'"
                :card="selectedCard"
                @generate="currentPage = 'excuse'"
                @again="currentPage = 'draw'" />
    <ExcuseScreen v-else-if="currentPage === 'excuse'"
                  :card="selectedCard"
                  :tone="selectedTone"
                  @tone-change="selectedTone = $event"
                  @back="currentPage = 'home'"
                  @again="currentPage = 'draw'" />
  </div>
</template>
```

---

### HomePage.vue

**功能**: 顯示標題 + 「Start My Fate 🔮」按鈕

**Props**: 無

**Emits**: `@start`

**模板**:
```vue
<template>
  <div class="home-page">
    <h1>Skip Day 🔮</h1>
    <p>Tap the cards. Escape reality.</p>
    <button @click="$emit('start')">Start My Fate 🔮</button>
  </div>
</template>
```

---

### DrawCards.vue

**功能**: 顯示 3-5 張背面卡牌，點擊後翻轉並發送事件

**Props**:
```javascript
cards: Array  // 所有卡牌
```

**Emits**: `@selected` (傳送選中的卡牌物件)

**邏輯**:
```javascript
const visibleCards = ref([])
const flippedCardId = ref(null)
const isAnimating = ref(false)

onMounted(() => {
  // 隨機選 3-5 張卡牌
  visibleCards.value = shuffle(props.cards).slice(0, Math.random() * 3 + 3)
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
```

**模板**:
```vue
<template>
  <div class="draw-cards">
    <div class="cards-grid">
      <div v-for="card in visibleCards"
           :key="card.cardId"
           class="card"
           :class="{ flipped: flippedCardId === card.cardId }"
           @click="handleCardClick(card)">
        <!-- 卡牌背面 -->
        <div class="card-back">🔮</div>
        <!-- 卡牌正面 -->
        <div class="card-front">{{ card.name }}</div>
      </div>
    </div>
  </div>
</template>
```

---

### CardResult.vue

**功能**: 顯示卡牌名稱 + 訓語 + 兩個按鈕

**Props**:
```javascript
card: Object  // { cardId, name, fortune }
```

**Emits**: `@generate`, `@again`

**模板**:
```vue
<template>
  <div class="card-result">
    <div class="card-display">
      <h2>{{ card.name }}</h2>
      <p class="fortune">{{ card.fortune }}</p>
    </div>
    <div class="buttons">
      <button @click="$emit('generate')">生成理由 📄</button>
      <button @click="$emit('again')">再抽一次 🔁</button>
    </div>
  </div>
</template>
```

---

### ExcuseScreen.vue

**功能**: 顯示理由 + 語氣選擇器 + 複製按鈕

**Props**:
```javascript
card: Object      // 當前卡牌
tone: String      // 當前語氣
```

**Emits**: `@tone-change`, `@back`, `@again`

**邏輯**:
```javascript
const tones = ['平靜', '中二', '社恐']
const isCopied = ref(false)

const handleToneChange = (newTone) => {
  emit('tone-change', newTone)
}

const handleCopy = () => {
  const reasonText = getReason(props.card.cardId, props.tone)
  navigator.clipboard.writeText(reasonText)
  
  isCopied.value = true
  setTimeout(() => {
    isCopied.value = false
  }, 2000)
}
```

**模板**:
```vue
<template>
  <div class="excuse-screen">
    <h2>{{ card.name }}</h2>
    
    <!-- 理由文本 -->
    <div class="reason-box">
      <p>{{ getReason(card.cardId, tone) }}</p>
    </div>
    
    <!-- 語氣選擇 -->
    <div class="tone-selector">
      <button v-for="t in tones"
              :key="t"
              :class="{ active: tone === t }"
              @click="handleToneChange(t)">
        {{ t }}
      </button>
    </div>
    
    <!-- 複製按鈕 -->
    <button @click="handleCopy" class="copy-btn">
      {{ isCopied ? '已複製！' : '複製 ✂️' }}
    </button>
    
    <!-- 返回按鈕 -->
    <div class="nav-buttons">
      <button @click="$emit('back')">回首頁 🏠</button>
      <button @click="$emit('again')">再抽一次 🔁</button>
    </div>
  </div>
</template>
```

---

## 📊 資料檔案結構

### data/cards.js

```javascript
export const cards = [
  {
    cardId: 1,
    name: '懶骨頭之星',
    fortune: '給我請假，現在。',
  },
  {
    cardId: 2,
    name: '摸魚隱者',
    fortune: '偷閒是一種修行。',
  },
  {
    cardId: 3,
    name: '社恐聖杯騎士',
    fortune: '退縮於人群之河。',
  },
  {
    cardId: 4,
    name: '鹹魚女皇',
    fortune: '朕決定擺爛。',
  },
  {
    cardId: 5,
    name: '逃避傻瓜',
    fortune: '命運叫我別上班。',
  },
]
```

### data/reasons.js

```javascript
export const reasons = {
  // 卡牌 1：懶骨頭之星
  '1-平靜': '我今天身體有些不適，需要休息調整。感謝理解。',
  '1-中二': '懶骨頭之星降臨，吾已被選中，必須靜修一日方能恢復力量。',
  '1-社恐': '...不好意思，今天狀態不太好，能否...請假？',
  
  // 卡牌 2：摸魚隱者
  '2-平靜': '有些臨時安排需要處理，今天可能無法上班。',
  '2-中二': '摸魚之術已啟動，本日必須修行以精進功法。',
  '2-社恐': '呃...今天有點累，想請個假可以嗎...？',
  
  // 卡牌 3：社恐聖杯騎士
  '3-平靜': '因為個人因素需要調整，今天請假一天。',
  '3-中二': '社恐魔力發動，人群催眠開始，必須隱居一日。',
  '3-社恐': '...人太多了，我...我需要靜一靜，可以請假嗎...？',
  
  // 卡牌 4：鹹魚女皇
  '4-平靜': '今日身心俱疲，決定休息一天以恢復精力。',
  '4-中二': '朕已頒布聖旨，全國放假一天，包括朕自己。',
  '4-社恐': '...實在撐不住了...能讓我休息嗎...拜託...',
  
  // 卡牌 5：逃避傻瓜
  '5-平靜': '有急事需要處理，今天無法如期上班。',
  '5-中二': '傻瓜的逃避之道已示現，今日必定逃脫枷鎖！',
  '5-社恐': '我...我想請假...今天不行嗎...？',
}

// 輔助函數
export const getReason = (cardId, tone) => {
  return reasons[`${cardId}-${tone}`] || '占卜失敗，請重試'
}
```

---

## 🎨 SCSS 規劃

### assets/styles/main.scss

```scss
// ============ 顏色 & 變數 ============
$primary: #6b4ce6;        // 紫色（占卜主題）
$secondary: #f0a500;      // 金色
$text-dark: #222;
$text-light: #666;
$bg-light: #fafafa;
$white: #fff;

$transition: all 0.3s ease;

// ============ 全局樣式 ============
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, $primary, #a78bfa);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

#app {
  background: $white;
  border-radius: 20px;
  padding: 40px;
  max-width: 600px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

// ============ HomePage.vue ============
.home-page {
  text-align: center;
  
  h1 {
    font-size: 48px;
    color: $primary;
    margin-bottom: 10px;
  }
  
  p {
    font-size: 18px;
    color: $text-light;
    margin-bottom: 40px;
  }
  
  button {
    padding: 16px 32px;
    font-size: 18px;
    background: linear-gradient(135deg, $primary, $secondary);
    color: $white;
    border: none;
    border-radius: 50px;
    cursor: pointer;
    transition: $transition;
    box-shadow: 0 8px 20px rgba($primary, 0.3);
    
    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 12px 30px rgba($primary, 0.4);
    }
    
    &:active {
      transform: translateY(-2px);
    }
  }
}

// ============ DrawCards.vue ============
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
  
  > div {
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    font-size: 40px;
    font-weight: bold;
    transition: opacity 0.6s ease;
    backface-visibility: hidden;
  }
  
  .card-back {
    background: linear-gradient(135deg, $primary, $secondary);
    color: $white;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
  }
  
  .card-front {
    background: $bg-light;
    color: $text-dark;
    opacity: 0;
    transform: rotateY(180deg);
    font-size: 12px;
    padding: 10px;
    text-align: center;
    line-height: 1.2;
  }
  
  &.flipped {
    .card-back {
      opacity: 0;
      transform: rotateY(180deg);
    }
    
    .card-front {
      opacity: 1;
      transform: rotateY(0deg);
    }
  }
}

// ============ CardResult.vue ============
.card-result {
  text-align: center;
  padding: 30px 0;
}

.card-display {
  margin-bottom: 40px;
  
  h2 {
    font-size: 36px;
    color: $primary;
    margin-bottom: 10px;
  }
  
  .fortune {
    font-size: 20px;
    color: $secondary;
    font-style: italic;
  }
}

.buttons {
  display: flex;
  gap: 15px;
  
  button {
    flex: 1;
    padding: 12px 20px;
    font-size: 16px;
    border: 2px solid $primary;
    background: $white;
    color: $primary;
    border-radius: 25px;
    cursor: pointer;
    transition: $transition;
    
    &:hover {
      background: $primary;
      color: $white;
      transform: translateY(-2px);
    }
  }
}

// ============ ExcuseScreen.vue ============
.excuse-screen {
  padding: 20px 0;
  
  h2 {
    text-align: center;
    color: $primary;
    margin-bottom: 30px;
    font-size: 28px;
  }
}

.reason-box {
  background: $bg-light;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 30px;
  min-height: 100px;
  display: flex;
  align-items: center;
  
  p {
    font-size: 16px;
    line-height: 1.6;
    color: $text-dark;
  }
}

.tone-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
  
  button {
    flex: 1;
    padding: 12px;
    border: 2px solid $primary;
    background: $white;
    color: $primary;
    border-radius: 20px;
    cursor: pointer;
    transition: $transition;
    font-weight: bold;
    
    &:hover {
      background: rgba($primary, 0.1);
    }
    
    &.active {
      background: $primary;
      color: $white;
    }
  }
}

.copy-btn {
  width: 100%;
  padding: 14px;
  font-size: 16px;
  background: linear-gradient(135deg, $primary, $secondary);
  color: $white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: $transition;
  margin-bottom: 20px;
  font-weight: bold;
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba($primary, 0.3);
  }
  
  &:active {
    transform: translateY(0);
  }
}

.nav-buttons {
  display: flex;
  gap: 10px;
  
  button {
    flex: 1;
    padding: 12px;
    border: 2px solid $text-light;
    background: $white;
    color: $text-light;
    border-radius: 20px;
    cursor: pointer;
    transition: $transition;
    
    &:hover {
      border-color: $primary;
      color: $primary;
    }
  }
}

// ============ 響應式設計 ============
@media (max-width: 480px) {
  #app {
    padding: 20px;
  }
  
  .home-page h1 {
    font-size: 36px;
  }
  
  .card-display h2 {
    font-size: 28px;
  }
  
  .cards-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

---

## 🔄 互動流程串連

```
1. App.vue 初始化
   ↓
2. HomePage 顯示 → 用戶點「Start My Fate 🔮」
   ↓ emit('start')
3. App 更新 currentPage = 'draw'
   ↓
4. DrawCards 顯示隨機卡牌 → 用戶點卡牌
   ↓ emit('selected', card)
5. App 更新 selectedCard → 觸發翻轉動畫（600ms）
   ↓
6. App 更新 currentPage = 'result'
   ↓
7. CardResult 顯示卡牌 + 訓語 → 用戶點「生成理由 📄」
   ↓ emit('generate')
8. App 更新 currentPage = 'excuse'
   ↓
9. ExcuseScreen 顯示理由 + 語氣選擇器
   ↓
10a. 用戶選語氣 → emit('tone-change')
     App 更新 selectedTone → 理由即時更新
     
10b. 用戶點「複製 ✂️」
     App 調用 getReason() 複製到剪貼簿
     顯示 2 秒「已複製！」
     
10c. 用戶點「再抽一次 🔁」
     App 更新 currentPage = 'draw'
     回到第 4 步
     
10d. 用戶點「回首頁 🏠」
     App 更新 currentPage = 'home'
     回到第 2 步
```

---

## 🚀 開始編碼的順序

1. **先寫 data 檔案**（cards.js 和 reasons.js）
2. **再寫 HomePage.vue**（最簡單）
3. **寫 DrawCards.vue**（有翻轉動畫）
4. **寫 CardResult.vue**（簡單顯示）
5. **寫 ExcuseScreen.vue**（語氣切換 + 複製）
6. **寫 App.vue**（主容器 + 狀態管理）
7. **寫 SCSS**（最後調整樣式）

每個 component 獨立，可以分開開發，最後在 App.vue 組裝。

---

就醬！不需要更多東西了，開始編碼吧 🔥

