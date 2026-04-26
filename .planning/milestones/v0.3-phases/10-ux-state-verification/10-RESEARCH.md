# Phase 10: UX State Verification - Research

**Researched:** 2026-04-19
**Domain:** Agent methodology — UX state verification patterns in qa-tester.md
**Confidence:** HIGH

## Summary

Phase 10 adds UX State Verification as a Tier 2 visual methodology to `agents/qa-tester.md`. All implementation decisions are locked in CONTEXT.md from the discussion phase — this is a pure authoring task with no external library selection or environment dependencies.

The work follows two patterns already established in the codebase: (1) the Visual Focus tiering infrastructure from Phase 8, and (2) the dual eval+visual-judgment pattern from Phase 9's Design Verification. Phase 10 fills in the `ux-states` methodology stub at line 747, extends the DESIGN-04 hover/focus pattern into a full 8-state interaction matrix, and adds STATE-* entries to the Confidence Summary Table.

The key technical insight is that state verification has two distinct modes: active triggering (force states into existence via route interception and storage manipulation) and passive DOM probing (fallback when active triggering would be destructive or spec provides no guidance). Active triggering catches "state defined but never fires" bugs that passive checks miss entirely.

**Primary recommendation:** Implement as a new `### UX State Verification` section in `agents/qa-tester.md` immediately after `### Design Verification` (line ~1098), following the identical subsection structure and confidence table pattern Phase 9 established.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**State Existence Detection (STATE-01)**
- D-01: Active triggering as default methodology — agent manipulates app state to force each UI state, then verifies it renders correctly. Goes beyond markup existence — confirms states actually fire under the right conditions.
- D-02: Per-state-type trigger heuristics:
  - Empty state: Clear storage (`localstorage-clear`, `cookie-delete`) + mock empty API response via `playwright-cli route` returning `[]`
  - Loading state: Intercept + delay API responses via `playwright-cli route` to hold the loading state visible
  - Error state: Route returning 5xx status codes (uses existing error simulation pattern from Phase 3 ERR-01)
  - First-run/onboarding: Clear all storage + cookies to simulate a fresh user session
- D-03: Passive DOM probing as fallback — when spec provides no trigger guidance or when active manipulation would be destructive, query DOM for state-indicator patterns (empty containers, skeleton elements, error boundaries, onboarding markers) via `evaluate()`.
- D-04: State cleanup after each trigger — agent must restore app state before proceeding to the next state check to prevent state bleed.

**Interaction State Matrix (STATE-02)**
- D-05: Interaction-first sweep — trigger states via hover/click/Tab like a user, eval to read computed styles after each trigger. Extends existing DESIGN-04 hover/focus verification pattern to the full state matrix.
- D-06: Component selection via accessibility tree snapshot — sweep elements matching: `button`, `[role="button"]`, `a`, `input`, `select`, `textarea`, `[tabindex]`.
- D-07: State trigger sequence per component: default (baseline eval) → hover (`playwright-cli hover`) → focus (Tab to element) → active (screenshot during click — visual judgment only, mousedown is too momentary for eval) → disabled (find disabled variants via `[disabled]` or `aria-disabled` query) → loading/error/success (navigate to states where app exposes them, or use `playwright-cli route` for error simulation).
- D-08: Active state is visual-judgment-only (Medium confidence) — the mousedown state resolves in milliseconds, making eval capture unreliable. Screenshot during click sequence + visual assessment.

**Interactive Feedback (STATE-03)**
- D-09: Cursor correctness via `getComputedStyle(el).cursor` eval probe — screenshots don't render cursors in headless Playwright. Check that buttons/links/interactive `[role]` elements return `pointer` and non-interactive elements return `default`/`auto`. High confidence when eval-confirmed.
- D-10: Toast/notification verification uses presence + disappearance assertions — assert visible after trigger action, then assert hidden/detached for auto-dismiss. Duration measurement only when spec explicitly declares a timing threshold (opt-in via spec, not default).
- D-11: Sticky header verification via two-step eval — (1) confirm `position: sticky` or `position: fixed` via computed style, (2) scroll via `mouse.wheel(0, 500)`, re-read `getBoundingClientRect().top` — value near 0 (or within declared `top` offset) confirms sticking. Screenshot after scroll as visual supplement.
- D-12: Content obscured by sticky header flagged as Medium confidence via visual judgment (supplements the eval probe).

**Animation & Transition Quality (STATE-04)**
- D-13: CSS computed-style inference as primary mechanism — eval reads `transition`, `animation`, `will-change` properties. Checks for: layout-triggering properties (`width`/`height` transitions vs `transform`/`opacity`), missing transitions on interactive elements, duration inconsistency across same component class.
- D-14: No frame-rate measurement — Playwright exposes no FPS API. The agent is honest about this ceiling. Structural quality checks via eval are High confidence; perceived smoothness cannot be verified mechanically.
- D-15: Screenshot judgment limited to loading animation presence — during async operations, a screenshot can confirm a spinner/skeleton exists vs. static text. Binary check: animation present or absent.
- D-16: Complements existing reduced-motion checks (a11y Tier 2 lines 288-297) — those verify animation suppression when `prefers-reduced-motion: reduce` is set. This covers animation quality when animations ARE enabled.

**Overarching Pattern**
- D-17: All four areas follow the dual pattern established in Phase 9: `evaluate()` for deterministic/measurable checks (High confidence) + visual judgment via screenshots for perceptual quality (Medium confidence).

### Claude's Discretion
- Exact JS evaluate snippets for state detection heuristics (empty state selectors, skeleton class patterns)
- Threshold values for sticky header position tolerance
- How to structure the methodology sections in qa-tester.md (section naming, ordering)
- Which interactive elements to prioritize in the state matrix sweep when many are present
- State cleanup implementation details (restore strategy after active triggering)

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| STATE-01 | Agent verifies empty states, loading states, error illustration states, and first-run/onboarding states exist | D-01 through D-04: active triggering methodology + passive fallback; route interception already available via `playwright-cli route` |
| STATE-02 | Agent systematically sweeps every interactive component through its full state matrix (default, hover, focus, active, disabled, loading, error, success) | D-05 through D-08: extends DESIGN-04 hover/focus pattern at lines 910-968; same eval-after-interaction approach |
| STATE-03 | Agent checks hover state coverage, cursor correctness, toast/notification behavior, and scroll behavior (sticky headers, anchor scroll) | D-09 through D-12: cursor via eval (headless limitation); toast via presence+disappearance; sticky via two-step eval |
| STATE-04 | Agent verifies transition smoothness, transition consistency across similar elements, and loading animation presence | D-13 through D-16: CSS property inference; honest FPS ceiling; loading animation as binary screenshot check |
</phase_requirements>

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| UX state verification methodology | Agent (qa-tester.md) | — | Methodology lives entirely in the agent's instruction set; no separate tier |
| State triggering (route interception, storage clear) | Agent via playwright-cli | — | Playwright-cli route/storage commands already in agent's toolset |
| Eval probes (cursor, computed styles, sticky) | Agent via playwright-cli eval | — | Same tool used by DESIGN-04 already |
| Visual judgment (screenshots) | Agent via playwright-cli screenshot | — | Same pattern as Phase 9 Design Verification |
| Spec trigger (Visual Focus section) | skills/run/SKILL.md forwarding | agents/qa-tester.md trigger logic | Infrastructure is Phase 8; Phase 10 fills in methodology only |

## Standard Stack

This phase adds no new libraries or dependencies. All implementation uses tools and patterns already in the codebase. [VERIFIED: codebase grep]

### Existing Tools Used

| Tool | Purpose | Established In |
|------|---------|----------------|
| `playwright-cli route` | Intercept/delay/mock network responses for state triggering | Phase 3 ERR-01 |
| `playwright-cli hover` | Trigger hover state before eval reads computed styles | Phase 9 DESIGN-04 |
| `playwright-cli eval` | Run JS for computed styles, cursor check, DOM queries | Phase 9 throughout |
| `playwright-cli run-code` | Multi-step JS execution (e.g., scroll + re-read bbox) | Phase 9 DESIGN-04 |
| `playwright-cli screenshot` | Visual capture for perceptual checks | Phase 1 through 9 |
| `playwright-cli snapshot` | Accessibility tree for interactive element identification | Phase 4 A11Y |
| `localstorage-clear` | Clear storage for empty/first-run state triggering | Phase 3 |
| `cookie-delete` | Clear cookies for first-run state triggering | Phase 3 |

**Installation:** None required.

## Architecture Patterns

### System Architecture Diagram

```
Spec: ## Visual Focus: ux-states
           |
           v
    skills/run/SKILL.md
    (Visual Focus forwarding block, lines 142-149)
           |
           v
    agents/qa-tester.md
    Visual Focus Trigger (lines 733-747)
           |
    [ux-states area requested]
           |
           v
    UX State Verification (Phase 10 — NEW SECTION)
    ┌─────────────────────────────────────────────┐
    │  STATE-01: State Existence                  │
    │  ┌──────────────────┐  ┌──────────────────┐ │
    │  │ Active Trigger   │  │ Passive Fallback  │ │
    │  │ route → mock []  │  │ eval DOM patterns │ │
    │  │ route → delay    │  │ (when destructive │ │
    │  │ route → 5xx      │  │  or no guidance)  │ │
    │  │ clear storage    │  └──────────────────┘ │
    │  └──────────────────┘                       │
    │           |                                  │
    │    STATE CLEANUP (D-04)                      │
    │           |                                  │
    │  STATE-02: Interaction Matrix Sweep          │
    │  snapshot → find interactive elements        │
    │  per element: default→hover→focus→active     │
    │              →disabled→loading/error/success │
    │  eval computed styles after each trigger     │
    │  screenshot at active (visual-only)          │
    │           |                                  │
    │  STATE-03: Interactive Feedback Quality      │
    │  cursor: eval getComputedStyle(el).cursor    │
    │  toast: assert visible → assert gone         │
    │  sticky: eval position → scroll → re-read   │
    │           |                                  │
    │  STATE-04: Animation & Transition Quality    │
    │  eval: transition/animation/will-change      │
    │  screenshot: loading anim present/absent     │
    │  (no FPS — honest ceiling)                   │
    └─────────────────────────────────────────────┘
           |
           v
    Reporting: STATE-* Confidence Table
    (appended to existing Confidence Summary Table)
```

### Recommended Project Structure

No new files. Implementation is a single new section added to:

```
agents/
└── qa-tester.md    # New ### UX State Verification section (~lines 1099+)
                    # Line 747: Remove "pending Phase 10" stub note
                    # Lines 355-375: Extend Confidence Summary Table with STATE-* rows
```

### Pattern 1: Active State Triggering with Passive Fallback

**What:** Attempt to force the UI into each target state using route interception or storage manipulation. If triggering would be destructive or spec provides no guidance, fall back to passive DOM probing.

**When to use:** Default for all four state types in STATE-01. Fallback engages when `playwright-cli route` setup is not feasible or app would lose user data.

**Example (Loading State):**
```javascript
// Source: D-02 decision (codebase pattern — playwright-cli route already used for error sim)
// Intercept API call, delay response to hold loading state visible
playwright-cli route "https://api.example.com/items" "{\"delay\": 3000, \"body\": []}"
// Navigate to trigger fetch
playwright-cli goto "https://app.example.com/items"
// Screenshot immediately — loading spinner/skeleton should be visible
playwright-cli screenshot --filename .qa/reports/state-loading.png
// Remove route to allow real response
playwright-cli route-clear "https://api.example.com/items"
```

**Example (Empty State via passive fallback):**
```javascript
// Source: D-03 decision — passive DOM probe
playwright-cli eval "() => {
  // Look for common empty state patterns
  const emptyPatterns = [
    '[data-empty]',
    '[data-state=\"empty\"]',
    '.empty-state',
    '.no-results',
    '[aria-label*=\"empty\"]',
    '[aria-label*=\"no \"]'
  ];
  return emptyPatterns.map(sel => ({
    selector: sel,
    found: !!document.querySelector(sel),
    visible: (() => {
      const el = document.querySelector(sel);
      if (!el) return false;
      const r = el.getBoundingClientRect();
      return r.width > 0 && r.height > 0;
    })()
  })).filter(r => r.found);
}"
```

### Pattern 2: Interaction State Matrix Sweep (Extension of DESIGN-04)

**What:** Extend the DESIGN-04 hover/focus eval-after-interaction pattern to 8 states per interactive element. Read computed styles after each state trigger.

**When to use:** STATE-02 — always when `ux-states` is in Visual Focus.

**Example (per-component sweep):**
```javascript
// Source: D-05 through D-08 decisions, extending DESIGN-04 lines 943-968 in qa-tester.md

// Step 1: Identify interactive elements via snapshot
playwright-cli snapshot
// -> find element ref for a button, e.g., ref="e42"

// Step 2: Baseline (default state)
playwright-cli eval "() => {
  const el = document.querySelector('button.primary');
  const s = getComputedStyle(el);
  return { bg: s.backgroundColor, color: s.color, opacity: s.opacity,
           border: s.borderColor, cursor: s.cursor,
           transform: s.transform, transition: s.transition };
}"

// Step 3: Hover state
playwright-cli hover e42
playwright-cli eval "() => { /* same properties */ }"

// Step 4: Focus state (Tab navigation)
playwright-cli press "Tab"  // or click then check activeElement
playwright-cli eval "() => { /* same properties */ }"

// Step 5: Active state — visual judgment only (D-08)
// Screenshot during click sequence — mousedown resolves in milliseconds
playwright-cli screenshot --filename .qa/reports/state-active-pre.png
playwright-cli click e42
// Note: active state eval is unreliable; screenshot captures intent

// Step 6: Disabled state — find via DOM query
playwright-cli eval "() => {
  const disabled = Array.from(
    document.querySelectorAll('[disabled], [aria-disabled=\"true\"]')
  ).map(el => ({
    tag: el.tagName,
    text: el.textContent.trim().slice(0, 40),
    hasVisualDisabledStyle: (() => {
      const s = getComputedStyle(el);
      return parseFloat(s.opacity) < 1 || s.cursor === 'not-allowed';
    })()
  }));
  return disabled;
}"
```

### Pattern 3: Cursor Correctness Probe

**What:** eval-only cursor check (screenshots don't render cursors in headless Playwright).

**When to use:** STATE-03 cursor correctness check.

**Example:**
```javascript
// Source: D-09 decision — headless cursor limitation documented
playwright-cli eval "() => {
  const interactive = Array.from(
    document.querySelectorAll('button, [role=\"button\"], a[href], input, select, textarea, [tabindex]')
  ).filter(el => {
    const r = el.getBoundingClientRect();
    return r.width > 0 && r.height > 0; // visible elements only
  });
  
  return interactive.map(el => ({
    tag: el.tagName,
    role: el.getAttribute('role'),
    text: el.textContent.trim().slice(0, 30),
    cursor: getComputedStyle(el).cursor,
    expectsPointer: true,
    correct: getComputedStyle(el).cursor === 'pointer'
  })).filter(el => !el.correct); // return only incorrect cursors
}"
```

### Pattern 4: Sticky Header Verification

**What:** Two-step eval — confirm sticky/fixed position, then scroll and re-read `getBoundingClientRect().top`.

**When to use:** STATE-03 scroll behavior check.

**Example:**
```javascript
// Source: D-11 decision — two-step eval approach
// Step 1: Confirm sticky positioning
playwright-cli eval "() => {
  const header = document.querySelector('header, [role=\"banner\"], nav:first-of-type');
  if (!header) return { found: false };
  const s = getComputedStyle(header);
  return {
    found: true,
    position: s.position,
    top: s.top,
    isSticky: s.position === 'sticky' || s.position === 'fixed'
  };
}"

// Step 2: Scroll and verify position holds
playwright-cli run-code "async (page) => {
  await page.mouse.wheel(0, 500);
  await page.waitForTimeout(300); // let scroll settle
  const header = document.querySelector('header, [role=\"banner\"]');
  const rect = header.getBoundingClientRect();
  const stickyTop = parseInt(getComputedStyle(header).top) || 0;
  return {
    top: rect.top,
    expectedTop: stickyTop,
    sticking: Math.abs(rect.top - stickyTop) <= 2 // 2px tolerance
  };
}"

// Visual supplement: screenshot after scroll
playwright-cli screenshot --filename .qa/reports/state-sticky-scroll.png
```

### Pattern 5: Animation/Transition Quality Probe

**What:** CSS computed-style inference for structural animation quality. Binary screenshot check for loading animation presence.

**When to use:** STATE-04.

**Example:**
```javascript
// Source: D-13 decision — CSS property inference
playwright-cli eval "() => {
  const interactive = Array.from(
    document.querySelectorAll('button, a, input, [role=\"button\"]')
  ).filter(el => {
    const r = el.getBoundingClientRect();
    return r.width > 0 && r.height > 0;
  });
  
  return interactive.map(el => {
    const s = getComputedStyle(el);
    const transition = s.transition;
    const animation = s.animation;
    const willChange = s.willChange;
    
    // Flag layout-triggering properties (width/height) vs performant (transform/opacity)
    const hasLayoutTrigger = /\\bwidth\\b|\\bheight\\b|\\btop\\b|\\bleft\\b/.test(transition);
    const hasPerformantProp = /transform|opacity/.test(transition);
    
    return {
      tag: el.tagName,
      text: el.textContent.trim().slice(0, 30),
      transition,
      hasTransition: transition !== 'none' && transition !== 'all 0s',
      hasLayoutTrigger,
      hasPerformantProp,
      willChange
    };
  });
}"
```

### Anti-Patterns to Avoid

- **Passive-only state detection:** Checking DOM for state markup without forcing states into existence misses "state defined but never fires" bugs. Active triggering is the default (D-01).
- **Skipping state cleanup:** Running multiple state triggers without restoring app state between them causes state bleed — later checks run against contaminated state (D-04).
- **Eval-ing active (mousedown) state:** The mousedown state resolves in <100ms — eval capture after click will see the post-active state, not the active state. Use screenshot judgment only (D-08).
- **Cursor screenshots:** Playwright headless mode does not render cursors in screenshots. Cursor correctness MUST use eval `getComputedStyle(el).cursor` (D-09).
- **Measuring FPS:** Playwright has no FPS API. Do not attempt to instrument `requestAnimationFrame` timing as a smoothness proxy — structural CSS property inference is the honest ceiling (D-14).
- **Conflating reduced-motion checks:** The existing Reduced Motion check (A11Y-12, lines 286-297) verifies animation suppression when `prefers-reduced-motion: reduce`. STATE-04 verifies animation quality when animations ARE enabled. These are complementary, not overlapping (D-16).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Network response mocking | Custom intercept infrastructure | `playwright-cli route` (already in agent) | Already implemented in Phase 3; reuse established pattern |
| Hover state triggering | Mouse position manipulation | `playwright-cli hover <ref>` | Already used in DESIGN-04 lines 956 |
| Scroll triggering | Custom scroll injection | `playwright-cli run-code` with `mouse.wheel()` | Playwright's native scroll API handles cross-browser edge cases |
| State-indicator DOM patterns | Hardcoded class list | Heuristic patterns + spec override | Apps vary; agent should use heuristics with spec-provided guidance taking priority |

**Key insight:** This phase extends existing playwright-cli capabilities rather than introducing new mechanisms. Every tool needed is already in the agent's toolset.

## Common Pitfalls

### Pitfall 1: Route Interception Scope Bleed
**What goes wrong:** A `playwright-cli route` mock for empty state testing remains active when the loading state check begins, causing the loading state to immediately resolve empty rather than showing loading UI.
**Why it happens:** Route mocks persist for the session unless explicitly cleared.
**How to avoid:** After each active state trigger completes, clear the route mock before setting up the next one (D-04 state cleanup).
**Warning signs:** Loading state check completes instantly with no spinner/skeleton visible.

### Pitfall 2: Accessibility Tree Miss on Dynamically Inserted Elements
**What goes wrong:** `playwright-cli snapshot` is taken before an async operation completes, missing interactive elements that are rendered after data loads.
**Why it happens:** Snapshot captures state at the moment it runs.
**How to avoid:** Take the interaction matrix snapshot after page has settled (no pending network requests). If the spec page has async-rendered content, trigger loading first, wait for content, then snapshot.
**Warning signs:** Snapshot shows fewer interactive elements than visible in a screenshot.

### Pitfall 3: Active State False-Negative
**What goes wrong:** Agent takes screenshot for active state check but the button has already returned to its resting state by the time the screenshot saves.
**Why it happens:** Mousedown → mouseup is faster than playwright-cli screenshot I/O.
**How to avoid:** Accept this as a known limitation (D-08). Report active state as Medium confidence visual judgment. Do not fail tests because active state could not be captured — note it as "active state: screenshot-only, unreliable capture window."
**Warning signs:** Active state screenshot looks identical to default state even for elements with visible press feedback.

### Pitfall 4: Sticky Header False-Positive
**What goes wrong:** `getBoundingClientRect().top` returns a value near 0 after scroll because the page scrolled but the viewport was too short to scroll far enough to test stickiness.
**Why it happens:** On short content pages, `mouse.wheel(0, 500)` may not have anything to scroll to.
**How to avoid:** Check `document.body.scrollHeight > window.innerHeight` before running the sticky scroll test. If the page isn't scrollable, note "page content too short to verify sticky behavior — observation only."
**Warning signs:** `scrollY` remains 0 after `mouse.wheel()` call.

### Pitfall 5: Toast Timing Assertion Failure
**What goes wrong:** Agent asserts toast is hidden/detached immediately after asserting visible, causing a race where the toast hasn't auto-dismissed yet but the re-check fires too quickly.
**Why it happens:** Toast auto-dismiss timings range from 2-8 seconds; polling too fast creates a false "toast stuck" finding.
**How to avoid:** Per D-10, duration measurement is opt-in via spec. For default behavior, check presence first, then wait a reasonable period (use `playwright-cli run-code` with `waitForTimeout`) before asserting dismissal. Flag as Medium confidence when no timing spec is declared.
**Warning signs:** Toast assertion failures on elements that are visibly auto-dismissing.

## Code Examples

Verified patterns from existing qa-tester.md and locked CONTEXT.md decisions:

### Empty State Passive Probe
```javascript
// Source: D-03 decision — DOM heuristic patterns
playwright-cli eval "() => {
  const selectors = [
    '[data-empty]', '[data-state=\"empty\"]', '[data-testid*=\"empty\"]',
    '.empty-state', '.empty', '.no-results', '.no-items', '.no-data',
    '[aria-label*=\"empty\"]', '[aria-label*=\"no items\"]',
    '[class*=\"empty\"]', '[id*=\"empty\"]'
  ];
  // Also check: is a list container present but has 0 children?
  const listContainers = Array.from(
    document.querySelectorAll('ul, ol, [role=\"list\"], [role=\"grid\"]')
  ).filter(el => el.children.length === 0 && el.offsetParent !== null);
  
  return {
    bySelector: selectors.filter(s => document.querySelector(s)),
    emptyListContainers: listContainers.map(el => ({
      tag: el.tagName,
      id: el.id,
      className: el.className.slice(0, 50)
    }))
  };
}"
```

### Loading State Active Trigger (with cleanup)
```javascript
// Source: D-02 and D-04 decisions
// 1. Set up route delay
playwright-cli route "*/api/*" "{\"delay\": 2000}"
// 2. Navigate to trigger fetch
playwright-cli goto "[url]"
// 3. Screenshot while delayed
playwright-cli screenshot --filename .qa/reports/state-loading.png
// 4. Verify loading indicators
playwright-cli eval "() => {
  const spinners = document.querySelectorAll(
    '[role=\"progressbar\"], [aria-label*=\"loading\"], [class*=\"spinner\"], [class*=\"skeleton\"], [class*=\"loader\"]'
  );
  return { count: spinners.length, found: spinners.length > 0 };
}"
// 5. CLEANUP — remove route mock before next check (D-04)
playwright-cli route-clear "*/api/*"
```

### First-Run State Trigger
```javascript
// Source: D-02 decision
// Clear all storage to simulate fresh user session
playwright-cli run-code "async (page) => {
  await page.context().clearCookies();
}"
playwright-cli localstorage-clear
playwright-cli goto "[url]"
playwright-cli screenshot --filename .qa/reports/state-first-run.png
// Look for onboarding markers
playwright-cli eval "() => {
  const onboardingSelectors = [
    '[data-tour]', '[data-onboarding]', '[data-welcome]',
    '.onboarding', '.welcome', '.getting-started',
    '[aria-label*=\"welcome\"]', '[aria-label*=\"get started\"]'
  ];
  return onboardingSelectors.map(s => ({
    selector: s,
    found: !!document.querySelector(s),
    visible: (() => {
      const el = document.querySelector(s);
      if (!el) return false;
      const r = el.getBoundingClientRect();
      return r.width > 0 && r.height > 0;
    })()
  })).filter(r => r.found);
}"
```

### Transition Consistency Check (Same Component Class)
```javascript
// Source: D-13 decision — duration inconsistency across same component class
playwright-cli eval "() => {
  const buttons = Array.from(document.querySelectorAll('button'));
  const transitions = buttons
    .filter(el => el.offsetParent !== null) // visible only
    .map(el => ({
      text: el.textContent.trim().slice(0, 25),
      transition: getComputedStyle(el).transition,
      transitionDuration: getComputedStyle(el).transitionDuration
    }));
  
  // Group by transition-duration to find inconsistencies
  const durations = [...new Set(transitions.map(t => t.transitionDuration))];
  return {
    transitions,
    uniqueDurations: durations,
    inconsistent: durations.length > 1
  };
}"
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Check if empty state markup exists in DOM | Active triggering — force state, verify it fires | Phase 10 (this phase) | Catches "state defined but never triggered" class of bugs |
| hover/focus only (DESIGN-04) | Full 8-state matrix per interactive element | Phase 10 (this phase) | Catches missing disabled, error, success state styling |
| No cursor check | eval `getComputedStyle(el).cursor` | Phase 10 (this phase) | Only viable method in headless Playwright |

**Honest limitations (not deprecated, just ceilings):**
- FPS measurement: No Playwright API for this. CSS property inference is the honest ceiling.
- Active (mousedown) state: Resolves in milliseconds. Screenshot-only, Medium confidence.
- Cursor rendering in screenshots: Not possible in headless mode. Eval-only.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `playwright-cli route-clear` syntax clears a previously set route mock | Code Examples | Plan would need to specify exact cleanup syntax based on playwright-cli docs |
| A2 | `page.context().clearCookies()` is available in the `playwright-cli run-code` execution context | Code Examples | First-run state trigger would need alternative cookie clearing approach |
| A3 | `playwright-cli localstorage-clear` is a valid command (seen referenced in CONTEXT.md D-02) | Code Examples | Empty/first-run trigger would need alternative |

**Note:** A1-A3 are about exact playwright-cli command syntax in the code examples. The CONTEXT.md confirms these capabilities exist (D-02 references them); exact syntax is discretionary per CONTEXT.md "Claude's Discretion." The planner should note that code examples are illustrative — exact playwright-cli command syntax should be confirmed against the playwright-cli docs or existing patterns already in qa-tester.md.

## Open Questions

1. **playwright-cli route-clear syntax**
   - What we know: `playwright-cli route` for mocking is established (Phase 3 ERR-01)
   - What's unclear: The exact command to remove/clear a previously set route mock
   - Recommendation: Check existing qa-tester.md error simulation section for the cleanup pattern; if absent, use `playwright-cli run-code` with `page.unroute()` as the removal mechanism

2. **State matrix element prioritization at scale**
   - What we know: D-06 says sweep `button`, `[role="button"]`, `a`, `input`, `select`, `textarea`, `[tabindex]`
   - What's unclear: A page with 50+ interactive elements — complete sweep is impractical
   - Recommendation: In the methodology, define a tiering: primary buttons and CTAs first, navigation links second, form inputs third, tertiary/decorative elements only if time permits. Spec can override via explicit component list.

## Environment Availability

Step 2.6: SKIPPED (no external dependencies — phase is a pure methodology addition to agents/qa-tester.md; all playwright-cli capabilities are already available).

## Validation Architecture

`workflow.nyquist_validation` key is absent from `.planning/config.json`, treated as enabled.

### Test Framework

This project does not have an automated test suite for the agent instruction files — the agent's "tests" are functional QA runs against real web applications. There is no pytest/jest/vitest config. [VERIFIED: codebase — no test config files found]

| Property | Value |
|----------|-------|
| Framework | None — agent instruction validation is manual |
| Config file | none |
| Quick run command | Manual: install plugin, open test app, run `/qa:run` with ux-states focus |
| Full suite command | Manual: full spec run with `## Visual Focus: ux-states` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| STATE-01 | State existence detection fires for empty/loading/error/first-run | manual-only | `/qa:run` against app with triggerable states | N/A |
| STATE-02 | Interaction matrix sweep covers all 8 states per component | manual-only | `/qa:run` with `## Visual Focus: ux-states` | N/A |
| STATE-03 | Cursor/toast/sticky checks execute and produce findings | manual-only | `/qa:run` against app with sticky header | N/A |
| STATE-04 | Transition quality probe reports CSS property findings | manual-only | `/qa:run` against app with animated elements | N/A |

**Justification for manual-only:** Agent methodology lives in a markdown instruction file (`agents/qa-tester.md`). Correctness is verified by reading the instructions against the decisions in CONTEXT.md and by running the agent against a real app. There is no programmatic unit-test harness for agent instruction text. This is the established pattern throughout all prior phases (Phase 9 used the same approach).

### Wave 0 Gaps

None — no test infrastructure to scaffold. Validation is by code review of the methodology section against CONTEXT.md decisions, then manual functional verification.

## Security Domain

`security_enforcement` key is absent from `.planning/config.json`, treated as enabled.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | No | State triggering is storage/route manipulation — no auth endpoints |
| V3 Session Management | Marginal | First-run trigger clears cookies/storage — no new session data written; agent reads app state only |
| V4 Access Control | No | No new access control surfaces introduced |
| V5 Input Validation | No | No user input paths in this methodology |
| V6 Cryptography | No | No cryptographic operations |

### Known Threat Patterns for Agent Instruction Additions

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| CSS property exfiltration via eval | Information Disclosure | Security note from Phase 9 DESIGN-04 already blocks `--auth`, `--token`, `--key`, `--secret` prefixed properties — maintain in STATE-02 eval probes |
| Route mocks persisting beyond test scope | Elevation of Privilege | D-04 state cleanup mandate — route mocks MUST be cleared after each state check |
| First-run trigger destroying real user data | Denial of Service | Active triggering is for test environments only; methodology should include this caveat |

**Security carry-forward from Phase 9:** The eval probe security note (skip CSS properties beginning with `--auth`, `--token`, `--key`, `--secret`) already established in DESIGN-04 MUST be carried into any new eval probes added in this phase's methodology.

## Sources

### Primary (HIGH confidence)
- `agents/qa-tester.md` lines 910-968 — DESIGN-04 hover/focus pattern (exact pattern D-05 extends)
- `agents/qa-tester.md` lines 733-747 — Visual Focus Trigger and `ux-states` area stub
- `agents/qa-tester.md` lines 719-731 — Accessibility Focus Trigger (structural template to mirror)
- `agents/qa-tester.md` lines 240-310 — Structured Accessibility Tier 2 (methodology section template)
- `agents/qa-tester.md` lines 355-375 — Confidence Summary Table (to extend with STATE-*)
- `agents/qa-tester.md` lines 749-1098 — Design Verification (Phase 9 output — dual pattern reference)
- `.planning/phases/10-ux-state-verification/10-CONTEXT.md` — All locked decisions D-01 through D-17
- `.planning/REQUIREMENTS.md` — STATE-01 through STATE-04 definitions

### Secondary (MEDIUM confidence)
- `.planning/STATE.md` — Phase history and decision log confirming playwright-cli capability set

### Tertiary (LOW confidence)
- None — all claims verified from codebase or CONTEXT.md

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new libraries; all tools verified present in codebase
- Architecture: HIGH — structure mirrors Phase 9 Design Verification which is already implemented
- Pitfalls: HIGH — derived from locked decisions that explicitly call out limitations (D-08, D-09, D-14)
- Code examples: MEDIUM — illustrative; exact playwright-cli syntax for route-clear needs verification against existing patterns

**Research date:** 2026-04-19
**Valid until:** 2026-05-19 (stable — no external dependencies)
