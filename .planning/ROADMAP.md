# Roadmap: QA Agent v0.3 (Visual & UX Design Verification)

## Overview

This milestone bridges the gap between "does it work?" and "does it look and feel right?" — adding visual/UX design verification capabilities and orchestration guidance. The milestone starts with spec format extensions (infrastructure for new verification modes), then adds five clusters of new methodology in parallel: design verification, UX state verification, layout & content integrity, performance & responsive, and usage guidance.

**Execution model:** Phase 8 first (spec format foundation). Then Phases 9–13 are fully independent and can run in parallel across separate terminal panes.

## Phases

- [ ] **Phase 8: Spec Format Extensions** - Add Visual Focus tiering, Design Reference, and Browsers sections to spec format
- [ ] **Phase 9: Design Verification** - Design reference comparison, typography, images/media, color/theme, cross-page consistency
- [x] **Phase 10: UX State Verification** - Empty/loading/error states, interaction state matrix, interactive feedback, animation quality (completed 2026-04-20)
- [x] **Phase 11: Layout & Content Integrity** - Spacing/alignment, cross-page consistency, realistic-data overflow, placeholder detection (completed 2026-04-20)
- [ ] **Phase 12: Performance & Responsive** - Core Web Vitals, cross-browser rendering, breakpoint sweep, UX anti-pattern detection
- [ ] **Phase 13: Usage Guidance** - ORCHESTRATION.md reference doc and /qa:guide skill

## Phase Details

### Phase 8: Spec Format Extensions

**Goal**: Spec format supports visual tiering, design reference images, and cross-browser engine selection — enabling all downstream v0.3 methodology to be spec-driven

**Depends on**: Nothing (first phase of this milestone)

**Requirements**: INFRA-01, INFRA-02, INFRA-03

**Success Criteria** (what must be TRUE):
  1. A spec with `## Visual Focus` section triggers Tier 2 visual verification (parallel to `## Accessibility Focus`)
  2. A spec with `## Design Reference` section provides mockup image paths that the agent uses during comparison
  3. A spec with `## Browsers` section controls which Playwright engine(s) run (Chromium, WebKit, Firefox)
  4. SPEC-FORMAT.md documents all three new sections with examples
  5. Existing specs without these sections continue to work unchanged (backward compatible)

**Plans:** 1 plan

Plans:
- [x] 08-01-PLAN.md — Add Visual Focus, Design Reference, and Browsers sections to SPEC-FORMAT.md, qa-tester.md, skills/run/SKILL.md, and create sample spec

### Phase 9: Design Verification

**Goal**: Agent can verify visual design fidelity — comparing live pages against references, checking typography, images/media quality, color token consistency, and cross-page design consistency

**Depends on**: Phase 8 (Visual Focus tiering and Design Reference spec sections)

**Requirements**: DESIGN-01, DESIGN-02, DESIGN-03, DESIGN-04, DESIGN-05

**Parallel with**: Phases 10, 11, 12, 13

**Success Criteria** (what must be TRUE):
  1. Agent compares a live page against a provided mockup image and reports visual deviations with root cause diagnosis (layout shift, wrong color, missing element)
  2. Agent verifies fonts loaded correctly, are consistent across pages, and that text does not overflow its container
  3. Agent detects broken images, aspect ratio distortion, and images that appear upscaled or pixelated
  4. Agent checks hover/focus color states and dark mode switching consistency
  5. Agent infers and enforces cross-page consistency when no formal design system is declared

**Plans:** 2 plans

Plans:
- [x] 09-01-PLAN.md — Add Design Verification section header, Mockup Comparison (DESIGN-01), Typography (DESIGN-02), and Image & Media Quality (DESIGN-03) to qa-tester.md
- [x] 09-02-PLAN.md — Add Color & Theme Consistency (DESIGN-04), Cross-Page Consistency (DESIGN-05), and Confidence Table to qa-tester.md

### Phase 10: UX State Verification

**Goal**: Agent systematically verifies all UI states — empty, loading, error, first-run, and the full interactive state matrix — plus hover/scroll feedback and animation quality

**Depends on**: Phase 8 (Visual Focus tiering)

**Requirements**: STATE-01, STATE-02, STATE-03, STATE-04

**Parallel with**: Phases 9, 11, 12, 13

**Success Criteria** (what must be TRUE):
  1. Agent verifies empty state, loading state, error illustration state, and first-run/onboarding state exist and render correctly for each relevant component
  2. Agent sweeps each interactive component through its full state matrix: default, hover, focus, active, disabled, loading, error, success
  3. Agent checks cursor correctness on interactive elements, toast/notification timing and dismissal, and sticky header behavior during scroll
  4. Agent verifies transitions are smooth, consistent across similar element types, and loading animations are present where expected

**Plans:** 2/2 plans complete

Plans:
- [x] 10-01-PLAN.md — Add UX State Verification section header, State Existence (STATE-01), and Interaction State Matrix (STATE-02) to qa-tester.md
- [x] 10-02-PLAN.md — Add Interactive Feedback Quality (STATE-03), Animation & Transition Quality (STATE-04), Reporting table, and update Visual Focus stub note

### Phase 11: Layout & Content Integrity

**Goal**: Agent verifies layout precision, cross-page structural consistency, behavior with realistic-length content, and absence of placeholder/draft content

**Depends on**: Phase 8 (Visual Focus tiering)

**Requirements**: LAYOUT-01, LAYOUT-02, LAYOUT-03, LAYOUT-04

**Parallel with**: Phases 9, 10, 12, 13

**Success Criteria** (what must be TRUE):
  1. Agent checks spacing consistency, element alignment, and flags content clipping or overflow
  2. Agent verifies header/footer, navigation patterns, and key components are structurally consistent across pages
  3. Agent tests with realistic-length content (long names, large numbers, localization strings) and reports where layout breaks
  4. Agent detects Lorem ipsum, TODO/FIXME strings, filler data, inconsistent casing, and vague CTAs in visible content

**Plans:** 2/2 plans complete

Plans:
- [x] 11-01-PLAN.md — Add Layout & Content Integrity section header, Spacing & Alignment Checks (LAYOUT-01), and Cross-Page Structural Consistency (LAYOUT-02) to qa-tester.md
- [x] 11-02-PLAN.md — Add Realistic-Data Overflow Testing (LAYOUT-03), Placeholder & Draft Content Detection (LAYOUT-04), Reporting table, and update Visual Focus stub note

### Phase 12: Performance & Responsive

**Goal**: Agent measures Core Web Vitals, identifies cross-browser rendering differences, catches breakpoint overflow bugs, and flags UX anti-patterns

**Depends on**: Phase 8 (Browsers spec section)

**Requirements**: PERF-01, PERF-02, PERF-03, PERF-04

**Parallel with**: Phases 9, 10, 11, 13

**Success Criteria** (what must be TRUE):
  1. Agent measures CLS, LCP, and INP and identifies which elements or interactions cause poor scores
  2. Agent runs visual checks in Chromium and WebKit and reports rendering differences between engines
  3. Agent sweeps viewport widths and catches horizontal scroll, content overlap, and off-by-one breakpoint transitions
  4. Agent flags UX anti-patterns: modal without close button, cookie banner blocking content, pre-checked opt-ins, infinite scroll without position recovery

**Plans**: TBD

### Phase 13: Usage Guidance

**Goal**: Orchestrating agents and developers have clear documentation and a context-aware skill for understanding how to use the QA plugin

**Depends on**: Nothing (fully independent — can run in parallel with any phase including Phase 8)

**Requirements**: INFRA-04, INFRA-05

**Parallel with**: Phases 8, 9, 10, 11, 12

**Success Criteria** (what must be TRUE):
  1. `docs/ORCHESTRATION.md` exists and teaches an orchestrating agent how to invoke QA skills, interpret results, and integrate with GSD phase transitions
  2. `/qa:guide` skill reads the current project state and returns context-aware guidance on which QA commands to use and when
  3. A developer new to the plugin can read ORCHESTRATION.md and know exactly how to use every skill without reading source files

**Plans**: TBD

## Progress

Execution order: Phase 8 first. Then Phases 9 / 10 / 11 / 12 / 13 in parallel (all independent after Phase 8).

Phase 13 has zero dependencies — it can run during Phase 8 or any time.

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 8. Spec Format Extensions | 0/1 | Not started | - |
| 9. Design Verification | 0/2 | Not started | - |
| 10. UX State Verification | 2/2 | Complete    | 2026-04-20 |
| 11. Layout & Content Integrity | 2/2 | Complete   | 2026-04-20 |
| 12. Performance & Responsive | 0/? | Not started | - |
| 13. Usage Guidance | 0/? | Not started | - |
