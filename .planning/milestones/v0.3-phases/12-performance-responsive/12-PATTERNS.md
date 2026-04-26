# Phase 12: Performance & Responsive - Pattern Map

**Mapped:** 2026-04-20
**Files analyzed:** 1 (agents/qa-tester.md — modify only)
**Analogs found:** 1 / 1 (exact match within the same file)

---

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `agents/qa-tester.md` | agent methodology | event-driven + request-response | `agents/qa-tester.md` lines 749-1802 (Phases 9-11 sections) | exact — same file, same pattern, same structural conventions |

---

## Pattern Assignments

### `agents/qa-tester.md` — new `### Performance & Responsive` section

**Analog:** The three preceding Tier 2 visual sections within the same file:
- `### Design Verification` (lines 749-1098)
- `### UX State Verification` (lines 1099-1420)
- `### Layout & Content Integrity` (lines 1421-1802)

---

### Section Header and Security Note Pattern

**Source:** `agents/qa-tester.md` lines 1421-1427 (Layout & Content Integrity opening)

```markdown
### Layout & Content Integrity

Tier 2 layout and content integrity verification — runs when `layout-integrity` is listed in the spec's `## Visual Focus` section. All four areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).

**Security note:** When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports.

**Environment note:** DOM content injection (LAYOUT-03) mutates live page state. Use only in test environments — never against production data. Reload the page after all LAYOUT-03 checks before proceeding.
```

**How to apply for Phase 12:** Copy the section header/intro pattern verbatim. Use the same dual-pattern declaration sentence ("All four areas use the same dual pattern..."). The security note is required in every Tier 2 section — include it. Replace the environment note with a note about lab-value labeling (D-05 requirement: "All Web Vitals findings are labeled as lab-environment values").

---

### Numbered Procedure Step Pattern

**Source:** `agents/qa-tester.md` lines 1109-1200 (STATE-01 State Existence, active triggering steps)

```markdown
#### State Existence (STATE-01)

Verify that each of the four core application states ... Active triggering is the default methodology...

1. **Active triggering is the default methodology** (per D-01). For each state type, attempt to force the state into existence...

2. **Empty state** (per D-02): Clear storage and mock empty API response:
   - `playwright-cli run-code "async (page) => { ... }"` + `playwright-cli localstorage-clear`
   - Navigate to the page, take screenshot
   - Eval probe for empty state indicators:
   ```
   playwright-cli eval "() => { ... }"
   ```
   - If no empty state indicator found ...: flag as **High confidence** — "..."
   - Cleanup: `playwright-cli route-clear "*/api/*"` (per D-04)
```

**How to apply for Phase 12:** Each PERF-0X subsection uses numbered steps with sub-bullets for each command, followed by the `playwright-cli eval/run-code` snippet, followed by a conditional finding statement. The pattern: step → command(s) → eval snippet → "If [condition]: flag as **[Confidence]** — '[finding text]'".

---

### eval Snippet Format Pattern

**Source:** `agents/qa-tester.md` lines 800-840 (DESIGN-02 Typography Verification eval blocks)

```markdown
1. Run font load check:
   ```
   playwright-cli eval "async () => {
     await document.fonts.ready;
     const families = new Set();
     document.querySelectorAll('h1, h2, h3, p, span, a, button, li').forEach(el => {
       families.add(getComputedStyle(el).fontFamily);
     });
     return { fontsReady: true, families: Array.from(families) };
   }"
   ```
2. If the spec or a design tokens file declares expected font families, compare...
```

**How to apply for Phase 12:** The RESEARCH.md already provides the exact eval snippets for all four PERF areas (Patterns 1-6). Format them identically to the above: indented code block under the numbered step, using the `playwright-cli eval/run-code "..."` prefix. The multiline async patterns use `run-code` (not `eval`) — match the existing `run-code` block style from STATE-01 (lines 1114, 1139) and STATE-03 (line 1327).

---

### run-code Multiline Block Pattern

**Source:** `agents/qa-tester.md` lines 1327-1338 (STATE-03 sticky header scroll verification)

```markdown
   ```
   playwright-cli run-code "async (page) => {
     await page.mouse.wheel(0, 500);
     await page.waitForTimeout(300);
     const header = await page.evaluate(() => {
       const h = document.querySelector('header, [role=\"banner\"], nav:first-of-type');
       if (!h) return null;
       const rect = h.getBoundingClientRect();
       const stickyTop = parseInt(getComputedStyle(h).top) || 0;
       return { top: rect.top, expectedTop: stickyTop, sticking: Math.abs(rect.top - stickyTop) <= 2 };
     });
     return header;
   }"
   ```
```

**How to apply for Phase 12:** The PerformanceObserver injection (PERF-01 Step 1) and the CLS visibilitychange dispatch (Step 3) are multiline async patterns — use `playwright-cli run-code "async (page) => { ... }"` exactly as shown above. Note escaped internal quotes use `\"` not `'`.

---

### Confidence Closing Line Pattern

**Source:** `agents/qa-tester.md` lines 1550-1552 and 1691 (LAYOUT-01 and LAYOUT-03 closing lines)

```
Confidence: eval-confirmed overflow (scrollWidth > clientWidth) = High. Sibling gap outlier (>20% deviation from median) = Medium. Asymmetric padding (>4px imbalance) = Medium (design intent may be intentional). Grid/flex gap inconsistency = Medium. Visual alignment judgment from screenshot = Medium.
```

```
Confidence: eval-confirmed overflow after injection = High. Screenshot visual layout break after injection = Medium.
```

**How to apply for Phase 12:** Each PERF-0X subsection ends with a single `Confidence:` summary line. Format: `Confidence: [finding] = [Level]. [finding] = [Level].` — inline text, not a table. List High findings before Medium. Include the honest ceiling notes (e.g., "animation smoothness = not assessable" from STATE-04) for cases like INP-not-measured.

---

### Reporting Findings Table Pattern

**Source:** `agents/qa-tester.md` lines 1780-1802 (Reporting Layout & Content Integrity Findings)

```markdown
#### Reporting Layout & Content Integrity Findings

Present layout and content integrity findings grouped by confidence level, consistent with the existing design verification, UX state verification, and accessibility findings format.

| Finding type | Confidence |
| --- | --- |
| Content clipping detected (scrollWidth > clientWidth, any element) | High |
| Overflow confirmed after content injection (scrollWidth > clientWidth) | High |
| Placeholder regex match in visible text or attribute | High |
| ...
| Sibling gap outlier (>20% deviation from sibling median) | Medium |
```

**How to apply for Phase 12:** The final subsection is always `#### Reporting [Section Name] Findings` with the same intro sentence referencing all prior sections by name, followed by a two-column `| Finding type | Confidence |` table. List all High findings first, then Medium, then Observations. The planner should add a `#### Reporting Performance & Responsive Findings` subsection with entries for all PERF-01 through PERF-04 findings.

---

### Visual Focus Trigger Stub Removal

**Source:** `agents/qa-tester.md` lines 743 and 747

Line 743 (keep, update to remove pending note reference):
```
  `performance-responsive` — Core Web Vitals, cross-browser rendering, viewport sweep (Phase 12)
```

Line 747 (remove entirely — this is the pending note to delete):
```
**Note:** Tier 2 visual methodology for `performance-responsive` will be added in Phase 12. If a Visual Focus section requests that area before Phase 12 ships, note in the report: "Visual Focus requested for performance-responsive — full methodology pending Phase 12."
```

**How to apply for Phase 12:** The planner's first edit action is to delete lines 747-748 (the pending note paragraph). The area description at line 743 remains unchanged.

---

### Confidence Summary Table Extension Pattern

**Source:** `agents/qa-tester.md` lines 352-373 (Reporting Accessibility Findings table, which is the master confidence table)

```markdown
### Reporting Accessibility Findings

| Finding type | Confidence |
| --- | --- |
| Focus not moving into modal | High |
| Focus escaping modal (trap broken) | High |
...
| Focus not moving to first error after submission | Medium |
```

**How to apply for Phase 12:** The CONTEXT.md (line 79) says to extend the Confidence Summary Table at lines 355-375 with PERF-* entries. The research confirms this is at the accessibility reporting table. The planner should append PERF-* rows to this table after the last accessibility row (line ~373), following the same two-column format.

---

### Section Insertion Point

**Source:** `agents/qa-tester.md` lines 1801-1819

The last line of the Layout & Content Integrity reporting table (line 1802) is:
```
| Visual alignment/spacing judgment from screenshot | Medium |
```

Immediately after that (line 1803) is:
```
### Design Reference
```

**New section inserts between lines 1802 and 1803.** The full `### Performance & Responsive` section (all four PERF subsections + reporting table) goes here. The `### Design Reference` section shifts down.

---

## Shared Patterns

### Dual Pattern Declaration (applies to all four PERF subsections)
**Source:** `agents/qa-tester.md` lines 751-752 (Design Verification opening)
```markdown
All five areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).
```
**Apply to:** PERF-01, PERF-02, PERF-03, PERF-04 subsection introductions. Replace "five areas" with "four areas".

### Security Note (applies to all Tier 2 sections)
**Source:** `agents/qa-tester.md` line 755
```markdown
**Security note:** When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports.
```
**Apply to:** The `### Performance & Responsive` section header block. Required in all Tier 2 visual sections — see lines 755, 1103, and 1425.

### overflow detection (`scrollWidth > clientWidth`) reuse
**Source:** `agents/qa-tester.md` lines 1528-1548 (LAYOUT-01 content clipping) and lines 825-841 (DESIGN-02 text overflow)
```markdown
   playwright-cli eval "() => {
     const overflowing = [];
     document.querySelectorAll('p, h1, h2, h3, ...').forEach(el => {
       if (el.scrollWidth > el.clientWidth || el.scrollHeight > el.clientHeight) {
         overflowing.push({ tag: el.tagName, text: ..., scrollW: el.scrollWidth, clientW: el.clientWidth });
       }
     });
     return overflowing;
   }"
```
**Apply to:** PERF-03 breakpoint sweep overflow detection. The RESEARCH.md Pattern 3 extends this to `document.documentElement.scrollWidth > window.innerWidth` for document-level overflow and uses the same `scrollWidth > clientWidth` pattern for per-container checks.

### getBoundingClientRect intersection pattern
**Source:** `agents/qa-tester.md` lines 1183-1193 (STATE-01 first-run state bounding box check)
```markdown
     const r = el.getBoundingClientRect();
     return r.width > 0 && r.height > 0;
```
**Apply to:** PERF-04 cookie banner overlap detection. The RESEARCH.md Pattern 4 uses the full four-line intersection math (`!(bRect.right <= mRect.left || bRect.left >= mRect.right || bRect.bottom <= mRect.top || bRect.top >= mRect.bottom)`). This is the same getBoundingClientRect geometry pattern established across Phases 9-11.

### navigate + go-back sequence pattern
**Source:** `agents/qa-tester.md` lines 38-40 (core commands reference)
```
- `playwright-cli goto <url>` — navigate to a URL
- `playwright-cli go-back` / `playwright-cli go-forward` — browser history
```
**Apply to:** PERF-04 infinite scroll position recovery test. The 5-step sequence (eval scrollY → scroll down → click detail → go-back → eval scrollY) uses existing commands. No new commands needed.

### Engine-tagged findings pattern
**Source:** `agents/qa-tester.md` lines 1822-1823 (Browsers section)
```markdown
**Your responsibility:** Include the active engine name in your report header and findings. Example: "[webkit] Login form — PASS". If you notice rendering differences between engines (reported across multiple runs), note them as cross-browser findings.
```
**Apply to:** PERF-02 cross-browser rendering findings. Format cross-browser diff findings as "[webkit] [element selector] — [property]: [webkit value] vs [chromium baseline]".

---

## No Analog Found

None — all PERF-01 through PERF-04 patterns have direct analogs in the existing Phases 9-11 methodology sections.

---

## Code Excerpts Directly Reusable

### Overflow detection base (PERF-03 inherits from DESIGN-02/LAYOUT-01)
**Source:** `agents/qa-tester.md` lines 826-838
```
   playwright-cli eval "() => {
     const overflowing = [];
     document.querySelectorAll('p, h1, h2, h3, h4, h5, h6, span, li, a, button, label, td, th').forEach(el => {
       if (el.scrollWidth > el.clientWidth || el.scrollHeight > el.clientHeight) {
         overflowing.push({
           tag: el.tagName,
           text: el.textContent.slice(0, 60),
           scrollW: el.scrollWidth,
           clientW: el.clientWidth,
           scrollH: el.scrollHeight,
           clientH: el.clientHeight
         });
       }
     });
     return overflowing;
   }"
```

### getComputedStyle multi-property read (PERF-02 inherits from DESIGN-05)
**Source:** `agents/qa-tester.md` lines 1034-1057
```
   playwright-cli eval "() => {
     const sample = (sel) => {
       const el = document.querySelector(sel);
       if (!el) return null;
       const s = getComputedStyle(el);
       return {
         fontFamily: s.fontFamily,
         fontSize: s.fontSize,
         fontWeight: s.fontWeight,
         color: s.color,
         backgroundColor: s.backgroundColor,
         lineHeight: s.lineHeight
       };
     };
     return {
       h1: sample('h1'),
       body: sample('body'),
       button: sample('button, [role=\"button\"]'),
       ...
     };
   }"
```

### playwright-cli resize command (PERF-03)
**Source:** `agents/qa-tester.md` line 60
```
- `playwright-cli resize <width> <height>` — change viewport size
```

### Multi-viewport check list (PERF-03 extends lines 439-446)
**Source:** `agents/qa-tester.md` lines 439-446
```markdown
**What to check at each viewport:**
- Responsive breakpoint issues — overlapping elements, text truncation, hidden content that shouldn't be hidden
- Touch target sizes at mobile viewports...
- Horizontal scrolling — should not appear at any defined viewport. Check document width vs viewport width
- Navigation patterns — hamburger menus at mobile, full nav at desktop
- Content reflow — text and images should reflow gracefully, not overlap or get clipped
```

---

## Metadata

**Analog search scope:** `agents/qa-tester.md` (full file — single-file phase)
**Files scanned:** 1 source file + 2 planning documents
**Pattern extraction date:** 2026-04-20
**Phase is documentation-only:** New methodology text added to `agents/qa-tester.md`. No new source files, no new npm packages, no new infrastructure.

### Edit Operations for Planner

In priority order:

1. **Delete** lines 747-748 from `agents/qa-tester.md` (the "pending Phase 12" note paragraph)
2. **Insert** new `### Performance & Responsive` section at line 1803 (between last Layout & Content Integrity reporting table row and `### Design Reference`)
3. **Extend** the Confidence Summary Table (around line 373) with PERF-* rows appended after the last accessibility row
