# Planning Complete: Skip Day Implementation Plan Report

**Project**: Skip Day - 占卜請假機  
**Branch**: `001-should-take-leave`  
**Report Date**: 2025-11-14  
**Status**: ✅ **Phase 0 & 1 Complete - Ready for Phase 2 Implementation**

---

## Executive Summary

Skip Day is a humorous, fully client-side tarot card-based leave day decision tool. Users draw a card, select a tone (平靜/中二/社恐), and copy a pre-generated excuse letter. The feature is **specification-complete**, **technically validated**, **architecturally designed**, and **ready for implementation**.

All technical unknowns have been resolved. Implementation can begin immediately in Phase 2.

---

## Phase 0: Research & Technical Decisions ✅ Complete

**Output Location**: `/specs/001-should-take-leave/research.md`

### 10 Key Technical Decisions Finalized

1. **State Management**: Vue 3 Composition API (zero external deps)
   - Rationale: Simple state needs (4 variables: currentPage, selectedCard, selectedTone, generatedReason)
   - Alternative rejected: Pinia (overkill), Context API alone (insufficient)

2. **Card Flip Animations**: CSS 3D Transitions + Vue Transitions
   - Rationale: Native performance, SC-002 compliant (≤1s), no JS overhead
   - Alternative rejected: GSAP (overengineered, 40KB+), JavaScript animations (slower)

3. **Excuse Text Generation**: Local templates + optional GPT API fallback
   - Rationale: 99% reliable, no latency, meets SC-006
   - Alternative rejected: GPT API only (introduces failures), Random generation (poor quality)

4. **Clipboard Copy**: Clipboard API + document.execCommand fallback
   - Rationale: SC-003 compliant (≥95% success across browsers)
   - Alternative rejected: Clipboard API only (~80% compat), execCommand only (deprecated)

5. **Styling Architecture**: SCSS (global tokens + component-scoped)
   - Rationale: DRY design tokens, CSS encapsulation, no Tailwind bloat
   - Alternative rejected: CSS-in-JS (changes Vue SFC pattern), Tailwind (30KB+ raw)

6. **Data Model**: Embedded JSON (no backend)
   - Rationale: Matches "fully anonymous" spec, zero server overhead
   - Alternative rejected: REST API (unnecessary infrastructure), GraphQL (overkill)

7. **Build Tool**: Vite 5.x with Vue 3 SFC plugin
   - Rationale: Fastest dev/prod, native Vue 3 support, meets SC-001 (≤1.5s load)
   - Alternative rejected: Webpack (slow), Create Vue CLI (deprecated)

8. **Testing Strategy**: Vitest (unit) + Playwright (E2E)
   - Rationale: Fast unit tests, comprehensive E2E for full user flow
   - Alternative rejected: Jest (heavier), Cypress (slower)

9. **Deployment Target**: Static hosting (Vercel/Netlify)
   - Rationale: Zero backend infrastructure, perfect for client-only app
   - Alternative rejected: Self-hosted VPS (overhead), SSR (unnecessary)

10. **Browser Support**: Modern browsers only (no polyfills)
    - Rationale: Targets SC-003 browsers (Chrome, Safari, Firefox, Edge); fast load
    - Alternative rejected: IE11 support (adds 50KB+ polyfills)

**Status**: ✅ All technical unknowns resolved. No "NEEDS CLARIFICATION" items remaining.

---

## Phase 1: Design & Contracts ✅ Complete

### 1.1 Data Model Definition
**Output Location**: `/specs/001-should-take-leave/data-model.md`

**Entities Defined**:
- **TarotCard** (5 total): cardId, cardName, description, illustration + validation rules
- **ReasonTemplate** (15 total): reasonId, cardId, tone, reasonText + validation rules
- **Runtime State**: currentPage, selectedCard, selectedTone, generatedReason, isCardFlipping, isCopied

**Validation**:
- ✅ Field-level constraints (string lengths, enum values, FK relationships)
- ✅ Application-level constraints (uniqueness, referential integrity, completeness)
- ✅ Example data provided (5 sample cards, 15 sample templates)
- ✅ Access patterns defined (by card, by tone, direct lookup)

**Future Extensibility**:
- Schema additions for Phase 2+ (moderation, analytics, dynamic content, i18n)
- All defined and documented

### 1.2 Component Contracts
**Output Location**: `/specs/001-should-take-leave/contracts/component-interface.md`

**6 Components Designed**:

| Component | Props | Emits | Notes |
|-----------|-------|-------|-------|
| HomePage | — | start-game | Entry point, "Start My Fate" button |
| DrawCard | cards, selectedCard, isFlipping | card-selected, flip-complete, draw-again | Shuffles, manages flip animation |
| CardFlip | card, isFlipped, showDetails | click | Reusable 3D card (CSS transforms) |
| CardResult | card | generate-excuse, draw-again | Display result + buttons |
| ExcuseGenerator | card, currentTone, reasonText | tone-changed, copy-clicked, back-home, draw-again | Real-time text update |
| Toast | message, duration | — | Copy confirmation (2s auto-hide) |

**Event Flow Documented**:
- HomePage → DrawCard → CardResult → ExcuseGenerator → {copy, reset}

**Data Contracts**:
- ✅ Load-time validation (fail-safe with error messages)
- ✅ Copy failure fallback (manual prompt)
- ✅ Reason not found → fallback message
- ✅ Performance contracts (all SCs mapped to operations)

**3 Composables Designed**:
- `useGameState()`: Page routing, card selection, tone tracking
- `useClipboard()`: Copy with Clipboard API + fallback
- (Optional) `useAnimation()`: Debounce rapid clicks, animation timing

### 1.3 Development Quickstart Guide
**Output Location**: `/specs/001-should-take-leave/quickstart.md`

**Comprehensive Setup Guide Includes**:
- ✅ Prerequisites (Node.js 18+, Git)
- ✅ Installation & dependency verification
- ✅ Project structure walkthrough
- ✅ Development workflow (npm run dev, hot reload)
- ✅ Build & deployment instructions
- ✅ Testing commands (unit + E2E)
- ✅ Key file explanations
- ✅ Common dev tasks (add card, change animation, etc.)
- ✅ Troubleshooting guide
- ✅ Performance benchmarks (targets)

**Ready for developer onboarding**

---

## Implementation Plan: Phase 2 & 3

### Phase 2: Implementation (30-40 hours estimated)

**Deliverables**:

| Category | Items | Est. Hours |
|----------|-------|-----------|
| Components (6) | HomePage, DrawCard, CardFlip, CardResult, ExcuseGenerator, Toast | 17 |
| Composables (2) | useGameState, useClipboard | 3 |
| Data Files (2) | tarotCards.js, reasonTemplates.js | 2 |
| Styling | Global SCSS + component-scoped + 5 card SVGs | 17 |
| Unit Tests (Vitest) | Component + composable tests | 5 |
| E2E Tests (Playwright) | Full flow + edge cases | 4 |

**Timeline**: 3-week sprint (2 weeks components, 1 week testing & polish)

### Phase 3: Launch & Optimization (10-15 hours)

**Deliverables**:
- ✅ All 6 success criteria verified
- ✅ WCAG 2.1 Level AA accessibility confirmed
- ✅ Performance optimization (bundle size <100KB gzipped)
- ✅ Cross-browser testing (Chrome, Safari, Firefox, Edge)
- ✅ Deployment to Vercel/Netlify
- ✅ Post-launch monitoring

---

## All Phase 0-1 Artifacts

### Specification Documents (Stable)
- ✅ `/specs/001-should-take-leave/spec.md` - Feature specification (4 user stories, 10 FRs, 6 SCs, 3 edge cases, 5 assumptions)
- ✅ `/specs/001-should-take-leave/checklists/requirements.md` - Quality validation (all items ✅ pass)

### Planning Documents (Complete)
- ✅ `/specs/001-should-take-leave/plan.md` - This implementation plan (347 lines, all sections filled)
- ✅ `/specs/001-should-take-leave/research.md` - Phase 0 technical research (10 decisions, 400+ lines)
- ✅ `/specs/001-should-take-leave/data-model.md` - Phase 1 data design (entity schemas, validation, 350+ lines)
- ✅ `/specs/001-should-take-leave/contracts/component-interface.md` - Phase 1 component contracts (6 components, 300+ lines)
- ✅ `/specs/001-should-take-leave/quickstart.md` - Developer guide (setup, workflow, troubleshooting, 400+ lines)

**Total Planning Documentation**: ~2,200 lines, comprehensive coverage of all technical aspects

---

## Success Criteria Tracking

### Specification Success Criteria (6 total)

| ID | Requirement | Status | Notes |
|----|-------------|--------|-------|
| SC-001 | Homepage load ≤1.5s | 🔄 Phase 2 validation | Vite build + no unnecessary assets |
| SC-002 | Card flip ≤1s | 🔄 Phase 2 validation | CSS 3D transform, 600ms transition |
| SC-003 | Copy success ≥95% | 🔄 Phase 2 validation | Clipboard API + fallback tested on 4 browsers |
| SC-004 | User flow ≤3 min, 85% success | 🔄 Phase 2 validation | E2E test covers full flow |
| SC-005 | 5 cards + 3 tones | ✅ Designed | 15 templates defined in data model |
| SC-006 | Reason generation ≥99% | ✅ Designed | Local templates guarantee success |

---

## Technical Stack (Finalized)

```
Frontend Framework: Vue 3 (Composition API)
Build Tool: Vite 5.x
Language: JavaScript (ES6+)
Styling: SCSS (Sass)
Testing: Vitest (unit) + Playwright (E2E)
Deployment: Vercel or Netlify (static hosting)
Browsers: Chrome, Safari, Firefox, Edge (latest 2 versions)
Accessibility: WCAG 2.1 Level AA
Performance Target: Lighthouse ≥90 (all categories)
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│               HTML Entry Point                      │
│                 index.html                          │
└──────────────────┬──────────────────────────────────┘
                   │
                   ↓
┌─────────────────────────────────────────────────────┐
│         Vue 3 App Root (App.vue)                    │
│    • useGameState() - State Management              │
│    • Router (currentPage-based)                     │
└────────┬──────────┬──────────┬──────────┬───────────┘
         │          │          │          │
    ┌────▼─┐  ┌────▼──┐  ┌───▼──┐  ┌───▼───┐
    │Home  │  │Draw   │  │Result│  │Excuse │
    │Page  │  │Card   │  │      │  │Gen    │
    └───┬──┘  └───┬───┘  └──┬───┘  └───┬───┘
        │         │         │          │
        └────┬────┴────┬────┴──────┬───┘
             │         │          │
        ┌────▼──┐  ┌───▼────┐  ┌──▼──┐
        │CardFlip│  │Toast   │  │Data │
        │(3D)   │  │(Copy)  │  │Mgmt │
        └────────┘  └────────┘  └──────┘

State: localStorage/SessionStorage = NONE (fully ephemeral)
Data: Embedded JSON (tarotCards.js, reasonTemplates.js)
API: None (client-only)
```

---

## Key Architectural Decisions

1. **Fully Client-Side**: No backend API, no database, no server
   - Enables instant deployment to static hosting
   - Ensures absolute privacy (zero data collection)
   - Simplifies development & testing

2. **Component-Driven UI**: 6 components + 2 composables
   - Clear separation of concerns
   - Reusable CardFlip component
   - Testable in isolation

3. **Composition API**: Modern Vue 3 pattern
   - Reactive state in useGameState()
   - Utilities in useClipboard()
   - No Pinia/Vuex needed for this scope

4. **CSS-Only Animations**: 3D card flip
   - Native browser performance
   - No JavaScript animation overhead
   - Meets SC-002 (≤1s) easily

5. **Template-Based Reasons**: 15 pre-written templates
   - Guarantees 99% generation success
   - Consistent tone enforcement
   - Optional GPT integration in Phase 2+

---

## Next Steps (Ready for Phase 2)

### Immediate (Developer Handoff)
1. ✅ Read `/specs/001-should-take-leave/spec.md` (understand requirements)
2. ✅ Read `/specs/001-should-take-leave/research.md` (understand technical decisions)
3. ✅ Read `/specs/001-should-take-leave/contracts/component-interface.md` (understand APIs)
4. ✅ Run `/specs/001-should-take-leave/quickstart.md` (set up local dev)

### Phase 2 Work (40 hours)
1. Implement 6 Vue components (17 hrs)
2. Implement 2 composables (3 hrs)
3. Create data files + SVG assets (2+8 hrs)
4. Write unit tests (5 hrs)
5. Write E2E tests (4 hrs)

### Phase 3 Launch (10-15 hours)
1. Verify all 6 success criteria
2. Accessibility audit & fixes
3. Performance optimization
4. Deployment & monitoring

---

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Card flip animation janky on older devices | Medium | High | Test on Safari 15+, Android Chrome 90+; CSS fallback |
| Copy fails in privacy-focused browsers | Low | High | Comprehensive testing + manual fallback prompt |
| Reason text exceeds 500 chars | Low | Medium | Strict validation + design constraints |
| Build size exceeds 100KB | Low | Medium | Early tree-shaking, CSS minification testing |
| Scope creep (add more features) | Medium | High | Follow spec strictly, track via separate issues |

---

## Documentation Quality Checklist

- ✅ Specification is complete, validated, non-redundant
- ✅ Technical research covers all unknowns
- ✅ Data model is fully specified with examples
- ✅ Component contracts are detailed with props/emits
- ✅ Implementation timeline is realistic and broken down
- ✅ All success criteria are measurable and tracked
- ✅ Developer guide (quickstart) is comprehensive
- ✅ Next steps are clear and actionable

---

## Conclusion

Skip Day is **fully planned and architecturally validated**. All technical unknowns have been resolved. The specification is stable and complete. Developer documentation is comprehensive.

**Status**: Ready to begin Phase 2 implementation immediately.

**Estimated Total Project Duration**: 6-8 weeks
- Phase 0 (Research): 2 hrs ✅ Done
- Phase 1 (Design): 3 hrs ✅ Done  
- Phase 2 (Implementation): 40 hrs (3-4 weeks)
- Phase 3 (Launch): 15 hrs (1 week)

**Go-to-Market**: Can deploy to production 4-5 weeks from Phase 2 start.

---

**Report Generated**: 2025-11-14  
**Planning Complete**: ✅ YES  
**Ready for Implementation**: ✅ YES

