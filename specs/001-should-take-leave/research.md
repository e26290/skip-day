# Phase 0 Research: Skip Day Technical Unknowns & Decisions

**Feature**: Skip Day - 占卜請假機  
**Branch**: `001-should-take-leave`  
**Date**: 2025-11-14  
**Tech Stack**: Vite + Vue 3 + JavaScript + SCSS  

## Technical Overview

This document resolves all technical unknowns identified during planning. Each decision includes rationale and rejected alternatives.

---

## 1. State Management Approach

### Decision: Vue 3 Composition API + Reactive Refs (No external state library)

**Rationale**:
- Skip Day is a stateless, client-only application with simple state needs:
  - Currently selected card (TarotCard)
  - Currently selected tone (平靜 | 中二 | 社恐)
  - Current page (home | draw | result | excuse)
  - Generated reason text (ephemeral)
- No backend synchronization, no persistence, no complex async workflows
- Vue 3's Composition API handles reactive state elegantly for this scope
- Zero external dependencies = faster load, smaller bundle

**Alternatives Rejected**:
- **Pinia (Vuex successor)**: Overkill for 4 state variables; adds complexity and bundle size
- **Context API alone**: Insufficient for passing data across deeply nested components
- **Props drilling**: Viable but verbose; reactive composition API cleaner for modal-like page transitions

**Implementation Pattern**:
```javascript
// src/composables/useGameState.js
import { ref, computed } from 'vue'

export function useGameState() {
  const currentPage = ref('home') // 'home' | 'draw' | 'result' | 'excuse'
  const selectedCard = ref(null) // TarotCard object
  const selectedTone = ref('平靜') // tone selection
  const generatedReason = ref('') // current excuse text
  
  const resetGame = () => {
    currentPage.value = 'home'
    selectedCard.value = null
    selectedTone.value = '平靜'
    generatedReason.value = ''
  }
  
  return { currentPage, selectedCard, selectedTone, generatedReason, resetGame }
}
```

---

## 2. Card Flip Animation & Transitions

### Decision: CSS Transitions + Vue Transitions for card flip; CSS Keyframes for page transitions

**Rationale**:
- Card flip requires 3D perspective effect (transform origin, rotate Y-axis)
- CSS 3D transforms native in all modern browsers; no library needed
- Vue's `<Transition>` component perfect for page-level fade/slide effects
- Success Criteria SC-002 requires ≤1s flip animation (achievable with CSS)
- SC-001 requires ≤1.5s homepage load (CSS-only avoids JS animation overhead)

**Alternatives Rejected**:
- **GSAP**: Powerful but overengineered for card flip; adds 40KB+ to bundle
- **Framer Motion**: React-specific, incompatible with Vue 3
- **JavaScript-driven animations (animate())**: Lower performance, requires JS execution on every frame

**Implementation Details**:
```scss
// Card flip using CSS 3D
.card-container {
  perspective: 1000px;
  width: 100px;
  height: 150px;
  position: relative;
}

.card {
  width: 100%;
  height: 100%;
  transition: transform 0.6s ease-in-out;
  transform-style: preserve-3d;
  
  &.flipped {
    transform: rotateY(180deg);
  }
  
  .card-front, .card-back {
    position: absolute;
    backface-visibility: hidden;
  }
  
  .card-back {
    transform: rotateY(180deg);
  }
}
```

```vue
<!-- Page transitions -->
<Transition name="fade" mode="out-in">
  <component :is="currentPageComponent" :key="currentPage" />
</Transition>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

---

## 3. Excuse Text Generation

### Decision: Hybrid approach - Pre-written templates + optional GPT API fallback

**Rationale**:
- Assumption states "理由可由 GPT API 或手寫模板提供"
- Phase 1 (MVP): Use pre-written templates only (reliable, fast, no API keys needed)
- Phase 2 (Enhancement): Add optional GPT API integration for dynamic variety
- Spec requires 5 cards × 3 tones = 15 reasons minimum (easily manageable with templates)
- Success Criteria SC-006 requires ≥99% generation success (local templates guarantee this)

**Template Structure**:
```javascript
const reasonTemplates = {
  '懶骨頭之星': {
    '平靜': '我今天身體有些不適，需要休息調整。謝謝。',
    '中二': '懶骨頭之星降臨，吾已被選中，需要靜修一日。',
    '社恐': '...不好意思，今天狀態不太好，能否...請假？'
  },
  // ... more cards
}
```

**Alternatives Rejected**:
- **GPT API only**: Adds latency (API calls), requires API key management, introduces failure points
- **Hardcoded single tone**: Fails SC-005 (3 tones per card)
- **Random text generation (Markov chains)**: Unreliable output quality, harder to maintain tone consistency

---

## 4. Clipboard Copy Implementation

### Decision: Clipboard API with fallback to document.execCommand()

**Rationale**:
- Modern solution: `navigator.clipboard.writeText()` works in Chrome, Safari, Firefox, Edge (all HTTPS)
- Fallback for older browsers: `document.execCommand('copy')` with DOM selection
- SC-003 requires ≥95% success across four browsers
- Clipboard API + fallback achieves this reliably

**Implementation**:
```javascript
// src/composables/useClipboard.js
export function useClipboard() {
  const copyToClipboard = async (text) => {
    try {
      if (navigator.clipboard && window.isSecureContext) {
        await navigator.clipboard.writeText(text)
        return true
      } else {
        // Fallback for non-secure contexts or older browsers
        const textarea = document.createElement('textarea')
        textarea.value = text
        document.body.appendChild(textarea)
        textarea.select()
        const success = document.execCommand('copy')
        document.body.removeChild(textarea)
        return success
      }
    } catch (err) {
      console.error('Copy failed:', err)
      return false
    }
  }
  
  return { copyToClipboard }
}
```

**Alternatives Rejected**:
- **Clipboard API only**: Fails in non-HTTPS contexts and older browsers (↓ SC-003 to ~80%)
- **execCommand() only**: Deprecated, unreliable in newer browser versions
- **Third-party library (clipboard.js)**: 3KB+ additional bundle; native solution sufficient

---

## 5. Styling Architecture: Global vs. Component-Scoped SCSS

### Decision: Hybrid - Global SCSS for typography, colors, spacing; Component-scoped SCSS for layout

**Rationale**:
- Global file defines design tokens (colors, typography, breakpoints) → reusable across components
- Component-scoped styles prevent conflicts and enable CSS-in-JS-like encapsulation
- Vue's `<style scoped>` automatically applies unique class prefixes
- SCSS variables and mixins centralized for DRY principle

**File Structure**:
```
src/assets/styles/
├── global.scss          # Design tokens, resets, utility classes
├── tokens.scss          # Color palette, typography scale, spacing
└── mixins.scss          # Responsive breakpoints, animations

src/components/
├── HomePage.vue
│   ├── template
│   ├── script
│   └── style (scoped)   # Component-specific layout & animations
└── CardFlip.vue
    └── style (scoped)   # Card flip-specific transforms
```

**Alternatives Rejected**:
- **Component-scoped only**: Duplicates token definitions across files
- **Global CSS only**: Risk of style conflicts in large applications; harder to maintain
- **CSS-in-JS (Tailwind, styled-components)**: Changes from Vue 3 SFC pattern; Tailwind adds 30KB+ raw build

---

## 6. Data Model & API Design

### Decision: Local data JSON + Vue props/emits (no API layer needed for MVP)

**Rationale**:
- Spec: "無登入、無資料保存" (fully anonymous, no backend)
- All data (cards, reason templates) embedded in frontend
- No server communication required for core flow
- Deployment: Static site (Netlify, Vercel, GitHub Pages) sufficient

**MVP Data Structure**:
```javascript
// src/data/tarotCards.js
export const tarotCards = [
  {
    cardId: 1,
    cardName: '懶骨頭之星',
    description: '給我請假，現在。',
    illustration: '/images/card-1.png',
  },
  // ... 4 more cards (total 5)
]

// src/data/reasonTemplates.js
export const reasonTemplates = {
  1: { // cardId 1
    '平靜': '我今天身體有些不適，需要休息調整。謝謝。',
    '中二': '懶骨頭之星降臨，吾已被選中，需要靜修一日。',
    '社恐': '...不好意思，今天狀態不太好，能否...請假？'
  },
  // ... more cards
}
```

**Alternatives Rejected**:
- **REST API backend**: Adds server infrastructure (Node.js, database) for zero data persistence
- **GraphQL endpoint**: Overkill for read-only, static data
- **External card/reason service**: Introduces latency, API reliability dependency

---

## 7. Asset Management & Deployment

### Decision: SVG cards + static image assets; Vite asset optimization; Deploy to Vercel/Netlify

**Rationale**:
- SVG cards scale perfectly to any device (crisp, small file size)
- Vite automatically optimizes images and code-splits CSS/JS
- Static site hosting (no server needed) aligns with "fully anonymous" requirement
- SC-001 requires ≤1.5s homepage load: Vite's aggressive tree-shaking achieves this

**Deployment Path**:
1. Local dev: `npm run dev` (Vite dev server with HMR)
2. Production: `npm run build` (outputs `dist/` optimized bundle)
3. Host: `dist/` directory on Vercel or Netlify (automatic from Git)

**Alternatives Rejected**:
- **Raster images (PNG/JPG)**: Larger filesize, quality issues on retina displays
- **Server-side rendering (SSR)**: Unnecessary for client-only app; adds complexity
- **Self-hosted (VPS)**: Overhead for static content; CDN-hosted faster/cheaper

---

## 8. Testing Strategy

### Decision: Vitest (unit tests) + Playwright (e2e tests); Skip integration tests for MVP

**Rationale**:
- Vitest: Fast unit testing for Vue components, composables, utilities (no backend integration needed)
- Playwright: E2E testing covers full user flows (homepage → draw → excuse → copy)
- No integration tests needed: No backend, no database, no inter-service communication
- MVP focus: Validate Happy Path (draw card → generate reason → copy)
- Edge cases (FR-010 animation feedback, SC-003 copy success) covered by E2E

**Example Test Structure**:
```javascript
// tests/unit/useGameState.test.js - Vitest
import { describe, it, expect } from 'vitest'
import { useGameState } from '@/composables/useGameState'

describe('useGameState', () => {
  it('initializes with home page', () => {
    const { currentPage } = useGameState()
    expect(currentPage.value).toBe('home')
  })
  
  it('resets game state', () => {
    const { selectedCard, resetGame } = useGameState()
    selectedCard.value = { cardId: 1 }
    resetGame()
    expect(selectedCard.value).toBeNull()
  })
})

// tests/e2e/happy-path.spec.js - Playwright
import { test, expect } from '@playwright/test'

test('Full flow: draw card → generate reason → copy', async ({ page }) => {
  await page.goto('http://localhost:5173')
  await page.click('button:has-text("Start My Fate")')
  
  // Select first card
  await page.click('[data-testid="card-0"]')
  
  // Verify card flip animation
  await expect(page.locator('.card-flipped')).toBeVisible({ timeout: 1000 })
  
  // Generate excuse
  await page.click('button:has-text("Generate Excuse")')
  
  // Copy
  await page.click('button:has-text("Copy")')
  await expect(page.locator('text=已複製')).toBeVisible({ timeout: 2000 })
})
```

**Alternatives Rejected**:
- **Jest**: Heavier than Vitest; Vitest optimized for Vite projects
- **Cypress**: Heavier than Playwright; Playwright better for modern browsers
- **Only unit tests**: Fails to validate full user flow (E2E essential for SC-004)

---

## 9. Development & Build Configuration

### Decision: Vite 5.x with Vue 3 SFC plugin; SCSS preprocessor; Git-based workflow

**Rationale**:
- Vite 5.x: Latest, optimized for Vue 3
- SFC (Single File Component): `.vue` format with `<template>`, `<script>`, `<style scoped>`
- SCSS built-in via `sass` package (auto-detected by Vite)
- Git workflow: Feature branches per spec, commit messages tied to requirements

**vite.config.js Template**:
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    strictPort: false,
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vue': ['vue'],
        },
      },
    },
  },
})
```

**Alternatives Rejected**:
- **Webpack**: Slower dev server, more config boilerplate
- **Next.js/Nuxt**: Unnecessary for client-only, no-backend app
- **Create Vue / Vue CLI**: Deprecated in favor of Vite

---

## 10. Browser Support & Polyfills

### Decision: Modern browsers only (Chrome, Safari, Firefox, Edge latest 2 versions); No polyfills

**Rationale**:
- Success Criteria SC-003 targets "Chrome、Safari、Firefox、Edge"
- All targeted browsers support:
  - CSS 3D Transforms (for card flip)
  - Clipboard API (with fallback)
  - Fetch API (if future API layer added)
  - CSS Grid & Flexbox (for layout)
- No IE11 or old mobile support required (aligns with "modern web app" positioning)
- Zero polyfills = faster load time (supports SC-001)

**Alternatives Rejected**:
- **@babel/preset-env + core-js**: Adds 50KB+; unnecessary for stated target
- **Legacy browser support**: Increases complexity, contradicts performance goals

---

## Summary: Tech Stack Confirmations

| Area | Decision | Rationale |
|------|----------|-----------|
| State | Vue 3 Composition API | Simple state needs, zero external deps |
| Animations | CSS Transitions + Vue Transitions | Native, performant, SC-002 compliant |
| Text Gen | Local templates + optional GPT fallback | 99% reliable, no latency |
| Copy | Clipboard API + fallback | SC-003 compliant (≥95% across browsers) |
| Styles | SCSS (global + scoped) | DRY design tokens, scoped encapsulation |
| Data | Embedded JSON (no backend) | Matches "fully anonymous" spec |
| Build | Vite 5.x + Vue 3 plugin | Fastest dev/prod, SFC-native |
| Testing | Vitest + Playwright | Fast unit + comprehensive E2E |
| Deploy | Static hosting (Vercel/Netlify) | Aligns with zero-backend architecture |
| Browsers | Modern only (no polyfills) | Matches SC-003 targets, fast load |

---

## Next Steps (Phase 1)

1. **data-model.md**: Detailed schema for TarotCard, ReasonTemplate with validation rules
2. **contracts/**: API design document (even though local, defines component interfaces)
3. **quickstart.md**: Local dev setup, running Vite, running tests
4. Update `plan.md` with complete Phase 0-1 artifacts

