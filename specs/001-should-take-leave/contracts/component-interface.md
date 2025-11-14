# Component Interface Contracts

**Feature**: Skip Day - 占卜請假機  
**Date**: 2025-11-14  
**Note**: Though this is a client-only app with no backend API, we define component contracts to enforce data flow consistency and simplify testing.

---

## 1. HomePage Component

**Purpose**: Entry point; displays Skip Day branding and call-to-action button.

**Props** (Input):
```typescript
interface HomePageProps {
  // None - HomePage is root component
}
```

**Emits** (Output):
```typescript
interface HomePageEmits {
  'start-game': void; // Emitted when user clicks "Start My Fate 🔮"
}
```

**Internal State**:
```typescript
{
  // None - stateless presentation
}
```

**Behavior**:
- Display "Skip Day" title
- Display "Tap the cards. Escape reality." subtitle
- Display "Start My Fate 🔮" button
- On click: emit 'start-game' event to parent (App.vue)
- Load time requirement: ≤1.5s (SC-001)

**Usage in App.vue**:
```vue
<HomePage @start-game="currentPage = 'draw'" />
```

---

## 2. DrawCard (Tarot Draw Screen)

**Purpose**: Display 3-5 shuffled cards; let user select one to flip.

**Props** (Input):
```typescript
interface DrawCardProps {
  cards: TarotCard[];        // Array of 5 available cards
  selectedCard: TarotCard | null;  // Currently selected card (if any)
  isFlipping: boolean;       // Is flip animation in progress?
}
```

**Emits** (Output):
```typescript
interface DrawCardEmits {
  'card-selected': TarotCard;       // User clicked a card
  'flip-complete': TarotCard;       // Flip animation finished
  'draw-again': void;               // User wants to draw again (from result screen)
}
```

**Internal State**:
```typescript
{
  shuffledCards: TarotCard[];  // Shuffled order for this draw
  displayedCard: TarotCard | null; // Card shown during flip
}
```

**Behavior**:
- On mount: Shuffle `cards` array, store in `shuffledCards`
- Display 3-5 cards in grid (back-side visible)
- On user click card: Emit 'card-selected' with card object
- Trigger flip animation (CSS transition)
- After ~600ms: Emit 'flip-complete'
- Display card details (name, description, image)
- Show buttons: "Generate Excuse Letter 📄" and "Draw Again 🔁"
- Flip time requirement: ≤1s (SC-002)

**Usage in App.vue**:
```vue
<DrawCard 
  :cards="tarotCards"
  :selected-card="selectedCard"
  :is-flipping="isCardFlipping"
  @card-selected="onCardSelected"
  @flip-complete="onFlipComplete"
  @draw-again="onDrawAgain"
/>
```

---

## 3. CardFlip (Reusable Card Component)

**Purpose**: Encapsulate card flip animation and display logic.

**Props** (Input):
```typescript
interface CardFlipProps {
  card: TarotCard;           // Card to display
  isFlipped: boolean;        // Should card be flipped (showing front)?
  showDetails: boolean;      // Show name/description or just visual?
}
```

**Emits** (Output):
```typescript
interface CardFlipEmits {
  'click': void;             // User clicked the card
}
```

**Internal State**:
```typescript
{
  // None - parent manages flip state
}
```

**Behavior**:
- Display card back if `isFlipped === false`
- Display card front (with details) if `isFlipped === true`
- CSS 3D transform for flip animation
- On click: emit 'click' event (parent handles animation trigger)
- Smooth 600ms transition

**CSS Animation Requirement**:
```scss
.card-container {
  perspective: 1000px;
  transition: transform 0.6s ease-in-out;
  
  &.flipped {
    transform: rotateY(180deg);
  }
}
```

**Usage in DrawCard.vue**:
```vue
<CardFlip 
  v-for="card in shuffledCards"
  :key="card.cardId"
  :card="card"
  :is-flipped="card.cardId === selectedCard?.cardId"
  :show-details="true"
  @click="onCardClick(card)"
/>
```

---

## 4. ExcuseGenerator (Reason Generation Screen)

**Purpose**: Display generated excuse and tone selection UI.

**Props** (Input):
```typescript
interface ExcuseGeneratorProps {
  card: TarotCard;           // Selected card
  currentTone: '平靜' | '中二' | '社恐'; // Current tone selection
  reasonText: string;        // Generated reason for card + tone
}
```

**Emits** (Output):
```typescript
interface ExcuseGeneratorEmits {
  'tone-changed': '平靜' | '中二' | '社恐'; // User selected tone
  'copy-clicked': string;    // User clicked copy (passes text to copy)
  'back-home': void;         // User clicked "Back to Home 🔙"
  'draw-again': void;        // User clicked "Draw Again 🔁"
}
```

**Internal State**:
```typescript
{
  isCopying: boolean;        // Show loading state during copy?
  copySuccess: boolean;      // Show success message?
}
```

**Behavior**:
- Display selected card name & illustration (small preview)
- Display reason text in large, readable font
- Display 3 tone selection buttons (平靜 | 中二 | 社恐)
- Highlight current selection
- On tone click: Emit 'tone-changed' + update `reasonText` in parent
- On copy click: Emit 'copy-clicked' with `reasonText`
- Show "已複製" toast for 2s (SC-004)
- Real-time text update (no delay between tone selection and display)

**Usage in App.vue**:
```vue
<ExcuseGenerator 
  :card="selectedCard"
  :current-tone="selectedTone"
  :reason-text="generatedReason"
  @tone-changed="onToneChanged"
  @copy-clicked="onCopyClicked"
  @back-home="resetGame"
  @draw-again="currentPage = 'draw'"
/>
```

---

## 5. App (Root Component)

**Purpose**: State management and page routing.

**State**:
```typescript
interface AppState {
  currentPage: 'home' | 'draw' | 'result' | 'excuse';
  selectedCard: TarotCard | null;
  selectedTone: '平靜' | '中二' | '社恐';
  generatedReason: string;
  isCardFlipping: boolean;
  isCopied: boolean;
}
```

**Child Components**:
- `HomePage`
- `DrawCard`
- `CardFlip`
- `ExcuseGenerator`

**Event Handlers** (Flow):
```javascript
// Home → Draw
onStartGame() {
  this.currentPage = 'draw'
}

// Draw → Result (card selected)
onCardSelected(card) {
  this.selectedCard = card
  this.isCardFlipping = true
}

// Result (after flip)
onFlipComplete() {
  this.isCardFlipping = false
  this.currentPage = 'result'
}

// Result → Excuse (generate)
onGenerateExcuse() {
  const reason = getReason(this.selectedCard.cardId, this.selectedTone)
  this.generatedReason = reason.reasonText
  this.currentPage = 'excuse'
}

// Excuse → Change tone (update text)
onToneChanged(tone) {
  this.selectedTone = tone
  const reason = getReason(this.selectedCard.cardId, tone)
  this.generatedReason = reason.reasonText
}

// Excuse → Copy
async onCopyClicked(text) {
  const success = await copyToClipboard(text)
  if (success) {
    this.isCopied = true
    setTimeout(() => { this.isCopied = false }, 2000)
  }
}

// Reset (back to home or draw again)
resetGame() {
  this.currentPage = 'home'
  this.selectedCard = null
  this.selectedTone = '平靜'
  this.generatedReason = ''
  this.isCardFlipping = false
  this.isCopied = false
}
```

---

## 6. Data Contracts

### Input Data Contract (Immutable)

**TarotCards Repository**:
```javascript
// src/data/tarotCards.js
export const tarotCards = [
  {
    cardId: 1,
    cardName: string,        // Non-empty, 1-30 chars
    description: string,     // Non-empty, 1-100 chars
    illustration: string,    // Valid file path
  },
  // ... exactly 5 cards
];

// Validator
export const validateTarotCards = (cards) => {
  if (!Array.isArray(cards) || cards.length !== 5) return false;
  return cards.every(card => card.cardId && card.cardName && card.description);
};
```

**ReasonTemplates Repository**:
```javascript
// src/data/reasonTemplates.js
export const reasonTemplates = {
  '1-平靜': { cardId: 1, tone: '平靜', reasonText: string },
  '1-中二': { cardId: 1, tone: '中二', reasonText: string },
  '1-社恐': { cardId: 1, tone: '社恐', reasonText: string },
  // ... 3 templates per card (5 cards = 15 total)
};

// Accessor
export const getReason = (cardId, tone) => {
  const key = `${cardId}-${tone}`;
  return reasonTemplates[key] || null;
};

// Validator
export const validateReasonTemplates = (templates) => {
  const keys = Object.keys(templates);
  if (keys.length !== 15) return false;  // 5 cards × 3 tones
  
  for (let i = 1; i <= 5; i++) {
    for (const tone of ['平靜', '中二', '社恐']) {
      if (!templates[`${i}-${tone}`]) return false;
    }
  }
  return true;
};
```

### Output Data Contract (Copy to Clipboard)

**Clipboard Content**:
```
Text only, plain format (no HTML)
Content: ReasonTemplate.reasonText
Length: 50-500 characters
Example: "我今天身體有些不適，需要休息調整。感謝理解。"
```

---

## 7. Validation & Error Handling

**Load-Time Validation**:
```javascript
// In App.vue mounted()
const cardsValid = validateTarotCards(tarotCards);
const templatesValid = validateReasonTemplates(reasonTemplates);

if (!cardsValid || !templatesValid) {
  console.error('Data validation failed');
  // Display error message: "占卜失敗，請重試"
  this.showError = true;
}
```

**Copy Failure Fallback**:
```javascript
async onCopyClicked(text) {
  try {
    const success = await copyToClipboard(text);
    if (!success) throw new Error('Copy failed');
    
    // Success: show toast
    this.isCopied = true;
    setTimeout(() => { this.isCopied = false }, 2000);
  } catch (err) {
    // Fallback: show manual copy prompt
    console.error('Copy failed:', err);
    // Display: "複製失敗，請手動複製："
    // With text in a readonly textarea
  }
}
```

**Reason Not Found**:
```javascript
onToneChanged(tone) {
  const reason = getReason(this.selectedCard.cardId, tone);
  if (!reason) {
    console.error(`Reason not found for ${this.selectedCard.cardId}-${tone}`);
    this.generatedReason = '占卜失敗，請重試'; // Fallback message
  } else {
    this.generatedReason = reason.reasonText;
  }
}
```

---

## 8. Performance Contracts

| Operation | Target | SC Reference |
|-----------|--------|---------------|
| HomePage load | ≤1.5s | SC-001 |
| Card flip animation | ≤1s | SC-002 |
| Copy success rate | ≥95% | SC-003 |
| User flow completion | ≤3 min, 85% success | SC-004 |
| Reason generation | ≤100ms (local) | Not SC |
| Tone text update | Instant (no API call) | Not SC |
| Toast display | 2 seconds | FR-006 |

---

## Next Steps (Phase 2)

1. Implement components per these contracts
2. Add unit tests for each component
3. Add E2E tests for full user flow
4. Validate all success criteria metrics

