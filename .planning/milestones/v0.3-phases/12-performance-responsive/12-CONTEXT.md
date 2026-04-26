# Phase 12: Performance & Responsive - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase implements the performance & responsive methodology in `qa-tester.md` — the Tier 2 visual checks triggered by `## Visual Focus: performance-responsive` in a spec. It covers four requirement areas: Core Web Vitals measurement with element attribution (PERF-01), cross-browser rendering comparison (PERF-02), breakpoint sweep for responsive bugs (PERF-03), and UX anti-pattern detection (PERF-04). Phase 8 created the spec format infrastructure (`## Visual Focus` tiering, `## Browsers` engine selection); Phases 9-11 established the dual pattern (eval probes + visual judgment). This phase fills in the methodology for performance and responsive verification.

</domain>

<decisions>
## Implementation Decisions

### Core Web Vitals Measurement (PERF-01)
- **D-01:** Raw PerformanceObserver via `playwright-cli eval` — no external dependencies, no CDN fetches, no library injection. The agent runs JavaScript directly in the page context to read CLS, LCP, and INP entries from browser performance APIs.
- **D-02:** LCP measurement requires a simulated interaction (`page.click('body')` or equivalent) before reading — the browser does not finalize the LCP entry until user input occurs. The agent must trigger this interaction step explicitly.
- **D-03:** CLS measurement must be read after dispatching a `visibilitychange` event (or equivalent page-hide trigger) — reading CLS mid-session returns an incomplete value. The agent triggers this before reading the final CLS score.
- **D-04:** INP is only meaningful when the agent has interacted with the page (clicks, form fills, navigations). For passively loaded pages with no interactions, INP should report "not measured" rather than "good" or 0.
- **D-05:** All Web Vitals findings are labeled as **lab-environment values** — these are synthetic measurements from a test run, not field/CrUX data. The agent is honest about this distinction in every performance finding.
- **D-06:** Element attribution — the agent identifies WHICH elements or interactions cause poor scores (e.g., "LCP element: hero image at 3.2s", "CLS shift: ad banner inserting above fold"). Uses PerformanceObserver entry attributes (`element`, `sources`, `target`) to attribute findings.

### Cross-Browser Rendering Comparison (PERF-02)
- **D-07:** Computed style comparison via `playwright-cli eval` as primary mechanism — run the same `getComputedStyle` / `getBoundingClientRect` probes in each browser engine, diff the results. High confidence when eval-detected.
- **D-08:** Chromium is the baseline ("correct") — any other engine (WebKit, Firefox) that diverges gets flagged as a cross-browser finding. This is consistent with the existing Browsers section which defaults to Chromium.
- **D-09:** Target property list (~15-20 properties): `fontFamily`, `fontWeight`, `lineHeight`, `letterSpacing`, `gap` (flexbox), `scrollbarWidth`, `borderRadius` (sub-pixel rendering), `position: sticky` threshold behavior, `display` (flex/grid support), `overflow` behavior, `transform` rendering, `opacity` sub-pixel, `boxShadow`, `filter/backdrop-filter` support.
- **D-10:** Intentional divergences explicitly exempted to suppress noise: native form controls (`<select>`, `<input type="date">`), scrollbar appearance, system font fallbacks, `-webkit-` prefixed properties. These are by-design differences, not bugs.
- **D-11:** Screenshot supplement — after eval comparison, take screenshots in each engine at the same viewport size. Visual judgment catches perceptual differences that individual property probes miss (Medium confidence).

### Breakpoint Sweep (PERF-03)
- **D-12:** Two-phase approach — discrete breakpoints by default, continuous sweep as escalation path.
  - **Phase 1 (default):** Test at 6 known breakpoints: 320, 375, 768, 1024, 1280, 1440px. Fast, named, actionable findings tied to specific widths.
  - **Phase 2 (escalation):** Continuous sweep in ~50px increments from 320→1440. Activates when (a) a discrete test fails (to find the exact failure boundary), or (b) the spec opts in via a flag.
- **D-13:** Overflow detection runs at every width regardless of phase — `evaluate()` checks `document.documentElement.scrollWidth > window.innerWidth` on the document and `scrollWidth > clientWidth` on key container elements.
- **D-14:** Additional checks at each width: content overlap (elements with overlapping `getBoundingClientRect` values), text truncation (`scrollWidth > clientWidth` on text containers), hidden content that shouldn't be (`display: none` or `visibility: hidden` elements that were visible at adjacent breakpoints).
- **D-15:** Builds on existing multi-viewport infrastructure (line 430+) — the breakpoint sweep is an additional dedicated methodology that runs as part of `performance-responsive` Tier 2, not a replacement for the existing viewport testing.

### UX Anti-Pattern Detection (PERF-04)
- **D-16:** Dual pattern with fixed detection logic per anti-pattern — eval probes for structural checks (High confidence) + behavioral testing for interaction-dependent checks. Matches established agent methodology.
- **D-17:** Four required anti-patterns with specific detection methods:
  - **Modal without close button:** Eval query `[role="dialog"]`, `dialog`, `.modal` elements for presence of a close/dismiss mechanism (button with close-related aria-label, X icon, or Escape key handler). High confidence when close mechanism is absent.
  - **Pre-checked opt-ins:** Eval query `input[type="checkbox"][checked]` and `input[type="checkbox"]:checked` in marketing/consent contexts. High confidence when pre-checked in opt-in context.
  - **Cookie banner blocking content:** Eval `getBoundingClientRect` intersection check — measure whether banner/overlay element overlaps `[role="main"]` or main content area. High confidence when intersection confirmed.
  - **Infinite scroll without position recovery:** Behavioral sequence: scroll down (accumulate content), navigate to a detail page, navigate back, eval `window.scrollY` — did position restore to pre-navigation value? High confidence when eval-confirmed position loss.
- **D-18:** Fixed list, not spec-extensible — the four required anti-patterns are the scope for this phase. Extensibility adds complexity without clear payoff.

### Overarching Pattern
- **D-19:** All four areas follow the dual pattern established in Phase 9: `evaluate()` for deterministic/measurable checks (High confidence) + visual judgment via screenshots for perceptual quality (Medium confidence).

### Claude's Discretion
- Exact JS evaluate snippets for PerformanceObserver setup and reading
- Threshold values for "poor" Web Vitals scores (can use Google's published good/needs-improvement/poor thresholds)
- Exact CSS property list for cross-browser comparison (the ~15-20 target list is a starting point)
- How to structure the methodology sections in qa-tester.md (section naming, ordering)
- Breakpoint sweep increment size for continuous mode (50px is guidance, not rigid)
- How to detect "marketing/consent context" for pre-checked opt-in checks
- Screenshot timing and frequency during breakpoint sweep

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Spec Format Infrastructure (Phase 8 output)
- `agents/qa-tester.md` — Lines 733-747: Visual Focus Trigger including `performance-responsive` area definition
- `agents/qa-tester.md` — Line 743: `performance-responsive` area stub ("Core Web Vitals, cross-browser rendering, viewport sweep")
- `agents/qa-tester.md` — Line 747: Pending note to remove ("Tier 2 visual methodology for `performance-responsive` will be added in Phase 12")
- `agents/qa-tester.md` — Lines 1819-1827: Browsers section (engine switching, engine-tagged findings)
- `examples/SPEC-FORMAT.md` — Visual Focus section reference with `performance-responsive` area
- `examples/SPEC-FORMAT.md` — Browsers section reference
- `skills/run/SKILL.md` — Lines 142-149: Visual Focus forwarding block in agent invocation template

### Existing Patterns to Extend
- `agents/qa-tester.md` — Lines 749-1097+: Design Verification + UX State Verification + Layout & Content Integrity methodology (Phases 9-11 output — dual eval+visual pattern, confidence framework, section structure to mirror)
- `agents/qa-tester.md` — Lines 430-451: Multi-Viewport Testing (existing viewport infrastructure — breakpoint sweep builds on this, does not replace)
- `agents/qa-tester.md` — Lines 355-375: Confidence Summary Table (extend with PERF-* entries)

### Requirements
- `.planning/REQUIREMENTS.md` §PERF-01 — Core Web Vitals (CLS, LCP, INP) with element attribution
- `.planning/REQUIREMENTS.md` §PERF-02 — Cross-browser rendering (Chromium, WebKit) with difference reporting
- `.planning/REQUIREMENTS.md` §PERF-03 — Viewport sweep for horizontal scroll, overlap, breakpoint bugs
- `.planning/REQUIREMENTS.md` §PERF-04 — UX anti-patterns (modal close, cookie banner, opt-ins, infinite scroll)

### Prior Phase Context
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — Visual Focus tiering decisions, Browsers section decisions
- `.planning/phases/09-design-verification/09-CONTEXT.md` — Dual pattern decisions (D-15), confidence framework, methodology structure
- `.planning/phases/10-ux-state-verification/10-CONTEXT.md` — Active triggering, behavioral testing sequences
- `.planning/phases/11-layout-content-integrity/11-CONTEXT.md` — Overflow detection, sibling comparison, cross-page eval patterns

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **`playwright-cli eval`** — run arbitrary JS for PerformanceObserver, getComputedStyle, getBoundingClientRect, scrollWidth checks
- **`playwright-cli screenshot`** — visual state capture for perceptual checks and cross-browser comparison
- **`playwright-cli resize`** — viewport resize for breakpoint sweep (existing command, line 60)
- **`playwright-cli snapshot`** — accessibility tree for element identification (modal detection, form controls)
- **`playwright-cli navigate`** / **`playwright-cli navigate-back`** — navigation for infinite scroll position recovery test
- **`playwright-cli click`** — interaction triggering for INP measurement and behavioral anti-pattern tests
- **`playwright-cli route`** — available for network interception if needed (not primary for this phase)
- **Confidence framework** — High/Medium/Low with rationale, established throughout qa-tester.md
- **Overflow detection** — `scrollWidth > clientWidth` pattern already used in DESIGN-02 and LAYOUT-01/03

### Established Patterns
- **Dual pattern** (eval + visual judgment) — established in Phase 9, used in Phases 10-11
- **Visual Focus Trigger** — `performance-responsive` area stub exists at line 743, pending note at line 747
- **Browsers section** — engine switching and engine-tagged findings at lines 1819-1827
- **Multi-viewport testing** — existing infrastructure at lines 430-451 with default viewports
- **Engine-tagged findings** — "[webkit] Login form — PASS" pattern already documented

### Integration Points
- `agents/qa-tester.md` — New Performance & Responsive methodology section goes here (after Layout & Content Integrity)
- `agents/qa-tester.md` line 747 — Remove "Tier 2 visual methodology pending Phase 12" note for `performance-responsive`
- `agents/qa-tester.md` Confidence Summary Table — Extend with PERF-* confidence entries
- `agents/qa-tester.md` Browsers section — May need cross-reference to new cross-browser comparison methodology

</code_context>

<specifics>
## Specific Ideas

### Lab Values, Not Field Data
The agent must be upfront that Web Vitals measurements are synthetic lab values — they reflect the test environment, not real user conditions. Findings should say "Lab LCP: 3.2s (hero image)" not just "LCP: 3.2s". This prevents users from treating agent findings as equivalent to CrUX/RUM data.

### Two-Phase Breakpoint as Smart Default
The discrete-first, continuous-on-failure approach means most test runs stay fast (6 resize calls) while still catching mid-range bugs when there's signal to investigate. This mirrors the existing Tier 1/Tier 2 depth philosophy — baseline always runs, deep check activates when triggered.

### Anti-Pattern Detection as Behavioral Tests
Two of the four anti-patterns (cookie banner overlap, infinite scroll recovery) require interaction sequences, not just DOM inspection. This makes PERF-04 more behaviorally complex than the eval-only patterns in earlier phases — the agent must execute multi-step interaction flows and verify outcomes.

### Cross-Browser Baseline Convention
Using Chromium as the "correct" baseline and flagging other engines as divergent matches both the existing Browsers section default (Chromium-only when section absent) and real-world expectations (most developers test in Chrome first). The intentional-divergence exemption list prevents noisy false positives on platform-specific form controls and scrollbars.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 12-performance-responsive*
*Context gathered: 2026-04-20*
