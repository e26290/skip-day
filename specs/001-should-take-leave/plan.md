# Implementation Plan: Skip Day - 占卜請假機

**Branch**: `001-should-take-leave` | **Date**: 2025-11-14 | **Spec**: `/specs/001-should-take-leave/spec.md`
**Input**: Feature specification from `/specs/001-should-take-leave/spec.md`

## Summary

Skip Day is a humorous, client-only tarot card-based leave day decision tool. Users draw a tarot card, select a tone (平靜/中二/社恐), and copy a pre-generated excuse letter. Fully stateless, anonymous, no backend required. Built with Vue 3 + Vite + SCSS for fast dev/production performance. MVP requires 5 cards × 3 tone combinations = 15 reason templates. All technical unknowns resolved in Phase 0 research; implementation ready for Phase 2.

## Technical Context

**Language/Version**: JavaScript (ES6+), Vue 3, Vite 5.x  
**Primary Dependencies**: vue@3, vite@5, sass (SCSS support)  
**Storage**: None (fully stateless, no persistence)  
**Testing**: Vitest (unit), Playwright (E2E)  
**Target Platform**: Web browsers (Chrome, Safari, Firefox, Edge - latest 2 versions)
**Project Type**: Single-page web application (SPA), frontend-only  
**Performance Goals**: Homepage load ≤1.5s (SC-001), card flip ≤1s (SC-002), copy success ≥95% (SC-003), user flow ≤3 min (SC-004)  
**Constraints**: Zero-backend architecture, fully anonymous, no user tracking, no persistence, fast load (≤1.5s)  
**Scale/Scope**: 5 tarot cards, 3 tone variations, 4 screens (home/draw/result/excuse), ~15 Vue components total, ~200 LOC core logic

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Status**: ✅ **PASS** (No constitution constraints defined for this project; generic web SPA guidelines apply)

The project aligns with typical web application best practices:
- ✅ Single-page application with clear component structure
- ✅ Modular state management (Vue Composition API)
- ✅ Testable architecture (unit + E2E separation)
- ✅ Performance-conscious (Vite, CSS-only animations, no unnecessary libraries)
- ✅ Accessibility considerations (form inputs, color contrast per WCAG 2.1)
- ✅ Security: No user data collection, fully anonymous operation

**No complexity violations** requiring justification.

## Project Structure

### Documentation (this feature)

```text
specs/001-should-take-leave/
├── spec.md                   # Feature specification (4 user stories, 10 FRs, 6 SCs)
├── plan.md                   # This file - implementation strategy
├── research.md               # Phase 0 output: 10 technical decisions (state mgmt, animations, copy, etc.)
├── data-model.md             # Phase 1 output: Entity schemas (TarotCard, ReasonTemplate)
├── quickstart.md             # Phase 1 output: Local dev setup & workflow guide
├── contracts/
│   └── component-interface.md # Phase 1 output: Component APIs & event contracts
└── checklists/
    └── requirements.md       # Quality validation (all items ✅ pass)
```

### Source Code (repository root)

```text
skip-day/
├── src/
│   ├── main.js                   # Vue app initialization
│   ├── App.vue                   # Root component (state mgmt + routing)
│   ├── components/
│   │   ├── HomePage.vue          # Intro screen with "Start My Fate" button
│   │   ├── DrawCard.vue          # Card selection screen (3-5 cards in grid)
│   │   ├── CardFlip.vue          # Reusable card component with 3D flip animation
│   │   ├── CardResult.vue        # Card result display (name, description, buttons)
│   │   ├── ExcuseGenerator.vue   # Excuse screen with tone selector
│   │   └── Toast.vue             # Feedback toast (copy confirmation)
│   ├── composables/
│   │   ├── useGameState.js       # Game state (currentPage, selectedCard, tone, etc.)
│   │   └── useClipboard.js       # Clipboard copy utilities with fallback
│   ├── data/
│   │   ├── tarotCards.js         # 5 tarot card definitions (cardId, name, desc, image)
│   │   └── reasonTemplates.js    # 15 excuse templates (5 cards × 3 tones)
│   └── assets/
│       ├── styles/
│       │   ├── global.scss       # Design tokens, resets, utility classes
│       │   ├── tokens.scss       # Color palette, typography scale, spacing
│       │   └── animations.scss   # Shared animation definitions (flip, fade)
│       └── images/
│           └── cards/            # 5 card SVG illustrations
├── public/
│   ├── favicon.ico
│   └── manifest.json
├── tests/
│   ├── unit/
│   │   ├── composables.test.js   # Tests for useGameState, useClipboard
│   │   └── components.test.js    # Component unit tests
│   └── e2e/
│       └── happy-path.spec.js    # Playwright E2E flow (draw → tone → copy)
├── index.html                    # HTML entry point
├── vite.config.js                # Vite configuration (Vue 3 plugin, SCSS)
├── package.json                  # Dependencies & scripts
├── package-lock.json             # Dependency lock file
└── .gitignore                    # Git ignore rules

# Planning documentation (git tracked)
specs/
└── 001-should-take-leave/
    ├── spec.md, plan.md, research.md, data-model.md, quickstart.md
    ├── contracts/component-interface.md
    └── checklists/requirements.md
```

**Structure Decision**: Monolithic single-page application (Option 1). No backend, no separate frontend/backend split needed. All logic in Vue components with shared composables. Static hosting (Vercel/Netlify) sufficient.

---

## Phase 1: Design & Contracts

**Duration**: ~3 hours (completed)  
**Output**: 
- `/specs/001-should-take-leave/data-model.md` (entity schemas, validation)
- `/specs/001-should-take-leave/contracts/component-interface.md` (component APIs)
- `/specs/001-should-take-leave/quickstart.md` (dev setup guide)

### 1.1 Data Model Finalized

**TarotCard Entity** (5 total):
```
cardId (number) | cardName (string) | description (string) | illustration (path)
```

**ReasonTemplate Entity** (15 total: 5 cards × 3 tones):
```
reasonId (string: "cardId-tone") | cardId (FK) | tone (enum: 平靜|中二|社恐) | reasonText (string)
```

**Runtime State**:
```
currentPage | selectedCard | selectedTone | generatedReason | isCardFlipping | isCopied
```

All validation rules defined, referential integrity constraints specified.

### 1.2 Component APIs Defined

| Component | Props | Emits | Notes |
|-----------|-------|-------|-------|
| HomePage | — | start-game | Root entry point |
| DrawCard | cards, selectedCard, isFlipping | card-selected, flip-complete, draw-again | Shuffles cards, manages flip state |
| CardFlip | card, isFlipped, showDetails | click | Reusable 3D card with CSS transforms |
| CardResult | card | generate-excuse, draw-again | Display result + next-step buttons |
| ExcuseGenerator | card, currentTone, reasonText | tone-changed, copy-clicked, back-home, draw-again | Real-time text update on tone change |
| Toast | message, duration | — | Copy confirmation (2s auto-hide) |

**Event Flow**: HomePage → DrawCard → CardResult → ExcuseGenerator → {copy, reset}

**Data Contracts**:
- Load-time validation of cards + templates (fail-safe: show error)
- Copy failure fallback (show manual copy prompt)
- Reason not found → fallback message

### 1.3 Setup & Quickstart

**Local Dev Setup** (in `quickstart.md`):
```bash
npm install           # Install deps
npm run dev           # Start Vite (localhost:5173)
npm run test:unit    # Vitest unit tests
npm run test:e2e     # Playwright E2E tests
npm run build        # Production build
```

**Project structure documented**, key files explained, development workflow defined.

---

## Phase 2: Implementation (Planned)

**Duration**: ~30-40 hours (estimated)  
**Deliverables**: All Vue components, composables, data files, tests

### 2.1 Core Component Implementation

| Component | Est. Hours | Dependencies | Tests |
|-----------|-----------|--------------|-------|
| HomePage.vue | 2 | — | Button click → emit event |
| DrawCard.vue | 4 | CardFlip, useGameState | Shuffle, selection, flip complete |
| CardFlip.vue | 3 | — | 3D animation, CSS scoped |
| CardResult.vue | 2 | — | Button routing |
| ExcuseGenerator.vue | 4 | useClipboard, useGameState | Tone selection, real-time update, copy |
| Toast.vue | 2 | — | 2s auto-hide |
| **Composables** | 3 | — | Unit tests |
| **Data Files** | 2 | — | Validation tests |

### 2.2 Styling & Assets

| Item | Est. Hours |
|------|-----------|
| Global SCSS (tokens, animations) | 2 |
| Component-scoped styles | 4 |
| 5 card illustrations (SVG) | 8 |
| Responsive design (mobile/tablet) | 3 |

### 2.3 Testing

| Test Type | Est. Hours | Coverage |
|-----------|-----------|----------|
| Unit tests (Vitest) | 5 | composables, components, data validation |
| E2E tests (Playwright) | 4 | Full user flow, edge cases |
| Manual QA | 3 | SC-001 (load time), SC-002 (flip), SC-003 (copy), SC-004 (flow time) |

### 2.4 Deployment

| Task | Est. Hours |
|------|-----------|
| Build optimization (bundle size) | 1 |
| Deploy to Vercel/Netlify | 1 |
| Performance monitoring | 1 |

**Phase 2 Total**: ~40 hours

---

## Phase 3: Polish & Launch

**Duration**: ~10-15 hours (estimated)  
**Deliverables**: Production-ready, performance optimized, all SCs verified

### 3.1 Success Criteria Verification

- **SC-001** (≤1.5s load): Measure with Lighthouse
- **SC-002** (≤1s flip): Test with Playwright timer
- **SC-003** (≥95% copy): Test across 4 browsers
- **SC-004** (≤3 min, 85% completion): User testing session
- **SC-005** (5 cards × 3 tones): Automated validation
- **SC-006** (≥99% reason generation): All templates loaded

### 3.2 Accessibility & Edge Cases

- WCAG 2.1 Level AA compliance
- Rapid click handling (debounce card selection)
- Copy fallback (non-HTTPS, old browsers)
- Error boundaries (data load failure)
- Mobile responsiveness (tested on iOS Safari, Android Chrome)

### 3.3 Documentation & Maintenance

- README.md with feature overview
- Deployment runbook
- Troubleshooting guide
- Future enhancement ideas (Phase 2+: GPT integration, i18n, analytics)

---

## Implementation Timeline

```
Week 1 (Phase 2 - Components & Styling):
  Mon-Tue: HomePage, DrawCard, CardFlip components
  Wed: CardResult, ExcuseGenerator, Toast components
  Thu-Fri: Styling, assets, responsive design

Week 2 (Phase 2 - Testing & Polish):
  Mon-Tue: Unit tests (Vitest)
  Wed-Thu: E2E tests (Playwright)
  Fri: Manual QA, performance tuning

Week 3 (Phase 3 - Launch):
  Mon-Tue: Success criteria verification
  Wed: Accessibility review
  Thu: Deployment to production
  Fri: Post-launch monitoring
```

**Critical Path**: Phase 2 component implementation (30+ hours) is the bottleneck. Testing and polish can overlap.

---

## Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|-----------|
| Card flip animation janky on low-end devices | Poor UX | Medium | Test on Safari 15, Android Chrome 90; fallback to fade if needed |
| Copy fails in privacy-focused browsers | SC-003 miss | Low | Thorough fallback testing; manual copy prompt |
| Tone text too long (>500 chars) | Readability | Low | Validation + style constraints; test with longest example |
| HTTPS required for Clipboard API | Dev confusion | Low | Document in quickstart; provide localhost fallback |
| Vite build unexpectedly large | SC-001 miss | Low | Tree-shake unused imports; CSS minification; test build size early |

---

## Dependencies & Prerequisites

**External**:
- Node.js 18+ (dev environment)
- Vue 3 (npm package)
- Vite 5.x (npm package)
- SCSS compiler (sass npm package)

**Internal**:
- Feature spec (✅ complete)
- Technical research (✅ complete)
- Data model (✅ complete)
- Component contracts (✅ complete)

**Deliverables from Phase 0-1**:
- ✅ research.md (10 technical decisions)
- ✅ data-model.md (entity schemas + validation)
- ✅ contracts/component-interface.md (component APIs)
- ✅ quickstart.md (dev setup guide)

---

## Success Metrics & Gates

**Phase 2 Gate** (before starting Phase 3):
- [ ] All components implemented with ≥80% test coverage
- [ ] E2E tests passing (happy path + edge cases)
- [ ] No console errors in dev/prod build
- [ ] Bundle size <100KB (gzipped)

**Phase 3 Gate** (before launch):
- [ ] All 6 success criteria verified
- [ ] WCAG 2.1 Level AA compliance confirmed
- [ ] Performance: Lighthouse score ≥90 all categories
- [ ] Cross-browser testing (Chrome, Safari, Firefox, Edge)
- [ ] Zero critical bugs in final QA

**Launch Gate**:
- [ ] Live on Vercel/Netlify
- [ ] All SCs measured & documented
- [ ] Monitoring in place (error tracking, analytics)
- [ ] Runbook created for any needed updates

## Phase 0: Research & Technical Decisions

**Duration**: ~2 hours (completed)  
**Output**: `/specs/001-should-take-leave/research.md` (10 key decisions)

### Decisions Finalized

| Decision | Outcome | Rationale |
|----------|---------|-----------|
| State Management | Vue 3 Composition API (no Pinia) | Simple state (4 variables), zero external deps |
| Animations | CSS 3D + Vue Transitions | Native performance, meets SC-002 (≤1s flip) |
| Excuse Generation | Local templates + optional GPT | 99% reliable, no latency, meets SC-006 |
| Copy Mechanism | Clipboard API + fallback | ≥95% success across browsers (SC-003) |
| Styling | SCSS (global + component-scoped) | DRY tokens, encapsulation, no Tailwind bloat |
| Data Model | Embedded JSON (no backend) | Matches "fully anonymous" spec |
| Build Tool | Vite 5.x | Fastest dev/prod, Vue 3 SFC native |
| Testing | Vitest + Playwright | Fast unit + comprehensive E2E |
| Deployment | Static hosting (Vercel/Netlify) | Zero server overhead, perfectly aligned |
| Browsers | Modern only, no polyfills | Targets SC-003 browsers, fast load (SC-001) |

**No NEEDS CLARIFICATION items identified.** All technical unknowns resolved.
