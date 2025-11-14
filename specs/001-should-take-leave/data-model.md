# Phase 1 Design: Data Model & Entity Schema

**Feature**: Skip Day - 占卜請假機  
**Branch**: `001-should-take-leave`  
**Date**: 2025-11-14  

## Overview

This document defines the complete data model for Skip Day, including entity schemas, relationships, validation rules, and state transitions.

---

## 1. Core Entities

### 1.1 TarotCard

Represents a tarot card available for selection in the draw phase.

**Schema**:
```typescript
interface TarotCard {
  // Identity
  cardId: number;              // Unique identifier (1-5 for MVP, scalable to N)
  
  // Display
  cardName: string;            // E.g., "懶骨頭之星", "摸魚隱者"
  description: string;         // One-liner fortune, E.g., "給我請假，現在。"
  illustration: string;        // Image URL (SVG or PNG, relative path acceptable)
  
  // Metadata
  createdAt: ISO8601DateTime;  // Timestamp when card was added (optional for MVP)
  updatedAt: ISO8601DateTime;  // Last modification (optional for MVP)
}
```

**Sample Data**:
```javascript
export const tarotCards = [
  {
    cardId: 1,
    cardName: '懶骨頭之星',
    description: '給我請假，現在。',
    illustration: '/images/cards/lazy-star.svg',
  },
  {
    cardId: 2,
    cardName: '摸魚隱者',
    description: '偷閒是一種修行。',
    illustration: '/images/cards/slacker-hermit.svg',
  },
  {
    cardId: 3,
    cardName: '社恐聖杯騎士',
    description: '退縮於人群之河。',
    illustration: '/images/cards/introvert-knight.svg',
  },
  {
    cardId: 4,
    cardName: '鹹魚女皇',
    description: '朕決定擺爛。',
    illustration: '/images/cards/sloth-empress.svg',
  },
  {
    cardId: 5,
    cardName: '逃避傻瓜',
    description: '命運叫我別上班。',
    illustration: '/images/cards/escape-fool.svg',
  },
];
```

**Validation Rules**:
- `cardId`: Must be positive integer, unique within collection
- `cardName`: Non-empty string, 1-30 characters, no null/undefined
- `description`: Non-empty string, 1-100 characters, must convey fortune/advice
- `illustration`: Valid file path or URL, file must exist at build time

**State Transitions**:
```
[Card in Deck] → [Selected by User] → [Displayed (Flipped)] → [Result Shown with Buttons]
```

---

### 1.2 ReasonTemplate

Represents a pre-written excuse/reason for a given card and tone combination.

**Schema**:
```typescript
interface ReasonTemplate {
  // Identity
  reasonId: string;            // Composite: `${cardId}-${tone}` (e.g., "1-平靜")
  cardId: number;              // Foreign key to TarotCard
  tone: '平靜' | '中二' | '社恐'; // Tone style
  
  // Content
  reasonText: string;          // Generated excuse for absence (200-500 chars)
  
  // Metadata (optional for MVP)
  author?: string;             // Who wrote/approved this reason
  createdAt?: ISO8601DateTime;
}
```

**Sample Data**:
```javascript
export const reasonTemplates = {
  '1-平靜': {
    reasonId: '1-平靜',
    cardId: 1,
    tone: '平靜',
    reasonText: '我今天身體有些不適，需要休息調整。感謝理解。',
  },
  '1-中二': {
    reasonId: '1-中二',
    cardId: 1,
    tone: '中二',
    reasonText: '懶骨頭之星降臨，吾已被選中，必須靜修一日方能恢復力量。',
  },
  '1-社恐': {
    reasonId: '1-社恐',
    cardId: 1,
    tone: '社恐',
    reasonText: '...不好意思，今天狀態不太好，能否...請假？感謝...',
  },
  // ... repeat for cards 2-5
};
```

**Complete Mapping** (15 templates for MVP):
| cardId | cardName | 平靜 | 中二 | 社恐 |
|--------|----------|------|------|------|
| 1 | 懶骨頭之星 | ✓ | ✓ | ✓ |
| 2 | 摸魚隱者 | ✓ | ✓ | ✓ |
| 3 | 社恐聖杯騎士 | ✓ | ✓ | ✓ |
| 4 | 鹹魚女皇 | ✓ | ✓ | ✓ |
| 5 | 逃避傻瓜 | ✓ | ✓ | ✓ |

**Validation Rules**:
- `reasonId`: Format: `${cardId}-${tone}`, must match cardId in TarotCard collection
- `cardId`: Must reference valid TarotCard (FK constraint)
- `tone`: Must be one of exactly three enum values
- `reasonText`: Non-empty string, 50-500 characters, must be appropriate in tone/context

**State Transitions**:
```
[Template Stored] → [Loaded on Excuse Screen] → [Displayed to User] → [Copied to Clipboard]
```

---

## 2. Runtime State (In-Memory)

State managed by Vue 3 Composition API, not persisted.

**Schema**:
```typescript
interface GameState {
  // Navigation
  currentPage: 'home' | 'draw' | 'result' | 'excuse';
  
  // Selection
  selectedCard: TarotCard | null;     // Currently selected card (null until drawn)
  selectedTone: '平靜' | '中二' | '社恐'; // Default: '平靜'
  
  // Content
  generatedReason: string;            // Reason text for current card + tone
  
  // UI Flags
  isCardFlipping: boolean;            // True during card flip animation
  isCopied: boolean;                  // True after copy, reverts after 2s
}
```

**Example Flow**:
```javascript
// Initial state
{
  currentPage: 'home',
  selectedCard: null,
  selectedTone: '平靜',
  generatedReason: '',
  isCardFlipping: false,
  isCopied: false,
}

// After clicking "Start My Fate"
{
  currentPage: 'draw',
  selectedCard: null,
  selectedTone: '平靜',
  generatedReason: '',
  isCardFlipping: false,
  isCopied: false,
}

// After clicking a card
{
  currentPage: 'draw', // still drawing
  selectedCard: { cardId: 1, ... }, // selected
  selectedTone: '平靜',
  generatedReason: '',
  isCardFlipping: true, // animation in progress
  isCopied: false,
}

// After flip animation completes
{
  currentPage: 'result',
  selectedCard: { cardId: 1, ... },
  selectedTone: '平靜',
  generatedReason: '',
  isCardFlipping: false,
  isCopied: false,
}

// After clicking "Generate Excuse"
{
  currentPage: 'excuse',
  selectedCard: { cardId: 1, ... },
  selectedTone: '平靜',
  generatedReason: '我今天身體有些不適，需要休息調整。感謝理解。',
  isCardFlipping: false,
  isCopied: false,
}

// After changing tone
{
  currentPage: 'excuse',
  selectedCard: { cardId: 1, ... },
  selectedTone: '中二', // changed
  generatedReason: '懶骨頭之星降臨，吾已被選中，必須靜修一日方能恢復力量。', // updated
  isCardFlipping: false,
  isCopied: false,
}

// After clicking Copy
{
  currentPage: 'excuse',
  selectedCard: { cardId: 1, ... },
  selectedTone: '中二',
  generatedReason: '懶骨頭之星降臨，吾已被選中，必須靜修一日方能恢復力量。',
  isCardFlipping: false,
  isCopied: true, // flips back to false after 2s
}
```

---

## 3. Data Relationships

### Relationship Diagram

```
TarotCard (1) ──→ (N) ReasonTemplate
    │
    └─ cardId (PK)
    
ReasonTemplate
    │
    ├─ cardId (FK → TarotCard.cardId)
    ├─ tone (enum)
    └─ reasonText (dependent on both)
```

### Access Patterns

**By Card**:
```javascript
// Get all reason variations for a card
const getReasonsByCard = (cardId) => {
  return Object.values(reasonTemplates).filter(t => t.cardId === cardId);
};

// Example: Card 1 → [1-平靜, 1-中二, 1-社恐]
```

**By Tone**:
```javascript
// Get all reasons in a given tone (rarely used; kept for completeness)
const getReasonsByTone = (tone) => {
  return Object.values(reasonTemplates).filter(t => t.tone === tone);
};
```

**Direct Lookup**:
```javascript
// Get specific reason
const getReason = (cardId, tone) => {
  const reasonId = `${cardId}-${tone}`;
  return reasonTemplates[reasonId];
};

// Example: getReason(1, '平靜') → {...reasonText: '我今天身體...'}
```

---

## 4. Data Loading & Initialization

### Load Order

1. **Page Load (HTML)**:
   - `src/main.js` initializes Vue app
   - Mount to `#app` div in `index.html`

2. **Vue App Mount**:
   - `src/App.vue` root component initializes
   - Call `useGameState()` composable
   - State initialized to defaults (currentPage: 'home', etc.)

3. **Data Import**:
   - `src/data/tarotCards.js` loaded (5 cards, ~2KB)
   - `src/data/reasonTemplates.js` loaded (15 reasons, ~5KB)
   - Imported into components as needed (lazy evaluation)

4. **HomePage Mount**:
   - Shuffle or pre-shuffle card order (optional randomization)
   - Ready for user interaction

**Performance Impact**:
- Total data: ~7KB (gzipped ~2KB)
- Load time: <100ms (negligible)
- Meets SC-001 (≤1.5s homepage load)

### Caching Strategy

- **Browser Cache**: Vite's production build includes content-hash in filenames; assets cached indefinitely
- **LocalStorage**: Not used (fully anonymous requirement)
- **Memory**: Runtime state in Vue refs; cleared on page refresh (desired behavior)

---

## 5. Validation & Constraints

### Field-Level Validation

**TarotCard**:
```javascript
const validateTarotCard = (card) => {
  const errors = [];
  
  if (!Number.isInteger(card.cardId) || card.cardId <= 0) {
    errors.push('cardId must be positive integer');
  }
  
  if (!card.cardName || card.cardName.length === 0 || card.cardName.length > 30) {
    errors.push('cardName must be 1-30 characters');
  }
  
  if (!card.description || card.description.length === 0 || card.description.length > 100) {
    errors.push('description must be 1-100 characters');
  }
  
  if (!card.illustration || typeof card.illustration !== 'string') {
    errors.push('illustration must be a valid path');
  }
  
  return errors.length === 0 ? null : errors;
};
```

**ReasonTemplate**:
```javascript
const validateReasonTemplate = (reason) => {
  const errors = [];
  const validTones = ['平靜', '中二', '社恐'];
  
  if (!reason.reasonId || !reason.reasonId.includes('-')) {
    errors.push('reasonId must be in format cardId-tone');
  }
  
  if (!validTones.includes(reason.tone)) {
    errors.push(`tone must be one of: ${validTones.join(', ')}`);
  }
  
  if (!reason.reasonText || reason.reasonText.length < 50 || reason.reasonText.length > 500) {
    errors.push('reasonText must be 50-500 characters');
  }
  
  return errors.length === 0 ? null : errors;
};
```

### Application-Level Constraints

1. **Uniqueness**: No duplicate cardIds or reasonIds allowed
2. **Referential Integrity**: Every ReasonTemplate.cardId must reference valid TarotCard.cardId
3. **Completeness**: Every TarotCard must have exactly 3 ReasonTemplates (one per tone)
4. **Immutability at Runtime**: Data never modified after load (no edits, deletes, or inserts during session)

---

## 6. Future Extensibility

### Schema Additions (Phase 2+)

**Content Moderation**:
```typescript
interface ReasonTemplate {
  // ... existing fields
  isApproved: boolean;        // Prevent inappropriate content
  approverNotes?: string;     // Moderation comments
}
```

**Analytics (Opt-in)**:
```typescript
interface GameSessionEvent {
  sessionId: string;          // Anonymous session ID (hashed)
  eventType: 'draw' | 'tone_change' | 'copy';
  cardId: number;
  tone: string;
  timestamp: ISO8601DateTime;
  // NO PII, fully anonymized
}
```

**Dynamic Content** (GPT Integration):
```typescript
interface DynamicReason {
  reasonId: string;
  cardId: number;
  tone: string;
  reasonText: string;         // From GPT API
  generatedAt: ISO8601DateTime;
  fallback: ReasonTemplate;   // If API fails
}
```

**Localization**:
```typescript
interface TarotCardLocalized {
  cardId: number;
  cardName: { en: string; 'zh-TW': string; ja: string };
  description: { en: string; 'zh-TW': string; ja: string };
  // ... etc
}
```

---

## Summary

| Entity | Count | Size | Load Time | Validation |
|--------|-------|------|-----------|-----------|
| TarotCard | 5 | ~2KB | <50ms | ✓ Defined |
| ReasonTemplate | 15 | ~5KB | <50ms | ✓ Defined |
| Runtime State | ~1 | ~100B | <1ms | ✓ Via composables |

All entities defined, validated, and ready for Phase 2 implementation.

