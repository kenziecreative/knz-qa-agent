# Phase 12: Performance & Responsive - Research

**Researched:** 2026-04-20
**Domain:** Web Vitals measurement via PerformanceObserver, cross-browser CSS comparison, breakpoint sweep automation, UX anti-pattern behavioral detection
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Core Web Vitals (PERF-01):**
- D-01: Raw PerformanceObserver via `playwright-cli eval` — no external dependencies, no CDN fetches, no library injection
- D-02: LCP requires a simulated body click before reading the final entry
- D-03: CLS requires dispatching a `visibilitychange` event (or equivalent page-hide trigger) before reading the final score
- D-04: INP reports "not measured" for pages with no agent interactions
- D-05: All findings labeled as **lab-environment values** — not field/CrUX data
- D-06: Element attribution via PerformanceObserver entry attributes (`element`, `sources`, `target`)

**Cross-Browser Rendering (PERF-02):**
- D-07: `getComputedStyle` / `getBoundingClientRect` eval probes as primary mechanism — diff results per engine
- D-08: Chromium is the baseline ("correct") — other engines flagged as divergent
- D-09: Target property list ~15-20 properties (see Standard Stack section)
- D-10: Intentional divergences explicitly exempted: native form controls, scrollbar appearance, system font fallbacks, `-webkit-` prefixed properties
- D-11: Screenshot supplement after eval comparison for perceptual differences

**Breakpoint Sweep (PERF-03):**
- D-12: Two-phase approach — 6 discrete breakpoints default (320, 375, 768, 1024, 1280, 1440px), continuous ~50px sweep as escalation
- D-13: Overflow detection at every width: `document.documentElement.scrollWidth > window.innerWidth` + per-container `scrollWidth > clientWidth`
- D-14: Additional checks per width: content overlap (overlapping `getBoundingClientRect`), text truncation, hidden content that shouldn't be hidden
- D-15: Extends existing multi-viewport infrastructure (lines 430-451) — not a replacement

**UX Anti-Patterns (PERF-04):**
- D-16: Dual pattern — eval probes (High) + behavioral sequences (High) per anti-pattern
- D-17: Four required anti-patterns: modal without close button, pre-checked opt-ins, cookie banner blocking content, infinite scroll without position recovery
- D-18: Fixed list, not spec-extensible

**Overarching:**
- D-19: All four areas follow dual pattern: `evaluate()` (High confidence) + screenshot visual judgment (Medium confidence)

### Claude's Discretion
- Exact JS eval snippets for PerformanceObserver setup and reading
- Threshold values for "poor" Web Vitals scores
- Exact CSS property list for cross-browser comparison (~15-20 is a starting point)
- How to structure methodology sections in qa-tester.md (naming, ordering)
- Breakpoint sweep increment size for continuous mode
- How to detect "marketing/consent context" for pre-checked opt-in checks
- Screenshot timing and frequency during breakpoint sweep

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| PERF-01 | Agent measures Core Web Vitals (CLS, LCP, INP) and identifies which elements/interactions cause poor scores | PerformanceObserver raw JS patterns verified; LCP finalization via body click; CLS finalization via visibilitychange; INP via event type observer; element attribution via entry.element, entry.sources |
| PERF-02 | Agent runs visual checks across browser engines (Chromium, WebKit) and reports rendering differences | getComputedStyle diff pattern; ~18 property target list researched; intentional divergence exemption list; screenshot supplement pattern; existing Browsers section infrastructure |
| PERF-03 | Agent sweeps viewport widths to catch horizontal scroll, content overlap, and off-by-one breakpoint bugs | playwright-cli resize command available; two-phase sweep pattern; overflow detection reuses DESIGN-02/LAYOUT-01 pattern; breakpoints defined |
| PERF-04 | Agent detects UX anti-patterns (modal without close, cookie banner overlap, pre-checked opt-ins, infinite scroll without position) | Detection logic defined per anti-pattern; getBoundingClientRect intersection pattern for banner overlap; navigate/navigate-back + scrollY for position recovery |
</phase_requirements>

---

## Summary

Phase 12 fills in the `performance-responsive` Tier 2 methodology stub in `qa-tester.md`. The phase is purely additive to one file — no new tools, no new infrastructure, no spec format changes. All four requirement areas (PERF-01 through PERF-04) follow the dual pattern established in Phases 9-11: eval probes for deterministic checks (High confidence) plus screenshot visual judgment for perceptual quality (Medium confidence).

The main implementation challenge is the PerformanceObserver lifecycle. Web Vitals are event-driven metrics that require specific lifecycle triggers to finalize: LCP needs a user interaction (body click) to emit its final entry, and CLS needs a page-visibility change to flush the accumulated score. Both patterns are well-documented and work in Playwright's eval context using a global variable accumulator approach: the observer writes to a `window.__qa*` global via `run-code`, then subsequent `eval` calls read the accumulated value. This avoids the async-callback-inside-eval problem.

The cross-browser comparison section extends the existing Browsers section in the agent by adding a structured diff procedure. The breakpoint sweep extends the existing multi-viewport infrastructure (lines 430-451) with a two-phase strategy: 6 discrete widths by default, continuous sweep only when a discrete width fails. The UX anti-patterns section is the most behaviorally complex — two of the four anti-patterns (cookie banner overlap and infinite scroll position recovery) require multi-step interaction sequences, not just DOM inspection.

**Primary recommendation:** Write the new `### Performance & Responsive` section in qa-tester.md to mirror the structure of the three preceding visual sections (Design Verification, UX State Verification, Layout & Content Integrity). One subsection per requirement area, each with numbered step procedures and a confidence table at the end. Insert it between the Layout & Content Integrity confidence table (line ~1802) and the `### Design Reference` section (line 1803). Remove the "pending Phase 12" note from line 747.

---

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| PerformanceObserver setup and reading | Browser (in-page JS) | Agent (orchestrates timing) | Metrics are browser-native; eval injects JS into page context |
| LCP/CLS finalization triggers | Agent (playwright-cli) | Browser (responds to events) | Agent triggers click and visibilitychange, browser finalizes metrics |
| Cross-browser engine switching | QA skill (run/SKILL.md) | Agent (receives active engine) | Skill launches each engine run; agent tags findings |
| CSS property comparison | Agent (eval diff) | — | getComputedStyle runs in-browser; diff logic in agent methodology |
| Viewport resize sweep | Agent (playwright-cli resize) | Browser (re-renders) | Agent controls viewport width; browser reports overflow |
| Overflow detection | Browser (in-page JS) | Agent (records findings) | scrollWidth/clientWidth read from DOM; agent evaluates results |
| UX anti-pattern DOM inspection | Browser (in-page JS via eval) | — | Modal, form, and overlay detection via DOM queries |
| UX anti-pattern behavioral testing | Agent (interaction sequences) | Browser (executes) | Agent drives navigate/scroll/back sequences; reads window.scrollY |

---

## Standard Stack

### Core (already available — no new dependencies)

| Tool | Purpose | Notes |
|------|---------|-------|
| `playwright-cli eval` | In-page JS execution for PerformanceObserver, getComputedStyle, getBoundingClientRect, scrollWidth | Supports `async () =>` functions; used throughout Phases 9-11 |
| `playwright-cli run-code` | Async page-level code for multi-step sequences and global state setup | Used in Phases 10-11 for route mocking, media emulation |
| `playwright-cli screenshot` | Visual capture for perceptual comparison supplement | Used throughout all visual phases |
| `playwright-cli resize` | Viewport dimension change for breakpoint sweep | Existing command at agent line 60 |
| `playwright-cli navigate` / `go-back` | Navigation for infinite scroll position recovery test | Existing commands |
| `playwright-cli click` | Interaction triggering for LCP finalization and anti-pattern behavioral tests | Existing command |
| `playwright-cli snapshot` | Accessibility tree for modal/dialog element identification | Existing command |

### No New Packages Required

The entire phase is implemented as methodology text in `agents/qa-tester.md`. No npm installs, no external scripts, no CDN fetches. All browser APIs used are standard (PerformanceObserver, getComputedStyle, getBoundingClientRect, scrollWidth, window.scrollY) — available in all three Playwright browser engines.

**Version verification:** Not applicable — no new package dependencies.

---

## Architecture Patterns

### System Architecture Diagram

```
Spec with "performance-responsive" in ## Visual Focus
         |
         v
qa-run skill parses Visual Focus
         |
         v
Agent receives test assignment with "Visual Focus: performance-responsive"
         |
         +---> PERF-01: Core Web Vitals
         |       |
         |       +--> run-code: inject window.__qaLCP / __qaCLS / __qaINP observers
         |       +--> navigate to target URL (page loads, observers fire)
         |       +--> click 'body' (finalizes LCP entry)
         |       +--> run-code: dispatch visibilitychange (finalizes CLS)
         |       +--> eval: read window.__qaLCP, __qaCLS, __qaINP
         |       +--> compare vs thresholds; attribute element from entry
         |
         +---> PERF-02: Cross-Browser Rendering (runs per engine in active run)
         |       |
         |       +--> eval: getComputedStyle on ~18 target properties per element
         |       +--> compare vs Chromium baseline (stored from prior engine run)
         |       +--> screenshot: visual supplement for perceptual diffs
         |       +--> flag divergences (exempt intentional: form controls, scrollbars)
         |
         +---> PERF-03: Breakpoint Sweep
         |       |
         |       +--> Phase 1: resize to 6 discrete widths (320, 375, 768, 1024, 1280, 1440)
         |       |     +--> eval: scrollWidth > window.innerWidth (overflow check)
         |       |     +--> eval: getBoundingClientRect overlap check
         |       |     +--> eval: text element scrollWidth > clientWidth
         |       |     +--> screenshot at each width
         |       +--> Phase 2 (on failure): continuous ~50px sweep in failing range
         |             +--> same overflow/overlap/truncation checks
         |
         +---> PERF-04: UX Anti-Pattern Detection
                 |
                 +--> [P1] Modal: snapshot/eval for [role="dialog"] without close mechanism
                 +--> [P2] Pre-checked: eval input[type="checkbox"][checked] in consent context
                 +--> [P3] Cookie banner: eval getBoundingClientRect intersection with [role="main"]
                 +--> [P4] Infinite scroll: navigate down -> navigate away -> back -> eval window.scrollY
                      check vs pre-navigation value
```

### Recommended File Change

Only one file changes in this phase:

```
agents/
└── qa-tester.md    # Add ### Performance & Responsive section (~200-300 lines)
                    # Remove "pending Phase 12" note at line 747
                    # Extend Confidence Summary Table with PERF-* entries
```

### Pattern 1: Global Accumulator for Async PerformanceObserver

The PerformanceObserver API is event-driven. Since `playwright-cli eval` expects synchronous return values, the recommended pattern uses `run-code` to inject a PerformanceObserver that writes to a `window.__qa*` global, then reads that global with a later `eval` call after triggering the finalization interaction.

```javascript
// Step 1: Inject observer via run-code (before or right after page load)
playwright-cli run-code "async (page) => {
  await page.evaluate(() => {
    // LCP observer
    window.__qaLCP = { value: 0, element: null };
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const last = entries[entries.length - 1];
      window.__qaLCP = {
        value: last.startTime,
        element: last.element ? last.element.tagName + (last.element.id ? '#' + last.element.id : '') + (last.element.className ? '.' + String(last.element.className).split(' ')[0] : '') : null,
        url: last.url || null
      };
    }).observe({ type: 'largest-contentful-paint', buffered: true });

    // CLS accumulator
    window.__qaCLS = { value: 0, largestSource: null };
    new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!entry.hadRecentInput) {
          window.__qaCLS.value += entry.value;
          if (entry.sources && entry.sources.length > 0) {
            const src = entry.sources[0];
            if (src.node) {
              window.__qaCLS.largestSource = src.node.tagName + (src.node.id ? '#' + src.node.id : '');
            }
          }
        }
      }
    }).observe({ type: 'layout-shift', buffered: true });

    // INP observer
    window.__qaINP = { value: 0, target: null };
    new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.duration > window.__qaINP.value) {
          window.__qaINP = {
            value: entry.duration,
            type: entry.name,
            target: entry.target ? entry.target.tagName : null
          };
        }
      }
    }).observe({ type: 'event', durationThreshold: 0, buffered: true });
  });
  return 'observers attached';
}"

// Step 2: Interact with page (triggers INP accumulation, helps LCP finalize)
playwright-cli click body

// Step 3: Finalize CLS by dispatching visibilitychange
playwright-cli run-code "async (page) => {
  await page.evaluate(() => {
    document.dispatchEvent(new Event('visibilitychange'));
  });
  return 'visibilitychange dispatched';
}"

// Step 4: Read all three metrics
playwright-cli eval "() => ({ lcp: window.__qaLCP, cls: window.__qaCLS, inp: window.__qaINP })"
```

**Source:** [CITED: web.dev/articles/debug-performance-in-the-field] + [CITED: smashingmagazine.com/2024/02/reporting-core-web-vitals-performance-api/] + [VERIFIED: established playwright-cli run-code pattern in qa-tester.md]

### Pattern 2: getComputedStyle Property Diff for Cross-Browser

```javascript
// Run in each engine — store result, then diff against Chromium baseline
playwright-cli eval "() => {
  const target = document.querySelector('body'); // or key element
  const style = getComputedStyle(target);
  return {
    fontFamily: style.fontFamily,
    fontWeight: style.fontWeight,
    lineHeight: style.lineHeight,
    letterSpacing: style.letterSpacing,
    gap: style.gap,
    borderRadius: style.borderRadius,
    position: style.position,
    display: style.display,
    overflow: style.overflow,
    overflowX: style.overflowX,
    transform: style.transform,
    opacity: style.opacity,
    boxShadow: style.boxShadow,
    backdropFilter: style.backdropFilter,
    webkitBackdropFilter: style.webkitBackdropFilter,
    scrollbarWidth: style.scrollbarWidth,
    textRendering: style.textRendering,
    fontSmoothing: style.webkitFontSmoothing
  };
}"
```

**Source:** [ASSUMED] — standard getComputedStyle properties; browser support confirmed via caniuse.com references

### Pattern 3: Viewport Overflow Check

```javascript
// Check at each breakpoint width
playwright-cli eval "() => {
  const docOverflow = document.documentElement.scrollWidth > window.innerWidth;
  const containers = Array.from(
    document.querySelectorAll('main, [role=\"main\"], section, article, .container, .wrapper')
  ).filter(el => el.offsetParent !== null);
  const overflowingContainers = containers.filter(el => el.scrollWidth > el.clientWidth);
  const textOverflow = Array.from(
    document.querySelectorAll('p, h1, h2, h3, h4, h5, h6, span, a, button, li, td, th, label')
  ).filter(el => el.offsetParent !== null && el.scrollWidth > el.clientWidth);
  return {
    documentOverflow: docOverflow,
    documentScrollWidth: document.documentElement.scrollWidth,
    windowInnerWidth: window.innerWidth,
    overflowingContainers: overflowingContainers.map(el => ({
      tag: el.tagName, id: el.id, class: String(el.className).slice(0, 40),
      scrollWidth: el.scrollWidth, clientWidth: el.clientWidth
    })),
    textOverflow: textOverflow.map(el => ({
      tag: el.tagName, text: el.textContent.trim().slice(0, 40),
      scrollWidth: el.scrollWidth, clientWidth: el.clientWidth
    }))
  };
}"
```

**Source:** [VERIFIED: extends overflow pattern from qa-tester.md DESIGN-02 and LAYOUT-01 — lines 1528]

### Pattern 4: Cookie Banner Overlap Detection

```javascript
playwright-cli eval "() => {
  const bannerSelectors = [
    '[id*=\"cookie\"]', '[class*=\"cookie\"]', '[id*=\"consent\"]', '[class*=\"consent\"]',
    '[id*=\"banner\"]', '[class*=\"gdpr\"]', '[class*=\"ccpa\"]',
    '[aria-label*=\"cookie\"]', '[role=\"dialog\"][aria-label*=\"cookie\"]'
  ];
  const mainSelectors = ['[role=\"main\"]', 'main', '#main', '#content', '.main-content'];
  const banner = bannerSelectors.map(s => document.querySelector(s)).find(Boolean);
  const main = mainSelectors.map(s => document.querySelector(s)).find(Boolean);
  if (!banner || !main) return { found: false, reason: !banner ? 'no banner found' : 'no main found' };
  const bRect = banner.getBoundingClientRect();
  const mRect = main.getBoundingClientRect();
  const overlaps = !(bRect.right <= mRect.left || bRect.left >= mRect.right ||
                     bRect.bottom <= mRect.top || bRect.top >= mRect.bottom);
  const bannerInViewport = bRect.top >= 0 && bRect.top < window.innerHeight;
  return {
    found: true, overlaps, bannerInViewport,
    banner: { top: bRect.top, bottom: bRect.bottom, height: bRect.height },
    main: { top: mRect.top, bottom: mRect.bottom }
  };
}"
```

**Source:** [ASSUMED] — derived from standard getBoundingClientRect intersection logic; [CITED: developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect]

### Pattern 5: Infinite Scroll Position Recovery Test

```javascript
// Step 1: Record scroll position before navigation
playwright-cli eval "() => window.scrollY"
// -> store as pre_nav_scroll

// Step 2: Scroll down to accumulate content
playwright-cli run-code "async (page) => {
  await page.evaluate(() => { window.scrollTo(0, document.body.scrollHeight * 0.5); });
  return 'scrolled to mid-page';
}"

// Step 3: Click a detail item (navigate forward)
playwright-cli click <list-item-link-ref>

// Step 4: Navigate back
playwright-cli go-back

// Step 5: Read restored position
playwright-cli eval "() => window.scrollY"
// -> compare to value from Step 2
// If scrollY ≈ 0 and pre-click scrollY was > 200: flag as High confidence — "infinite scroll: scroll position lost on back navigation"
```

**Source:** [ASSUMED] — derived from browser history.scrollRestoration behavior; [CITED: developer.mozilla.org/en-US/docs/Web/API/History/scrollRestoration]

### Pattern 6: Modal Close Mechanism Detection

```javascript
playwright-cli eval "() => {
  const dialogs = Array.from(document.querySelectorAll('[role=\"dialog\"], dialog, [class*=\"modal\"], [class*=\"overlay\"]'))
    .filter(el => el.offsetParent !== null); // only visible ones
  return dialogs.map(dialog => {
    const closeButtons = dialog.querySelectorAll(
      'button[aria-label*=\"close\" i], button[aria-label*=\"dismiss\" i], button[title*=\"close\" i], ' +
      '[class*=\"close\"], [class*=\"dismiss\"], [data-dismiss], [data-close], ' +
      'button:has(svg[aria-label*=\"close\" i])'
    );
    const hasEscapeHint = dialog.getAttribute('aria-modal') === 'true'; // modal dialogs should be closeable
    return {
      id: dialog.id || null,
      class: String(dialog.className).slice(0, 40),
      hasCloseButton: closeButtons.length > 0,
      closeButtonCount: closeButtons.length,
      hasEscapeHint
    };
  });
}"
```

**Source:** [ASSUMED] — derived from ARIA dialog pattern documentation

### Anti-Patterns to Avoid

- **Injecting web-vitals library from CDN:** D-01 explicitly prohibits external dependencies. Use raw PerformanceObserver only.
- **Reading LCP before body click:** The browser does not emit the final `largest-contentful-paint` entry until user input occurs. Reading immediately after navigation returns an incomplete or empty entry.
- **Reading CLS mid-session:** `layout-shift` entries accumulate over the full session. Reading without first triggering `visibilitychange` returns incomplete accumulated value.
- **Treating INP = 0 as "good":** Pages with no agent interaction should report INP as "not measured" per D-04, not score 0 (which would be misleading).
- **Flagging native form controls as cross-browser bugs:** `<select>`, `<input type="date">`, scrollbar appearance, and system font fallbacks are intentional per-engine differences per D-10.
- **Continuous sweep without prior discrete failure:** Run Phase 1 (6 widths) first. The continuous sweep (Phase 2) is an escalation path, not a default.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Web Vitals measurement | Custom timing logic | Raw PerformanceObserver (browser-native) | Already available in all Playwright engines; no CDN needed |
| Cross-browser launching | Engine-switching code in agent | Existing Browsers section + qa-run skill | Infrastructure already exists at skill lines 151-159 |
| Viewport resize | Custom JS viewport manipulation | `playwright-cli resize` | Existing command, tested, consistent |
| Element overlap detection | Custom geometry library | `getBoundingClientRect` intersection math | 4-line comparison, no library needed |
| Scroll position | Custom scroll tracking | `window.scrollY` + `window.scrollTo` via eval | Browser-native, works in all engines |

**Key insight:** Every capability this phase needs already exists in the browser's native APIs or in the established playwright-cli command set. The phase's job is to write methodology text that instructs the agent HOW to use these APIs, not to add new tooling.

---

## Common Pitfalls

### Pitfall 1: LCP Not Reporting
**What goes wrong:** `window.__qaLCP.value` remains 0 after navigation.
**Why it happens:** The browser only emits the final `largest-contentful-paint` entry after user input. If the agent reads the value before clicking the page, the observer has no finalized entry.
**How to avoid:** Always call `playwright-cli click body` (or equivalent interaction) before reading the LCP global. Per D-02, this is a required explicit step, not optional.
**Warning signs:** LCP value = 0 or equal to an early intermediate paint when the page clearly has a hero image.

### Pitfall 2: CLS Reading Incomplete Value
**What goes wrong:** CLS score appears lower than expected because layout shifts after initial load haven't been flushed.
**Why it happens:** The `layout-shift` PerformanceObserver accumulates entries throughout the session. The browser flushes the final batch when the page is hidden. Without triggering `visibilitychange`, the last batch of shifts is still buffered.
**How to avoid:** Per D-03, always dispatch `visibilitychange` via `run-code` before reading `window.__qaCLS.value`. Use `document.dispatchEvent(new Event('visibilitychange'))` — do NOT use `page.close()` as that ends the session.
**Warning signs:** CLS = 0 on a page that visibly has shifting content.

### Pitfall 3: Cross-Browser Comparison Without Baseline
**What goes wrong:** Agent flags every WebKit value as a "cross-browser difference" without knowing what the Chromium baseline actually is.
**Why it happens:** The PERF-02 check runs once per engine in separate test runs. The agent in the WebKit run doesn't have access to the Chromium results unless they were stored.
**How to avoid:** The cross-browser comparison is a WITHIN-AGENT-RUN activity when the agent receives both engine results in the same session (or via a prior run's report). For single-engine runs, the agent logs the computed style values and flags obvious non-standard rendering via visual screenshot judgment. Full property diffing requires the agent to have run in both engines and be reviewing both results.
**Warning signs:** Agent claiming cross-browser differences when only one engine has been tested.

### Pitfall 4: Breakpoint Sweep Screenshot Flood
**What goes wrong:** Agent takes a screenshot at every single pixel width during continuous sweep, producing 100+ screenshots.
**Why it happens:** Screenshot discipline from normal testing carried into sweep mode.
**How to avoid:** In Phase 1 (discrete), take screenshots only at failing widths. In Phase 2 (continuous), screenshot at the first failing width and at the failure boundary edge (last passing width, first failing width).
**Warning signs:** Report mentions 50+ screenshots from a single breakpoint sweep.

### Pitfall 5: INP Reported as "Good" on Static Pages
**What goes wrong:** Agent reports INP = 0ms or "Good" on a page that was loaded but never interacted with.
**Why it happens:** The `event` PerformanceObserver only fires for actual user interactions. No interactions = no entries = value stays at 0.
**How to avoid:** Per D-04, explicitly check `window.__qaINP.value === 0` and the absence of interaction history, then report "INP: not measured (no interactions recorded)" rather than a score.
**Warning signs:** Agent reporting "INP: 0ms — Good" on a page with no agent interactions recorded.

### Pitfall 6: Cookie Banner Detection False Negative
**What goes wrong:** Agent reports "no cookie banner found" when one exists.
**Why it happens:** Cookie banners use wildly inconsistent class/id naming. The selector list must be broad enough to catch non-standard implementations.
**How to avoid:** Use the broad selector list from Pattern 4 above. Supplement with a screenshot visual check — if the screenshot shows a blocking overlay the selectors missed, flag at Medium confidence.
**Warning signs:** Screenshot shows a full-width bottom bar but eval returns "no banner found."

---

## Core Web Vitals Thresholds (Claude's Discretion Item)

Official Google thresholds [CITED: web.dev/articles/vitals]:

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| INP | ≤ 200ms | 200ms – 500ms | > 500ms |
| CLS | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

**Lab-environment labeling requirement (D-05):** All findings must use "Lab" prefix: "Lab LCP: 3.2s (hero image `<img#hero>`)" not "LCP: 3.2s". This prevents treating synthetic test measurements as equivalent to CrUX field data.

---

## Cross-Browser Property List (Claude's Discretion Item)

Recommended ~18 properties for PERF-02 comparison [ASSUMED — derived from D-09 starting point and common browser rendering differences]:

| Property | Divergence Risk | Notes |
|----------|----------------|-------|
| `fontFamily` | Medium | System font fallbacks differ — exempt if using generic family |
| `fontWeight` | Low-Medium | Bold rendering can differ (700 vs bold keyword) |
| `lineHeight` | Medium | Sub-pixel rounding differs between engines |
| `letterSpacing` | Medium | WebKit adds extra spacing in some contexts |
| `gap` (flex/grid) | Low | Well-supported but worth checking |
| `scrollbarWidth` | HIGH — EXEMPT | Per D-10, intentional divergence — exempt |
| `borderRadius` | Low | Sub-pixel rendering differs slightly |
| `position` | Low | Check sticky threshold behavior specifically |
| `display` | Low | Flex/grid support is universal now |
| `overflowX` / `overflowY` | Medium | Value resolution can differ |
| `transform` | Low-Medium | 3D transform rendering can differ |
| `opacity` | Low | Sub-pixel compositing |
| `boxShadow` | Medium | Color value format differs (rgb vs hex) |
| `backdropFilter` | HIGH | WebKit requires `-webkit-backdrop-filter`; Firefox partial support |
| `-webkit-font-smoothing` | MEDIUM — EXEMPT | WebKit-only property, intentional |
| `textRendering` | Medium | geometricPrecision vs auto behavior |
| `alignItems` | Low | Check for "normal" vs "stretch" resolution |
| `justifyContent` | Low | Check for "normal" vs "flex-start" resolution |

**Exemption list to suppress:** native `<select>`, `<input type="date">`, `<input type="range">`, scrollbar-related properties, `-webkit-` prefixed vendor properties, system font fallback names.

---

## UX Anti-Pattern Detection Specifics (Claude's Discretion Items)

### Marketing/Consent Context for Pre-Checked Opt-ins

The challenge is distinguishing "pre-checked opt-in" (dark pattern) from "pre-checked acceptance" (normal form behavior like "remember me"). Recommended heuristic for detecting marketing/consent context [ASSUMED]:

1. Check if checkbox is inside a container with consent-related text: `[class*="consent"]`, `[class*="gdpr"]`, `[class*="cookie"]`, `[class*="subscribe"]`, `[class*="marketing"]`, `[class*="newsletter"]`
2. Check if the checkbox label text matches patterns: `/newsletter|marketing email|promotional|updates|offers|deals|third.party|partner/i`
3. Any pre-checked checkbox matching either heuristic = High confidence finding

Pre-checked checkboxes NOT in consent context (e.g., "remember me", "save card", "agree to terms of service" where checking is required) are out of scope.

### Infinite Scroll vs Paginated Lists

The infinite scroll position recovery test only applies when the page demonstrably uses infinite scroll (content expands on scroll, no traditional pagination). Detection heuristic: scroll to 80% of page height — if `document.body.scrollHeight` increases after a brief wait, infinite scroll is active. If no height increase, skip the position recovery test and note "no infinite scroll detected."

### Section Insertion Point

The new `### Performance & Responsive` section inserts at **line 1803** (immediately before `### Design Reference`), after the Layout & Content Integrity confidence table ends. The "pending Phase 12" note at **line 747** is removed as part of the same edit.

The Confidence Summary Table at **lines 355-375** is extended with PERF-* entries after the existing accessibility entries.

---

## Runtime State Inventory

Not applicable — this is a greenfield methodology addition (new text in qa-tester.md). No rename, refactor, or migration.

---

## Environment Availability

Step 2.6 result: No external dependencies. All browser APIs (PerformanceObserver, getComputedStyle, getBoundingClientRect, window.scrollY) are available in all Playwright browser engines by default. The `playwright-cli` tool and its `resize`, `run-code`, `eval`, `screenshot`, `navigate`, `go-back`, and `click` commands are all documented as existing in qa-tester.md.

| Dependency | Required By | Available | Notes |
|------------|------------|-----------|-------|
| PerformanceObserver | PERF-01 | All engines | Browser-native API |
| `largest-contentful-paint` observer type | PERF-01 LCP | Chromium, WebKit | Not available in Firefox — note in methodology |
| `layout-shift` observer type | PERF-01 CLS | Chromium, WebKit | Limited Firefox support |
| `event` observer type (INP) | PERF-01 INP | Chromium, WebKit, Firefox 144+ | Firefox 144 added interactionId support (Oct 2025) |
| `getComputedStyle` | PERF-02 | All engines | Universal |
| `getBoundingClientRect` | PERF-03, PERF-04 | All engines | Universal |
| `window.scrollY` | PERF-04 | All engines | Universal |
| `playwright-cli resize` | PERF-03 | Available | Documented at agent line 60 |

**Firefox caveat:** `largest-contentful-paint` and `layout-shift` PerformanceObserver types are not supported in Firefox. The methodology should note: "CLS and LCP are not measurable in Firefox — PERF-01 runs in Chromium and WebKit only. INP is measurable in Firefox 144+ via `event` type."

---

## Validation Architecture

`workflow.nyquist_validation` key is absent from `.planning/config.json` — treating as enabled.

This phase is documentation-only (writes methodology text to qa-tester.md). There is no application code to unit test. Validation is by manual inspection and a smoke test.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | Notes |
|--------|----------|-----------|-------------------|-------|
| PERF-01 | CLS/LCP/INP measurement with element attribution | Manual smoke test | Run qa:run against a test page with performance-responsive in Visual Focus | Verify report contains "Lab LCP:", "Lab CLS:", "Lab INP:" labels |
| PERF-02 | Cross-browser rendering comparison report | Manual smoke test | Run qa:run with `## Browsers: chromium, webkit` and `performance-responsive` | Verify report contains engine-tagged CSS property diff |
| PERF-03 | Breakpoint sweep catches horizontal overflow | Manual smoke test | Run qa:run against a known-overflow page with performance-responsive | Verify overflow detected at correct width |
| PERF-04 | UX anti-pattern detection | Manual smoke test | Run qa:run against page with a visible modal | Verify modal close check runs and reports |

No automated test files needed — the deliverable is agent methodology text, not application logic.

**Wave 0 gaps:** None — no test framework required for this phase.

---

## Security Domain

Phase 12 adds methodology text only. No new data flows, no authentication, no user data handling. The existing security note from Design Verification applies: "When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports." This note should be included in the Performance & Responsive section for consistency with Phases 9-11.

| ASVS Category | Applies | Note |
|---------------|---------|------|
| V2 Authentication | No | No auth logic in this phase |
| V3 Session Management | No | No session handling |
| V4 Access Control | No | No access control |
| V5 Input Validation | No | No user input |
| V6 Cryptography | No | No cryptographic operations |

---

## Open Questions (RESOLVED)

1. **PerformanceObserver buffered: true on already-loaded pages**
   - What we know: `buffered: true` replays already-collected entries when a new observer subscribes. This means the observer can be injected after page load and still see prior LCP/CLS entries.
   - What's unclear: Whether `playwright-cli run-code` reliably executes before or after the `load` event depending on when in the navigation flow it's called. If called too early (pre-load), the global might not survive navigation.
   - RESOLVED: Call the observer injection step immediately after the page URL has fully loaded (after `playwright-cli navigate` completes). Use `buffered: true` as the safety net. The methodology notes: "Inject observers after page load completes."

2. **WebKit support for `largest-contentful-paint`**
   - What we know: The `largest-contentful-paint` PerformanceObserver type is available in Chromium. WebKit (Safari) historically lagged on this.
   - What's unclear: Current WebKit support status in Playwright's bundled WebKit engine (which tracks upstream WebKit, not necessarily the same as Safari release).
   - RESOLVED: The methodology includes a try/catch wrapper: if `largest-contentful-paint` observation throws, note "LCP not supported in this engine — skip" rather than failing the whole check. This gracefully handles both supported and unsupported engines.

3. **Cookie banner detection on SPAs with deferred banner injection**
   - What we know: Many cookie banners are injected dynamically 1-3 seconds after page load via JavaScript.
   - What's unclear: Whether the agent should add a wait step before running the cookie banner check.
   - RESOLVED: Include a brief wait instruction (e.g., `playwright-cli run-code "async (page) => { await page.waitForTimeout(2000); }"`) before the cookie banner eval, explicitly for pages suspected of having a deferred consent banner.

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | Global accumulator pattern (`window.__qa*`) persists through page interactions without being cleared by app JS | Code Examples / Pattern 1 | If app clears window globals (unlikely but possible), observers lose accumulated data. Mitigation: use non-conflicting namespace like `window.__qaPerf*` |
| A2 | `document.dispatchEvent(new Event('visibilitychange'))` causes the CLS PerformanceObserver to flush its buffered entries | Code Examples / Pattern 1 | If the browser requires actual page hide (not synthetic event) to flush CLS, reading after synthetic event still misses the last batch. Fallback: use `observer.takeRecords()` directly before reading |
| A3 | 18-property list covers the highest-risk cross-browser rendering differences for Chromium vs WebKit | Cross-Browser Property List | Agent may miss important divergences or waste time on noise properties. Low impact — planner should note this list is Claude's discretion, can be adjusted |
| A4 | `input[type="checkbox"][checked]` attribute selector catches pre-checked checkboxes | UX Anti-Pattern / Pattern section | HTML attribute `checked` (default state) vs JS `.checked` property differ — an app might omit the HTML attribute but use JS to set checked state. Mitigation: supplement with `:checked` pseudo-class eval |
| A5 | `playwright-cli go-back` after infinite scroll navigation lands on same page at scroll position 0 (position lost) | Code Examples / Pattern 5 | If the SPA uses custom scroll restoration and DOES restore position correctly, the test should PASS. Methodology must compare actual scrollY vs pre-navigation scrollY, not assume failure |
| A6 | Firefox's `largest-contentful-paint` PerformanceObserver type is unsupported | Environment Availability | Firefox may have added support after August 2025 training cutoff. Methodology's try/catch wrapper handles this gracefully either way |

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| FID (First Input Delay) as Core Web Vital | INP (Interaction to Next Paint) replaced FID | March 2024 | INP measures full interaction latency; agent uses `event` type observer |
| LCP finalization required page close | LCP finalizes on user input (body click) | ~2022, stable | Agent can read LCP in same session without closing page |
| Web Vitals only measurable via web-vitals npm library | Raw PerformanceObserver is now sufficient for lab values | 2022+ | D-01 decision to avoid external dependencies is fully viable |

---

## Sources

### Primary (HIGH confidence)
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — Official CWV thresholds (LCP ≤2.5s, INP ≤200ms, CLS ≤0.1)
- [web.dev/articles/debug-performance-in-the-field](https://web.dev/articles/debug-performance-in-the-field) — PerformanceObserver attribution patterns (entry.element, entry.sources)
- [developer.mozilla.org/en-US/docs/Web/API/History/scrollRestoration](https://developer.mozilla.org/en-US/docs/Web/API/History/scrollRestoration) — scroll restoration API; SPA position loss pattern
- [developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect) — intersection detection pattern

### Secondary (MEDIUM confidence)
- [smashingmagazine.com/2024/02/reporting-core-web-vitals-performance-api/](https://www.smashingmagazine.com/2024/02/reporting-core-web-vitals-performance-api/) — Raw PerformanceObserver code patterns for LCP/CLS/INP
- [checklyhq.com/docs/learn/playwright/performance/](https://www.checklyhq.com/docs/learn/playwright/performance/) — Playwright-specific Web Vitals patterns
- [github.com/GoogleChrome/web-vitals/issues/180](https://github.com/GoogleChrome/web-vitals/issues/180) — Lab environment limitations; body click for LCP finalization; visibilitychange for CLS
- [gist.github.com/samarpanda/a5f561fc6af83883494f03544fba4664](https://gist.github.com/samarpanda/a5f561fc6af83883494f03544fba4664) — CLS visibilitychange + takeRecords pattern
- [blog.mozilla.org/performance/2025/10/15/firefox-144-ships-interactionid-for-inp/](https://blog.mozilla.org/performance/2025/10/15/firefox-144-ships-interactionid-for-inp/) — Firefox 144 INP/interactionId support

### Tertiary (LOW confidence — flagged in Assumptions Log)
- getComputedStyle cross-browser property differences — synthesized from lambdatest.com and caniuse.com results; specific property divergence list is [ASSUMED]

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new dependencies; all tools verified in existing qa-tester.md
- Architecture patterns: HIGH — extends established dual pattern; insertion point confirmed by line inspection
- PerformanceObserver JS patterns: MEDIUM — official web.dev patterns confirmed; global accumulator approach is standard Playwright workaround
- Cross-browser property list: LOW-MEDIUM — starting list from context decisions; specific divergences are assumed
- UX anti-pattern detection logic: MEDIUM — selector/intersection patterns are standard; consent-context heuristic is assumed

**Research date:** 2026-04-20
**Valid until:** 2026-07-20 (stable APIs — 90 days)
