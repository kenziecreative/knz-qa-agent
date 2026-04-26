# Phase 9: Design Verification - Research

**Researched:** 2026-04-19
**Domain:** Visual design verification methodology for QA agent
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Mockup Comparison (DESIGN-01)**
- D-01: Structured category checklist — agent scans reference vs live screenshot using fixed categories in order: typography, spacing, color, imagery, layout.
- D-02: Each category produces findings with confidence level (High/Medium/Low) and "live page shows X vs reference shows Y" format.
- D-03: Root cause diagnosis is narrative-driven — agent describes what's different and suggests why (layout shift, wrong color token, missing element, etc.).

**Typography Verification (DESIGN-02)**
- D-04: Dual approach — `eval()` JS probes for verifiable assertions (font loaded via `document.fonts.check()`, text overflow via `scrollWidth > clientWidth`, computed font family) PLUS visual judgment from screenshots for cross-page consistency and readability.
- D-05: When spec or design tokens file declares expected font families/sizes, agent uses those as baseline. When no expected values exist, agent infers consistency by comparing computed styles across pages.
- D-06: Font load detection uses `document.fonts.check('16px FontName')` and `document.fonts.ready`.

**Image & Media Quality (DESIGN-03)**
- D-07: Evaluate-first, screenshot fallback — JS probes (`naturalWidth`/`naturalHeight` vs rendered dimensions, `complete` property, `currentSrc`, lazy-load `src`/`data-src`) always run as Tier 1. Screenshot supplements when anomalies found or Tier 2 opted in.
- D-08: Aspect ratio distortion: compare `naturalWidth/naturalHeight` ratio vs rendered `width/height` ratio — threshold-based (>5% deviation = flag).
- D-09: Upscale detection: `naturalWidth < renderedWidth` or `naturalHeight < renderedHeight` = image displayed larger than its source resolution.

**Color & Theme Consistency (DESIGN-04)**
- D-10: Dual approach — `eval()` reads CSS custom properties and computed styles for programmatic token consistency. Visual judgment from screenshots covers holistic rendering (especially after dark mode toggle).
- D-11: Hover/focus color states verified by sequencing playwright actions (hover, focus) THEN reading computed styles via eval — computed styles only reflect current interaction state.
- D-12: Dark mode testing: agent toggles color scheme (via `eval` to set `prefers-color-scheme` media override OR clicking a theme toggle if present) then re-reads CSS variables and takes comparison screenshots.

**Cross-Page Consistency (DESIGN-05)**
- D-13: When no formal design system/tokens exist, agent infers consistency by sampling computed styles from key elements across multiple pages (headers, buttons, links, body text) and flagging deviations.
- D-14: When design tokens (CSS custom properties) exist, agent reads them and verifies consistent usage across pages.

**Overarching Pattern**
- D-15: All five areas follow the same dual pattern: `eval()` for deterministic/measurable checks + visual judgment for holistic/perceptual quality. Maps to existing Tier 1/Tier 2 infrastructure.

### Claude's Discretion
- Exact JS eval snippets and DOM queries for each check
- Threshold values for aspect ratio distortion and upscale detection
- How to structure the methodology sections in qa-tester.md (section naming, ordering within existing structure)
- Whether to add helper eval scripts or inline them in methodology descriptions
- Cross-page consistency sampling strategy (which elements, how many pages)

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DESIGN-01 | Agent compares live page against design reference images/HTML layouts and reports/fixes visual deviations with root cause diagnosis | Structured category checklist (D-01), narrative root cause (D-03), playwright-cli eval + screenshot patterns |
| DESIGN-02 | Agent verifies font loading, font consistency across pages, text overflow handling, and line length readability | document.fonts.check() (D-06), scrollWidth/clientWidth overflow detection (D-04), computed font-family probing |
| DESIGN-03 | Agent detects broken images, aspect ratio distortion, missing lazy-load placeholders, and upscaled images | naturalWidth/naturalHeight vs rendered dimension probing (D-07, D-08, D-09), image.complete property |
| DESIGN-04 | Agent verifies color token usage, hover/focus color states, and dark mode/theme switching consistency | CSS custom property reading (D-10), hover-then-eval pattern (D-11), emulateMedia dark mode (D-12) |
| DESIGN-05 | Agent enforces design system consistency or infers cross-page consistency | Computed style sampling across pages (D-13), CSS variable enumeration (D-14) |
</phase_requirements>

---

## Summary

Phase 9 fills the methodology stub that Phase 8 left in place. The infrastructure (spec sections, forwarding blocks, Visual Focus Trigger, Design Reference handler) is already live in `agents/qa-tester.md` and `skills/run/SKILL.md`. This phase writes the actual Tier 2 `design-verification` procedure the agent follows when triggered.

All five DESIGN requirements converge on one architectural pattern: `playwright-cli eval` for deterministic DOM/CSS probes (verifiable, high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (holistic, medium confidence). This dual pattern is already established in the accessibility Tier 2 sections — Phase 9 mirrors that structure.

The only deliverable is editing `agents/qa-tester.md`: adding a `### Design Verification` section (and its five subsections) under the `### Visual Focus Trigger` section, removing two placeholder phrases that say "methodology pending Phase 9," and extending the Confidence Summary Table with design verification entries.

**Primary recommendation:** Mirror the Structured Accessibility section format exactly — numbered procedure steps, explicit confidence level per finding type, common patterns to flag. One subsection per DESIGN requirement area. Use `playwright-cli eval` for JS probes and `playwright-cli run-code` for multi-step Playwright sequences (hover-then-read, dark mode toggle-then-read) that require page object access rather than a single evaluate expression.

---

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Design reference comparison | Agent (qa-tester.md) | — | Pure methodology — agent reads image paths from forwarded Design Reference block, screenshots live page, applies category checklist, visual judgment |
| Typography probing | Agent (qa-tester.md) | — | JS eval probes (font loaded, overflow, computed family) plus cross-page visual comparison — all within a single test session |
| Image quality probing | Agent (qa-tester.md) | — | JS eval against DOM (naturalWidth, complete, currentSrc) — no external tooling needed |
| Color/token verification | Agent (qa-tester.md) | — | CSS custom property reading via eval, hover/focus state sequencing, emulateMedia for dark mode |
| Cross-page consistency | Agent (qa-tester.md) | — | Agent navigates pages sequentially within session, sampling computed styles and comparing — no external tool |
| Dark mode emulation | playwright-cli run-code | Agent judgment | `page.emulateMedia()` requires page object — use `run-code` with an inline function; agent reads CSS variables before and after |

---

## Standard Stack

### Core
| Tool | Version | Purpose | Why Standard |
|------|---------|---------|--------------|
| playwright-cli | 0.1.8 [VERIFIED: installed] | Browser control, eval, screenshot, hover, run-code | Already the exclusive browser automation tool in this project — no alternatives considered |

### Supporting
| API | Source | Purpose | When to Use |
|-----|--------|---------|-------------|
| `document.fonts.check()` | Web Fonts API [CITED: MDN] | Detect whether a specific font has loaded | DESIGN-02 font load verification |
| `document.fonts.ready` | Web Fonts API [CITED: MDN] | Promise resolves when all fonts have loaded | DESIGN-02 — await before checking font state |
| `element.scrollWidth > element.clientWidth` | DOM [CITED: MDN] | Detect horizontal text overflow | DESIGN-02 overflow check |
| `getComputedStyle(el).fontFamily` | DOM CSSOM | Read actual rendered font family | DESIGN-02 cross-page font consistency |
| `img.naturalWidth / img.naturalHeight` | DOM [CITED: MDN] | Intrinsic image dimensions for distortion/upscale detection | DESIGN-03 |
| `img.complete` | DOM [CITED: MDN] | Detect broken/unloaded images | DESIGN-03 |
| `img.currentSrc` | DOM [CITED: MDN] | Detect which responsive image source loaded | DESIGN-03 lazy-load verification |
| `getComputedStyle(el).getPropertyValue('--token-name')` | CSSOM | Read CSS custom property value | DESIGN-04 token verification |
| `page.emulateMedia({ colorScheme: 'dark' })` | Playwright [VERIFIED: existing qa-tester.md pattern] | Simulate dark mode preference | DESIGN-04 dark mode testing |

### No Additional Packages Required
This phase adds zero npm dependencies. All capability comes from playwright-cli (already installed) and standard browser APIs available in every modern browser. [VERIFIED: playwright-cli 0.1.8 confirmed installed at /Users/kelseyruger/.volta/bin/playwright-cli]

---

## Architecture Patterns

### System Architecture Diagram

```
Spec has ## Visual Focus: design-verification
           |
           v
Visual Focus Trigger (qa-tester.md line 733)
  -- "design-verification" in areas list?
           |
          YES
           |
           v
Design Verification methodology (Phase 9 addition)
    |           |           |           |           |
    v           v           v           v           v
DESIGN-01   DESIGN-02   DESIGN-03   DESIGN-04   DESIGN-05
Mockup      Typography  Image/Media  Color/Theme  Cross-Page
Comparison  Checks      Checks       Checks       Consistency
    |           |           |           |               |
    v           v           v           v               v
eval probes + screenshot + visual judgment (per category)
    |
    v
Findings: confidence-grouped, "live shows X vs reference shows Y"
    |
    v
Confidence Summary Table entries (design verification rows)
```

### Recommended Section Structure in qa-tester.md

The new content goes immediately after line 747 (end of Visual Focus Trigger section), before line 749 (Design Reference section). The Design Reference section currently says "actual comparison methodology is implemented in Phase 9" — that placeholder is removed, replaced by a pointer to the methodology above it.

```
## [existing] Visual Focus Trigger  (lines 733-747)
  -- Phase 9 removes the "pending Phase N" clause

### Design Verification  (NEW — Phase 9)
  #### Mockup Comparison (DESIGN-01)
  #### Typography Verification (DESIGN-02)
  #### Image & Media Quality (DESIGN-03)
  #### Color & Theme Consistency (DESIGN-04)
  #### Cross-Page Consistency (DESIGN-05)
  #### Design Verification Confidence Table

## [existing] Design Reference  (lines 749-763)
  -- Phase 9 removes the "implemented in Phase 9" placeholder phrase
```

### Pattern 1: eval-first, screenshot-supplement

The canonical dual-mode probe pattern (decision D-15):

```javascript
// Step 1: deterministic probe via playwright-cli eval
// playwright-cli eval "() => { return getComputedStyle(document.querySelector('h1')).fontFamily }"
// => returns actual rendered font-family string

// Step 2: visual assessment via playwright-cli screenshot
// playwright-cli screenshot --filename .qa/reports/screenshot-fonts.png
// => agent reads image and judges readability, consistency

// Confidence assignment:
// - eval result differs from expected: High confidence (deterministic)
// - visual judgment shows inconsistency: Medium confidence (perceptual)
```

[VERIFIED: eval command confirmed via `playwright-cli --help eval`]

### Pattern 2: Hover-then-eval for interaction states

For DESIGN-04 hover/focus color verification (decision D-11):

```javascript
// Step 1: hover via playwright-cli
// playwright-cli hover <element-ref>

// Step 2: immediately read computed styles
// playwright-cli eval "() => getComputedStyle(document.querySelector('button')).backgroundColor"

// Step 3: screenshot to capture visual state
// playwright-cli screenshot --filename .qa/reports/hover-state.png

// NOTE: computed styles reflect current interaction state immediately after hover.
// Must eval in the same Playwright session without navigating away.
```

[VERIFIED: playwright-cli hover command confirmed]

### Pattern 3: Dark mode toggle via run-code

For DESIGN-04 dark mode testing (decision D-12), `page.emulateMedia()` requires page object access — use `run-code` rather than `eval`:

```javascript
// playwright-cli run-code "async (page) => {
//   await page.emulateMedia({ colorScheme: 'dark' });
//   const bg = await page.evaluate(() => getComputedStyle(document.documentElement).getPropertyValue('--color-background'));
//   return bg;
// }"
```

The existing `## Reduced Motion (A11Y-12)` section in qa-tester.md already uses `page.emulateMedia({ reducedMotion: 'reduce' })` — this is exactly the same pattern, applied to `colorScheme`. [VERIFIED: line 290 of qa-tester.md]

### Pattern 4: Cross-page computed style sampling

For DESIGN-05, agent navigates pages sequentially and accumulates style samples:

```
Page A → snapshot → eval (sample key elements: h1, button, body) → store values
Page B → snapshot → eval (same elements) → compare to Page A values
Page C → same pattern
Flag deviations > threshold as Medium confidence (visual judgment confirms)
```

Elements to sample (Claude's discretion per context): `h1`, `button`, `a`, `body`, `nav`. Enough to establish a pattern without exhausting every element.

### Pattern 5: Structured category checklist (DESIGN-01)

When a Design Reference image is present, the agent scans in fixed order:

1. **Typography** — font families, weights, sizes match reference
2. **Spacing** — padding, margins, gaps — especially balanced spacing on all sides (not just "is content clipped?" but "is bottom padding equal to top padding?")
3. **Color** — background colors, text colors, border colors
4. **Imagery** — images present, correct aspect ratios, correct positions
5. **Layout** — element placement, stacking order, alignment

Each category: "live page shows X vs reference shows Y" — or "matches reference" if correct. Root cause hypothesis for each deviation.

### Anti-Patterns to Avoid

- **Holistic "look and report":** Don't let the agent do free-form visual commentary. Structured categories prevent the spacing-flush-against-container miss (the specific failure from user session transcript — button had no bottom padding, agent only checked for content clipping).
- **eval without hover for interaction states:** Reading `computedStyle` before triggering hover returns the default state, not the hover state. Always sequence: hover first, eval second.
- **page.evaluate() framing in methodology:** The agent uses `playwright-cli eval`, not `page.evaluate()` directly. Write all methodology examples using the CLI command, not the Playwright API — agents get confused by API framing when they only have CLI access.
- **Blocking on missing design reference:** If no `## Design Reference` section is present, DESIGN-01 comparison cannot run. Skip it gracefully and note "no design reference provided — mockup comparison skipped" in the report. The other four areas (typography, images, color, cross-page) run regardless.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Dark mode simulation | CSS class toggle scripting | `page.emulateMedia({ colorScheme: 'dark' })` | emulateMedia is the authoritative way to set `prefers-color-scheme` in Playwright — matches real browser behavior |
| Image dimension probing | Canvas pixel inspection | `img.naturalWidth`, `img.naturalHeight`, `img.complete` | DOM properties are synchronous, accurate, and don't require rendering pipeline access |
| Font load detection | setTimeout polling | `document.fonts.check()` and `document.fonts.ready` | Web Font Loading API is designed for this — no polling needed |
| Pixel-perfect diffing | External image comparison library | Visual judgment (scoped to category checklist) | Pixel diff is explicitly Out of Scope in REQUIREMENTS.md — visual judgment is the intended approach |

**Key insight:** All design verification probes are available via standard DOM APIs and the existing `playwright-cli eval`/`run-code` commands. No new tools or libraries are needed.

---

## Common Pitfalls

### Pitfall 1: Spacing "Clipping" vs. "Balance" Confusion
**What goes wrong:** Agent checks whether content is clipped (overflows container) but misses that spacing is unbalanced — e.g., button has top padding but no bottom padding. Both are spacing issues but clipping checks miss the second.
**Why it happens:** Clipping is detectable via `scrollWidth > clientWidth`; unbalanced spacing is perceptual. The user's specific session transcript failure was exactly this case.
**How to avoid:** The typography/spacing category in DESIGN-01 must explicitly include "check balanced spacing on all sides" as a category requirement, not just "no clipping." Visual judgment from screenshot is the primary tool for balance detection.
**Warning signs:** Reports that say "spacing OK — no overflow detected" without also confirming equal padding on all sides.

### Pitfall 2: Computed Styles Require Hover/Focus State
**What goes wrong:** Agent reads `getComputedStyle(button).backgroundColor` without triggering hover first — gets the default state, reports hover color as "matches default" (incorrect).
**Why it happens:** `getComputedStyle` is snapshot-based — it reads the element in its current interaction state. If hover isn't active, hover styles are not returned.
**How to avoid:** For any hover/focus color check: (1) playwright-cli hover, (2) immediately playwright-cli eval. Do not navigate between steps.
**Warning signs:** Hover color findings that match default color with no noted difference.

### Pitfall 3: `playwright-cli eval` vs `playwright-cli run-code` Confusion
**What goes wrong:** Agent tries to use `eval` for multi-step Playwright operations (like `page.emulateMedia()`) that require the page object. `eval` only executes JS in the browser context — it can't call Playwright APIs.
**Why it happens:** The two commands look similar but do different things: `eval` runs JS in the page, `run-code` runs a Playwright function with page as argument.
**How to avoid:** Methodology should use `run-code` for any operation that needs Playwright API (emulateMedia, route, resize), and `eval` for operations that need DOM API (getComputedStyle, naturalWidth, document.fonts).
**Warning signs:** `page.emulateMedia is not defined` or similar errors in eval results.

### Pitfall 4: DESIGN-01 Blocked by Absent Design Reference
**What goes wrong:** Agent tries to run mockup comparison when no `## Design Reference` section is in the spec — no image paths available.
**Why it happens:** Design Reference is optional (fully backward compatible per Phase 8 decisions).
**How to avoid:** Methodology must start with: "If no Design Reference section was forwarded, skip DESIGN-01 and note 'no design reference provided — mockup comparison skipped.' Continue to DESIGN-02 through DESIGN-05."
**Warning signs:** Agent errors or confusion when Design Reference block is absent in the forwarded test assignment.

### Pitfall 5: Cross-Page Sampling with No Baseline
**What goes wrong:** On a single-page spec (only one URL), DESIGN-05 has nothing to compare — agent can't detect cross-page inconsistency without multiple pages.
**Why it happens:** Cross-page consistency requires at least two pages.
**How to avoid:** If spec only covers one URL, note "cross-page consistency check requires multiple pages — sampling on this page for future comparison" and record the sampled values. Don't error or skip entirely.
**Warning signs:** Agent confusion or silence on DESIGN-05 for single-page specs.

---

## Code Examples

Verified patterns from playwright-cli and browser APIs:

### Font Load Detection (DESIGN-02)
```javascript
// playwright-cli eval "async () => {
//   await document.fonts.ready;
//   const loaded = document.fonts.check('16px Inter');
//   const family = getComputedStyle(document.body).fontFamily;
//   return { loaded, family };
// }"
// Source: Web Fonts API (document.fonts.check is part of CSS Font Loading API)
// Confidence: High — deterministic API, not visual judgment
```

### Text Overflow Detection (DESIGN-02)
```javascript
// playwright-cli eval "() => {
//   const overflowing = [];
//   document.querySelectorAll('p, h1, h2, h3, h4, span, li').forEach(el => {
//     if (el.scrollWidth > el.clientWidth) {
//       overflowing.push({ tag: el.tagName, text: el.textContent.slice(0, 50) });
//     }
//   });
//   return overflowing;
// }"
// Source: DOM standard (scrollWidth/clientWidth)
// Confidence: High — deterministic
```

### Image Quality Probe (DESIGN-03)
```javascript
// playwright-cli eval "() => {
//   return Array.from(document.querySelectorAll('img')).map(img => ({
//     src: img.currentSrc || img.src,
//     complete: img.complete,
//     naturalW: img.naturalWidth,
//     naturalH: img.naturalHeight,
//     renderedW: img.getBoundingClientRect().width,
//     renderedH: img.getBoundingClientRect().height,
//     // broken if complete but naturalWidth === 0
//     broken: img.complete && img.naturalWidth === 0,
//     // distortion: ratio difference > 0.05
//     distorted: img.naturalWidth > 0 && Math.abs(
//       (img.naturalWidth / img.naturalHeight) -
//       (img.getBoundingClientRect().width / img.getBoundingClientRect().height)
//     ) > 0.05,
//     // upscaled: rendered larger than natural size
//     upscaled: img.naturalWidth > 0 && (
//       img.getBoundingClientRect().width > img.naturalWidth ||
//       img.getBoundingClientRect().height > img.naturalHeight
//     )
//   }));
// }"
// Source: DOM standard (HTMLImageElement)
// Confidence: High for broken/upscaled (deterministic); Medium for distortion (threshold-based)
```

### CSS Token Reading (DESIGN-04)
```javascript
// playwright-cli eval "() => {
//   const style = getComputedStyle(document.documentElement);
//   // Read all --color-* custom properties
//   return Array.from(document.styleSheets)
//     .flatMap(sheet => { try { return Array.from(sheet.cssRules); } catch(e) { return []; } })
//     .filter(rule => rule.selectorText === ':root')
//     .flatMap(rule => Array.from(rule.style))
//     .filter(prop => prop.startsWith('--color') || prop.startsWith('--bg'))
//     .map(prop => ({ prop, value: style.getPropertyValue(prop).trim() }));
// }"
// Source: CSSOM (CSSStyleSheet, CSSRule)
// Note: cross-origin stylesheets will throw — wrap in try/catch (already done above)
```

### Dark Mode Toggle (DESIGN-04)
```javascript
// playwright-cli run-code "async (page) => {
//   // Take light mode screenshot first
//   await page.screenshot({ path: '.qa/reports/light-mode.png' });
//   // Switch to dark mode
//   await page.emulateMedia({ colorScheme: 'dark' });
//   // Read CSS variables in dark mode
//   const darkBg = await page.evaluate(() =>
//     getComputedStyle(document.documentElement).getPropertyValue('--color-background').trim()
//   );
//   // Take dark mode screenshot
//   await page.screenshot({ path: '.qa/reports/dark-mode.png' });
//   return { darkBg };
// }"
// Source: Playwright API (page.emulateMedia) — confirmed via existing qa-tester.md line 290 pattern
// Note: use run-code (not eval) because emulateMedia is a Playwright API, not a browser API
```

### Hover State Color Check (DESIGN-04)
```javascript
// Step 1: playwright-cli hover <element-ref>
// Step 2 (immediately after):
// playwright-cli eval "() => ({
//   bg: getComputedStyle(document.activeElement).backgroundColor,
//   color: getComputedStyle(document.activeElement).color,
//   border: getComputedStyle(document.activeElement).borderColor
// })"
// Source: DOM CSSOM (getComputedStyle reflects current interaction state)
// Confidence: High — deterministic after hover is triggered
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Pixel-perfect image diffing (VIS-01) | Visual judgment + structured category checklist | Phase 9 design decision | Simpler, no external tooling — deferred to future VIS-01 requirement |
| Free-form "look and comment" visual review | Structured five-category checklist (typography, spacing, color, imagery, layout) | Phase 9 | Prevents the specific flush-spacing miss from user session transcript |

**Explicitly out of scope (REQUIREMENTS.md):**
- Pixel-level diff comparison using image hashing (VIS-01 — future)
- Design token extraction from Figma API (VIS-02 — future)
- Automated visual regression baseline management (VIS-03 — future)
- Color contrast ratio calculation (requires axe-core math)

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `playwright-cli run-code` accepts an async function with `page` parameter and can call `page.emulateMedia()` | Code Examples | Dark mode testing would need different approach — use `playwright-cli eval` with a CSS class toggle as fallback |
| A2 | `document.fonts.check('16px FontName')` reliably detects whether the named font is loaded vs. a fallback | Code Examples | Font load detection would need a different probe (compare computed fontFamily before/after fonts.ready) |
| A3 | Cross-origin stylesheet rules throw when accessed via CSSOM (requiring try/catch) | Code Examples | If hosting same-origin, the try/catch is unnecessary overhead — but safe to include regardless |

**Low risk overall:** A1 is based on the existing `page.emulateMedia` pattern already in qa-tester.md line 290, which uses `run-code`-style Playwright code. A2 is well-established Web Fonts API behavior. A3 is CORS standard behavior.

---

## Open Questions

1. **Aspect ratio distortion threshold — what's the right value?**
   - What we know: D-08 specifies ">5% deviation = flag" as the starting point
   - What's unclear: 5% may be too sensitive for responsive images using `object-fit: cover` (intentionally cropped), or too lenient for strict pixel-perfect designs
   - Recommendation: Use 5% as the default; add a note in the methodology that specs can explicitly note "aspect ratio distortion expected — responsive images use object-fit" to suppress this finding

2. **Line length readability check for DESIGN-02 — how to implement?**
   - What we know: DESIGN-02 mentions "line length readability" — this typically means ~45-75 characters per line (WCAG 1.4.8 advisory)
   - What's unclear: The context decisions don't specify an implementation approach — visual judgment, or a character count probe?
   - Recommendation: Visual judgment only (screenshot and assess) — character count probes require careful handling of variable-width fonts, and DESIGN-02 has the visual judgment path available

3. **Missing lazy-load placeholders (DESIGN-03) — detection approach?**
   - What we know: REQUIREMENTS.md mentions "missing lazy-load placeholders" as a DESIGN-03 requirement; context decisions don't specify detection
   - What's unclear: Lazy-load placeholder detection requires checking if `loading="lazy"` images have placeholder content visible before they load
   - Recommendation: Check `img[loading='lazy']` elements for `src` vs `data-src` pattern AND check if a placeholder background or element exists in the DOM. Visual judgment (screenshot before scroll) supplements.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| playwright-cli | All DESIGN checks | yes | 0.1.8 | — |
| Node.js | playwright-cli runtime | yes | 24.11.1 | — |

No external dependencies beyond the existing project stack. All browser APIs used (Web Fonts, CSSOM, HTMLImageElement) are available in any modern browser that Playwright targets.

---

## Validation Architecture

> nyquist_validation not explicitly set to false in config.json — section included.

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Manual verification (no automated test runner — QA agent is itself the testing tool) |
| Config file | N/A |
| Quick run command | Review qa-tester.md for correct structure and wording |
| Full suite command | `/qa:run` on a test project with `## Visual Focus: design-verification` |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| DESIGN-01 | Mockup comparison runs when Design Reference provided | manual | Spawn qa-tester with a spec containing Design Reference section | N/A |
| DESIGN-02 | Font load and overflow detected | manual | Spawn qa-tester on a page with custom fonts | N/A |
| DESIGN-03 | Broken/distorted/upscaled images flagged | manual | Spawn qa-tester on a page with images | N/A |
| DESIGN-04 | Dark mode and hover states verified | manual | Spawn qa-tester on a themed page | N/A |
| DESIGN-05 | Cross-page consistency sampled | manual | Spawn qa-tester on multi-page spec | N/A |

**Note:** This phase delivers agent methodology (Markdown text editing only). Verification is by reading qa-tester.md and confirming the methodology is correctly structured. The planner should include a verification task that runs the agent against an example spec to confirm the new methodology sections execute correctly.

### Wave 0 Gaps
None — no test infrastructure to create. Phase 9 is a documentation/methodology edit to `agents/qa-tester.md`.

---

## Security Domain

> security_enforcement not explicitly set to false — section included.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | no | — |
| V3 Session Management | no | — |
| V4 Access Control | no | — |
| V5 Input Validation | no | — |
| V6 Cryptography | no | — |

**Security note:** Phase 9 edits only `agents/qa-tester.md` — a Markdown instruction file. No code executes, no data is stored, no credentials are handled. Design reference image paths are relative project paths passed in the spec by the user — no security surface.

The only sensitive area: if the agent reads CSS custom properties that happen to encode sensitive values (e.g., `--auth-token`), those would appear in eval output. Methodology should note: "CSS custom property names beginning with `--auth`, `--token`, `--key`, or `--secret` should not be included in reports."

---

## Sources

### Primary (HIGH confidence)
- `agents/qa-tester.md` (this repo) — existing patterns for Structured Accessibility (lines 240-310), Confidence Table (lines 351-372), Accessibility Focus Trigger (lines 719-731), Visual Focus Trigger (lines 733-747), Design Reference handler (lines 749-763) [VERIFIED: read directly]
- `playwright-cli --help` (installed, v0.1.8) — command surface confirmed: eval, run-code, hover, screenshot, open --browser [VERIFIED: executed]
- `09-CONTEXT.md` — locked decisions D-01 through D-15 [VERIFIED: read directly]
- `skills/run/SKILL.md` lines 142-149 — forwarding block structure [VERIFIED: read directly]

### Secondary (MEDIUM confidence)
- MDN Web Docs — `document.fonts.check()`, `img.naturalWidth`, `img.complete`, `img.currentSrc`, `scrollWidth/clientWidth` — standard browser APIs, stable across modern browsers [CITED: MDN standard]
- Playwright `page.emulateMedia()` — existing usage confirmed in qa-tester.md line 290 [VERIFIED: codebase grep]

### Tertiary (LOW confidence)
- None.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — playwright-cli installed and verified; all browser APIs are DOM standards
- Architecture: HIGH — direct mirror of Structured Accessibility pattern already validated in Phase 4
- Pitfalls: HIGH — three of five pitfalls are derived from locked decisions (D-11, D-12 interaction state; D-07 eval command choice) and the user's specific session transcript failure

**Research date:** 2026-04-19
**Valid until:** 2026-10-19 (stable — browser APIs don't change; playwright-cli version pinned)
