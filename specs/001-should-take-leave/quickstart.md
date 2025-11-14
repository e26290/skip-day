# Quickstart Guide: Local Development Setup

**Feature**: Skip Day - 占卜請假機  
**Branch**: `001-should-take-leave`  
**Last Updated**: 2025-11-14  

---

## Prerequisites

- **Node.js**: v18+ (includes npm)
- **Git**: For cloning and branch management
- **Code Editor**: VS Code (recommended) or any editor
- **macOS/Linux/Windows**: All platforms supported

**Verify Installation**:
```bash
node --version    # Should output v18.0.0 or higher
npm --version     # Should output 8.0.0 or higher
git --version     # Should output 2.0.0 or higher
```

---

## 1. Clone & Setup Repository

```bash
# Clone the repository (if not already done)
cd /path/to/your/projects
git clone <repository-url> skip-day
cd skip-day

# Switch to feature branch
git checkout 001-should-take-leave

# Or create it if starting fresh:
# git checkout -b 001-should-take-leave
```

---

## 2. Install Dependencies

```bash
# Install npm packages defined in package.json
npm install

# Verify installation (check node_modules/ created)
ls -la node_modules/ | head -10

# Expected packages:
# - vue (^3.x)
# - vite (^5.x)
# - sass (for SCSS support)
# - @vitejs/plugin-vue (Vite's Vue plugin)
```

**What's Being Installed**:
- `vue`: Vue.js 3 framework for reactive UI components
- `vite`: Lightning-fast build tool and dev server
- `sass`: SCSS preprocessor for styling
- `@vitejs/plugin-vue`: Vite plugin for .vue Single File Components
- **Optional (dev)**: `vitest` (unit testing), `@playwright/test` (E2E testing), ESLint, Prettier

---

## 3. Project Structure Overview

```
skip-day/
├── src/
│   ├── main.js                 # App entry point (initializes Vue)
│   ├── App.vue                 # Root component (state & routing)
│   ├── components/
│   │   ├── HomePage.vue        # Intro screen
│   │   ├── DrawCard.vue        # Card selection screen
│   │   ├── CardFlip.vue        # Reusable card component (with 3D flip)
│   │   ├── ExcuseGenerator.vue # Excuse screen with tone selector
│   │   └── Toast.vue           # Feedback toast (copy confirmation)
│   ├── composables/
│   │   ├── useGameState.js     # Game state (currentPage, selectedCard, etc.)
│   │   └── useClipboard.js     # Clipboard copy utilities
│   ├── data/
│   │   ├── tarotCards.js       # 5 tarot cards (cardId, name, description, image)
│   │   └── reasonTemplates.js  # 15 excuse templates (5 cards × 3 tones)
│   └── assets/
│       ├── styles/
│       │   ├── global.scss     # Global styles, design tokens
│       │   ├── tokens.scss     # Colors, typography, spacing
│       │   └── animations.scss # Shared animation definitions
│       └── images/
│           └── cards/          # Card SVG illustrations (5 cards)
├── public/
│   └── [static assets]         # Favicon, manifest.json, etc.
├── tests/
│   ├── unit/                   # Vitest unit tests
│   │   ├── composables.test.js
│   │   └── components.test.js
│   └── e2e/                    # Playwright E2E tests
│       └── happy-path.spec.js
├── index.html                  # HTML entry point
├── vite.config.js              # Vite configuration
├── package.json                # Project metadata & dependencies
└── .gitignore                  # Git ignore rules

# Planning & Specification (not part of build)
specs/
└── 001-should-take-leave/
    ├── spec.md                 # Feature specification
    ├── plan.md                 # This implementation plan
    ├── research.md             # Phase 0 technical decisions
    ├── data-model.md           # Entity schemas & validation
    ├── contracts/
    │   └── component-interface.md  # Component APIs
    └── quickstart.md           # This file
```

---

## 4. Start Development Server

```bash
# Start Vite dev server with hot reload
npm run dev

# Output should show:
# VITE v5.x.x ready in XXX ms
# 
# ➜  Local:   http://localhost:5173/
# ➜  Press h for help
```

**Access in Browser**:
- Open `http://localhost:5173/` in your browser
- You should see the Skip Day homepage with "Tap the cards. Escape reality."

**Hot Module Replacement (HMR)**:
- Edit any `.vue`, `.js`, or `.scss` file
- Changes automatically reflect in browser (no page reload needed)

**Stop Server**:
- Press `Ctrl+C` (or `Cmd+C` on Mac) in terminal

---

## 5. Build for Production

```bash
# Create optimized production build
npm run build

# Output directory: dist/
# File sizes should be minimal:
# - dist/index.html (~2KB)
# - dist/assets/main-*.js (~50KB gzipped)
# - dist/assets/style-*.css (~10KB gzipped)

# Verify build output
ls -lh dist/

# Preview production build locally (optional)
npm run preview
# Then open http://localhost:4173/
```

---

## 6. Run Tests

### Unit Tests (Vitest)

```bash
# Run all unit tests
npm run test:unit

# Run in watch mode (re-run on file change)
npm run test:unit -- --watch

# Expected test files:
# - tests/unit/composables.test.js
# - tests/unit/components.test.js

# Sample test output:
# ✓ useGameState (3 tests)
#   ✓ initializes with home page
#   ✓ resets game state correctly
#   ✓ updates selected card
# 
# ✓ ExcuseGenerator (2 tests)
#   ✓ displays reason text
#   ✓ updates on tone change
# 
# Test Files  2 passed (2)
# Tests      5 passed (5)
```

### E2E Tests (Playwright)

```bash
# Run end-to-end tests (full browser automation)
npm run test:e2e

# Run in headed mode (see browser window)
npm run test:e2e -- --headed

# Test files:
# - tests/e2e/happy-path.spec.js (full user flow)

# Sample test output:
# happy-path.spec.js:
#   ✓ Full flow: draw card → generate reason → copy (2.3s)
#   ✓ Handles rapid card clicks gracefully (1.8s)
#   ✓ Copy success feedback appears (1.5s)
# 
# 3 passed (5.6s)
```

---

## 7. Key Files to Understand First

### 1. `src/main.js` - App Initialization
```javascript
// Imports Vue, mounts App component to #app div
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

### 2. `src/App.vue` - Root Component & State
```vue
<template>
  <div id="app">
    <component :is="currentPageComponent" />
  </div>
</template>

<script setup>
import { useGameState } from '@/composables/useGameState'
import { computed } from 'vue'

const { currentPage } = useGameState()

const currentPageComponent = computed(() => {
  const components = {
    home: HomePage,
    draw: DrawCard,
    result: CardResult,
    excuse: ExcuseGenerator,
  }
  return components[currentPage.value]
})
</script>
```

### 3. `src/data/tarotCards.js` - Card Definitions
```javascript
export const tarotCards = [
  {
    cardId: 1,
    cardName: '懶骨頭之星',
    description: '給我請假，現在。',
    illustration: '/images/cards/lazy-star.svg',
  },
  // ... 4 more cards
]
```

### 4. `src/data/reasonTemplates.js` - Excuse Text
```javascript
export const reasonTemplates = {
  '1-平靜': {
    cardId: 1,
    tone: '平靜',
    reasonText: '我今天身體有些不適，需要休息調整。感謝理解。',
  },
  // ... 14 more (5 cards × 3 tones)
}
```

### 5. `src/components/CardFlip.vue` - 3D Card Animation
```vue
<style scoped>
.card-container {
  perspective: 1000px;
  transition: transform 0.6s ease-in-out;
}
.card-container.flipped {
  transform: rotateY(180deg);
}
</style>
```

---

## 8. Development Workflow

### Making Changes

1. **Create/switch to branch** (if not already):
   ```bash
   git checkout 001-should-take-leave
   ```

2. **Edit a component** (e.g., HomePage.vue):
   ```bash
   vim src/components/HomePage.vue
   ```
   - Save file (Cmd+S)
   - Browser auto-refreshes (HMR)

3. **Check your changes**:
   ```bash
   npm run dev  # Dev server still running
   # View http://localhost:5173/ in browser
   ```

4. **Write tests** (alongside your code):
   ```bash
   vim tests/unit/HomePage.test.js
   npm run test:unit -- --watch
   ```

5. **Commit your work**:
   ```bash
   git add src/
   git add tests/
   git commit -m "feat: implement HomePage component"
   ```

6. **Push to remote** (when ready for review):
   ```bash
   git push origin 001-should-take-leave
   ```

### Common Development Tasks

**Add a new card**:
1. Create SVG illustration → `src/assets/images/cards/new-card.svg`
2. Add entry to `src/data/tarotCards.js`
3. Add 3 reason templates to `src/data/reasonTemplates.js` (one per tone)
4. Update validation in `tests/unit/data.test.js`

**Change card flip animation speed**:
1. Edit `src/components/CardFlip.vue`
2. Adjust `transition: 0.6s` to desired duration (meets SC-002 if ≤1s)
3. Browser auto-updates

**Modify tone selection UI**:
1. Edit `src/components/ExcuseGenerator.vue`
2. Change tone button styles in `<style scoped>`
3. Test in browser, verify copy still works

**Fix clipboard copy**:
1. Edit `src/composables/useClipboard.js`
2. Run `npm run test:e2e -- --headed` to see copy action in browser
3. Adjust fallback logic if needed

---

## 9. Build & Deployment Checklist

Before deploying to production:

- [ ] All tests passing: `npm run test:unit && npm run test:e2e`
- [ ] No console errors: Check browser DevTools
- [ ] SC-001 met: Homepage loads in ≤1.5s (check Network tab)
- [ ] SC-002 met: Card flip in ≤1s (verify with test)
- [ ] SC-003 met: Copy works in Chrome, Safari, Firefox, Edge
- [ ] SC-004 met: Full flow completable in ≤3 min by ~85% of testers
- [ ] Build succeeds: `npm run build` with no errors
- [ ] dist/ folder is small (<100KB total)

**Deploy to Vercel**:
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy (auto-detects Vite + Vue 3)
vercel

# Select project, confirm settings, get live URL
# Live site: https://skip-day.vercel.app
```

**Deploy to Netlify**:
```bash
# Via Git (recommended)
# Connect GitHub repo to Netlify Dashboard
# Build command: npm run build
# Publish directory: dist

# Or via Netlify CLI:
npm install -g netlify-cli
netlify deploy --prod --dir=dist
```

---

## 10. Troubleshooting

### Port 5173 Already in Use
```bash
# Use different port
npm run dev -- --port 3000
# Then visit http://localhost:3000/
```

### Node Modules Issues
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
npm run dev
```

### Card Images Not Loading
```bash
# Check paths in src/data/tarotCards.js
# Ensure images exist: src/assets/images/cards/*.svg
# Vite requires relative paths starting with /
# Example: '/images/cards/lazy-star.svg'
```

### Tests Fail
```bash
# Check if dev dependencies installed
npm install --save-dev vitest @vitejs/plugin-vue
npm run test:unit -- --reporter=verbose
```

### Hot Reload Not Working
```bash
# Check Vite dev server is running
# Press h in terminal for help
# Or restart with: npm run dev

# Check firewall/port blocking
lsof -i :5173  # See if process exists
```

### Build Too Large
```bash
# Analyze bundle size
npm run build
# Check dist/ folder
ls -lh dist/assets/

# If JS >100KB:
# - Check unused imports/dependencies
# - Enable compression in vite.config.js
```

---

## 11. Git Workflow Reminders

### Commit Message Format
```
feat: add HomePage component
fix: resolve copy button failure in Safari
refactor: extract useGameState composable
test: add E2E test for full user flow
docs: update quickstart guide
```

### Push to Remote
```bash
# View current branch
git branch

# Push with tracking (first time)
git push -u origin 001-should-take-leave

# Push (subsequent times)
git push

# View remote status
git log --oneline -5
```

### Sync with Main (If Needed)
```bash
# Fetch latest from main
git fetch origin main

# Rebase your branch on main (if main has new commits)
git rebase origin/main

# Or merge (if prefer merge commits)
git merge origin/main

# Resolve conflicts if any, then push
git push
```

---

## 12. Performance Benchmarks (Target)

Run these commands periodically:

```bash
# Build performance
time npm run build

# Bundle analysis (install first: npm install --save-dev rollup-plugin-visualizer)
npm run build -- --analyze

# Lighthouse score (using Google PageSpeed Insights)
# Target: Performance ≥90, Accessibility ≥90, Best Practices ≥90

# Test execution time
npm run test:unit -- --reporter=verbose
# Target: <500ms total

npm run test:e2e
# Target: <10s total
```

---

## Next Steps

1. ✅ **Setup complete**: You can now run `npm run dev` and see the app
2. **Phase 2**: Implement components from `src/components/`
3. **Phase 3**: Create tests in `tests/`
4. **Phase 4**: Deploy to production

**Questions?** Refer to:
- Spec: `/specs/001-should-take-leave/spec.md`
- Data Model: `/specs/001-should-take-leave/data-model.md`
- Component Contracts: `/specs/001-should-take-leave/contracts/component-interface.md`
- Tech Decisions: `/specs/001-should-take-leave/research.md`

---

**Happy coding! 🔮**

