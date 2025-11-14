# Skip Day - 開發任務清單

**Feature**: Skip Day 🔮 - 占卜請假機  
**Branch**: `001-should-take-leave`  
**Tech Stack**: Vue 3 + Vite + SCSS  
**Date**: 2025-11-14

---

## 📊 任務總覽

- **總任務數**: 21 個
- **Phase 1 (設置)**: 3 個任務
- **Phase 2 (基礎)**: 2 個任務  
- **Phase 3 (US1 - 首頁)**: 3 個任務
- **Phase 4 (US2 - 抽牌)**: 5 個任務
- **Phase 5 (US3 - 理由 & 語氣)**: 4 個任務
- **Phase 6 (US4 - 複製)**: 3 個任務
- **Phase 7 (完善)**: 1 個任務

**可並行任務**: 6 個（標記為 [P]）

---

## 🚀 Phase 1: 設置 (Setup)

- [ ] T001 初始化 Vue 3 + Vite 項目結構（npm create vite@latest skip-day -- --template vue）
- [ ] T002 建立 `src/` 目錄結構：components/、data/、assets/styles/
- [ ] T003 建立 `vite.config.js` 和 `main.js` 入口點

---

## 📦 Phase 2: 基礎 (Foundational)

這些任務必須在任何 User Story 之前完成。

- [ ] T004 [P] 建立 `src/data/cards.js` 並寫入 5 張卡牌資料（cardId, name, fortune）
- [ ] T005 [P] 建立 `src/data/reasons.js` 並寫入 15 個理由模板 + `getReason()` 輔助函數

---

## 👤 Phase 3: User Story 1 - 首頁展示

**目標**: 使用者進入首頁看到「Skip Day」標題和「Start My Fate 🔮」按鈕

**獨立測試標準**:
- 頁面加載時顯示 "Skip Day 🔮" 標題
- 顯示副標語 "Tap the cards. Escape reality."
- 按鈕文字正確，點擊可以觸發事件

**實施任務**:

- [ ] T006 [US1] 建立 `src/components/HomePage.vue` 組件（標題、副標語、按鈕模板）
- [ ] T007 [US1] 建立 `src/App.vue` 主容器（狀態管理、組件切換邏輯）
- [ ] T008 [US1] 建立 `src/assets/styles/main.scss` 首頁基礎樣式（顏色、排版、按鈕樣式）

---

## 🎴 Phase 4: User Story 2 - 抽牌與卡牌翻轉

**目標**: 使用者點「開始」後看到 3-5 張卡牌，點擊卡牌後翻轉並顯示卡牌名稱

**獨立測試標準**:
- 顯示 3-5 張背面卡牌（✨ 或其他符號）
- 點擊卡牌後在 600ms 內翻轉
- 翻轉後顯示卡牌名稱
- 翻轉後出現「生成理由 📄」和「再抽一次 🔁」按鈕

**實施任務**:

- [ ] T009 [P] [US2] 建立 `src/components/DrawCards.vue` 組件（卡牌網格、隨機選擇、點擊邏輯）
- [ ] T010 [P] [US2] 建立 `src/components/CardResult.vue` 組件（卡牌顯示、訓語、按鈕）
- [ ] T011 [US2] 實現卡牌翻轉 3D 動畫（CSS perspective + rotateY）於 `src/assets/styles/main.scss`
- [ ] T012 [US2] 在 `src/App.vue` 中整合 DrawCards → CardResult 的狀態流（currentPage、selectedCard、handleCardSelected）
- [ ] T013 [US2] 建立卡牌動畫的輔助函數 `shuffle()` 於 `src/utils/helpers.js`

---

## 💬 Phase 5: User Story 3 - 理由生成與語氣選擇

**目標**: 使用者點「生成理由 📄」後看到請假理由文本，可選擇三種語氣（平靜/中二/社恐），文本即時更新

**獨立測試標準**:
- 點擊「生成理由」按鈕後進入理由畫面
- 顯示根據卡牌 ID 生成的請假理由文本
- 顯示三個語氣選項按鈕
- 點擊不同語氣時，理由文本即時更新
- 當前選中的語氣按鈕有視覺高亮

**實施任務**:

- [ ] T014 [P] [US3] 建立 `src/components/ExcuseScreen.vue` 組件（理由展示、語氣選擇器、複製按鈕）
- [ ] T015 [US3] 在 `src/App.vue` 中新增 `selectedTone` 狀態變數並實現 tone-change 事件處理
- [ ] T016 [US3] 在 `src/assets/styles/main.scss` 中新增理由畫面樣式（理由框、語氣按鈕、選中狀態）
- [ ] T017 [US3] 實現語氣選擇器的即時文本更新邏輯（使用 `getReason(cardId, tone)` 函數）

---

## 📋 Phase 6: User Story 4 - 複製功能

**目標**: 使用者點「複製 ✂️」按鈕後理由複製到剪貼簿，顯示 2 秒的「已複製！」提示

**獨立測試標準**:
- 點擊「複製 ✂️」按鈕後文本被複製到剪貼簿
- 顯示「已複製！」提示信息
- 2 秒後提示自動消失，按鈕文字恢復為「複製 ✂️」
- 複製功能在 Chrome、Safari、Firefox、Edge 中可用

**實施任務**:

- [ ] T018 [P] [US4] 在 `src/utils/clipboard.js` 中建立 `copyToClipboard()` 函數（Clipboard API + fallback）
- [ ] T019 [US4] 在 `src/components/ExcuseScreen.vue` 中實現複製按鈕邏輯和 2 秒 Toast 提示
- [ ] T020 [US4] 在 `src/assets/styles/main.scss` 中新增複製狀態樣式（按鈕文字變化、Toast 動畫）

---

## ✨ Phase 7: 完善與收尾

- [ ] T021 全局樣式微調、響應式設計測試（mobile 375px 以上）、免責聲明添加

---

## 📈 依賴關係圖

```
Phase 1: 初始化
  ↓
Phase 2: 資料檔案 (cards.js, reasons.js)
  ↓
Phase 3: US1 首頁 ────┐
         (HomePage)   │
                      ↓
Phase 4: US2 抽牌 ← Phase 3 完成
         (DrawCards, CardResult)
         ↓
Phase 5: US3 理由 ← Phase 4 完成
         (ExcuseScreen, tone selection)
         ↓
Phase 6: US4 複製 ← Phase 5 完成
         (copy button, toast)
         ↓
Phase 7: 完善
```

---

## 🔄 並行執行機會

**可以同時進行的任務對**:

1. **T004 & T005**: 建立 `cards.js` 和 `reasons.js`（兩個獨立資料檔案）

2. **T009 & T010**: 建立 `DrawCards.vue` 和 `CardResult.vue`（兩個獨立組件，都需要在 T012 時整合）

3. **T014 & 其他**: 一旦 Phase 4 完成，`ExcuseScreen.vue` 可以與 Phase 4 的任務並行開發，最後在 T015 時整合

4. **T018 & 其他**: `clipboard.js` 可以獨立開發，最後在 T019 時使用

**建議並行時間表**:
- **Day 1**: Phase 1-2（T001-T005）+ Phase 3（T006-T008）並行開始
- **Day 2**: Phase 4（T009-T013）
- **Day 3**: Phase 5（T014-T017）
- **Day 4**: Phase 6（T018-T020）
- **Day 5**: Phase 7（T021）+ 測試

**單人快速路徑**: 順序執行 T001-T021，預計 3-5 天

---

## 🎯 MVP 範圍

**建議 MVP（最小可行產品）**: 完成 Phase 1-6

- ✅ 使用者可以進入首頁
- ✅ 使用者可以點擊開始並抽牌
- ✅ 使用者可以看到卡牌翻轉動畫
- ✅ 使用者可以生成請假理由
- ✅ 使用者可以選擇三種語氣
- ✅ 使用者可以複製理由到剪貼簿

**Post-MVP（後續功能）**: Phase 7 完善後可考慮
- 加入更多卡牌（目前 5 張）
- 加入 GPT API 動態生成
- 分享到社群媒體
- 黑暗模式
- 多語言支持

---

## 📝 執行建議

**優先順序**:

1. 按 Phase 順序執行（1 → 2 → 3 → 4 → 5 → 6 → 7）
2. 每個 Phase 完成後可以獨立測試（使用測試標準）
3. 如果有並行資源，優先並行「資料檔案」和「組件」開發

**檢查清單完成後**:

- [ ] npm install 安裝所有依賴
- [ ] npm run dev 啟動開發伺服器
- [ ] 測試每個 User Story 的獨立測試標準
- [ ] npm run build 確保生產構建成功
- [ ] 部署到 Vercel 或 Netlify

---

**開始編碼吧！** 🚀

