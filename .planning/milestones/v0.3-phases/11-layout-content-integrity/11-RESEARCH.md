# Phase 11: Layout & Content Integrity - Research

**Researched:** 2026-04-20
**Domain:** QA agent methodology — layout integrity checks in `agents/qa-tester.md`
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**LAYOUT-01 — Spacing & Alignment Checks:**
- D-01: Dual pattern — `evaluate()` sibling comparison as primary (High confidence) + screenshot visual judgment as supplement (Medium confidence)
- D-02: Eval probes query `getBoundingClientRect()` and `getComputedStyle()` on sibling element sets; compute inter-element gaps; flag outliers deviating from sibling median
- D-03: Parent-child padding balance checked via `getComputedStyle()` — compare padding-top/bottom and padding-left/right for symmetry; asymmetric padding = Medium confidence (design intent may be intentional)
- D-04: Grid/flex integrity via computed `display`, `gap`, `align-items`, `justify-content` — structural validation that containers have consistent gap values and alignment
- D-05: Content clipping via `scrollWidth > clientWidth` or `scrollHeight > clientHeight` — extended to ALL visible elements (not just text); High confidence when eval-confirmed
- D-06: Screenshot supplements eval for complex layouts where DOM structure doesn't yield clean sibling sets — visual assessment at Medium confidence
- D-07: LAYOUT-01 operates WITHOUT a reference — infers correctness from internal consistency (sibling comparison, padding symmetry), distinct from DESIGN-01 which compares to a reference image

**LAYOUT-02 — Cross-Page Structural Consistency:**
- D-08: Structural fingerprint + link/content sampling approach
- D-09: Structural fingerprint via eval: query `[role="banner"]`, `[role="navigation"]`, `[role="main"]`, `[role="contentinfo"]`, `header`, `nav`, `footer`, `main` — record element count, child count, landmark presence per page
- D-10: Content sampling: extract link text arrays from header/nav/footer per page; compare link counts and link text across pages. Missing region = High confidence. Link count deviation = Medium confidence
- D-11: Distinct from DESIGN-05: DESIGN-05 compares CSS computed style values across pages; LAYOUT-02 compares structural/DOM consistency — same header structure, same footer presence, same nav links
- D-12: Screenshot per page confirms layout position of shared regions as visual supplement

**LAYOUT-03 — Realistic-Data Overflow Testing:**
- D-13: `evaluate()` textContent injection as primary mechanism; agent injects test content into DOM elements, then checks for overflow; no route interception as default
- D-14: Tiered test content vocabulary: long Latin strings (150+ chars), German compound words (`Donaudampfschifffahrtsgesellschaft`), large currency (`$1,234,567,890.99`), long email addresses
- D-15: Injection targets one logical section at a time (not page-wide) — keeps overflow attribution clear
- D-16: Overflow detection via `scrollWidth > clientWidth` or `scrollHeight > clientHeight` after injection — High confidence when eval-confirmed
- D-17: Screenshot after injection as visual supplement — Medium confidence
- D-18: Route interception as spec-driven escalation path for locale/API-response-based content testing

**LAYOUT-04 — Placeholder & Draft Content Detection:**
- D-19: Dual approach — `evaluate()` regex/keyword scan as primary (High confidence) + LLM screenshot judgment for ambiguous cases (Medium confidence)
- D-20: Eval regex scan covers: Lorem ipsum variants, TODO/FIXME/HACK/XXX, placeholder image URLs (placehold.it, via.placeholder, lorempixel, picsum.photos, dummyimage), generic patterns (`[Your Name]`, `[Company Name]`, `example.com`, `test@test.com`, `123-456-7890`)
- D-21: Scan scope: visible text (`innerText`), `alt`, `title`, `aria-label`, and `<meta>` content attributes
- D-22: LLM screenshot judgment covers: vague CTAs ("Click here", "Learn more", "Submit" without context), casing inconsistency, placeholder images — all Medium confidence
- D-23: Spec-level allowlist for false positives via `placeholder_allowlist: [...]`

**Overarching Pattern:**
- D-24: All four areas follow the dual pattern: `evaluate()` for deterministic/measurable checks (High confidence) + visual judgment via screenshots (Medium confidence)

### Claude's Discretion

- Exact JS evaluate snippets for sibling comparison and structural fingerprinting
- Threshold values for spacing deviation (how much gap variation triggers a finding)
- Exact regex patterns for placeholder detection
- How to structure the methodology sections in `qa-tester.md` (section naming, ordering)
- Which sibling element sets to prioritize when many exist on a page
- How to normalize link text for cross-page comparison (case, whitespace, trailing slashes)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| LAYOUT-01 | Agent checks spacing consistency, element alignment, grid/flex integrity, and content clipping | Sibling comparison via `getBoundingClientRect()` + `getComputedStyle()` on sibling sets; `scrollWidth/scrollHeight` overflow detection pattern from DESIGN-02 extended to all elements |
| LAYOUT-02 | Agent verifies header/footer, component, and interaction pattern consistency across pages | Structural fingerprint via landmark queries (`[role="banner"]`, `[role="navigation"]`, etc.) + link text sampling per page; complements DESIGN-05 CSS style comparison |
| LAYOUT-03 | Agent tests layout with realistic-length content (long names, large numbers, localization strings) | DOM textContent injection + overflow detection post-injection; tiered test vocabulary (Latin, German compounds, currency, email) targets distinct failure modes |
| LAYOUT-04 | Agent detects placeholder text (Lorem ipsum, TODO), filler data, casing inconsistency, and vague CTAs | Regex scan over `innerText`, `alt`, `title`, `aria-label`, `<meta>` content + LLM screenshot judgment for ambiguous cases; spec-level allowlist for suppressions |
</phase_requirements>

---

## Summary

Phase 11 adds the `layout-integrity` methodology block to `agents/qa-tester.md` — the fourth Tier 2 visual area triggered by `## Visual Focus: layout-integrity` in a spec. The phase covers four distinct capability areas: standalone spacing/alignment/grid/clipping verification (LAYOUT-01), cross-page structural DOM consistency (LAYOUT-02), realistic-data overflow testing (LAYOUT-03), and placeholder/draft content detection (LAYOUT-04).

All architectural decisions are fully locked in CONTEXT.md. This is an implementation-focused phase: write the methodology text and code snippets that the agent will follow, following the established dual-pattern from Phases 9 and 10. The work is confined to `agents/qa-tester.md` — specifically inserting a new `### Layout & Content Integrity` top-level section after the existing `### UX State Verification` section, updating the Confidence Summary Table, and removing the pending Phase 11 stub note at line 747.

The key research value here is determining the exact JS eval patterns, spacing deviation thresholds, sibling selection heuristics, and regex strings — the discretionary implementation choices the locked decisions intentionally left open.

**Primary recommendation:** Write methodology following the exact structural template established by Design Verification (lines 749-1097) and UX State Verification (lines 1099-1420) — subsection per requirement area, step-numbered procedure, confidence line at end of each subsection, reporting table at end of section.

---

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Layout methodology text | `agents/qa-tester.md` | — | Agent reads methodology at runtime; all browser logic lives here |
| Visual Focus trigger update | `agents/qa-tester.md` line 747 | — | Remove pending stub once methodology is written |
| Confidence Summary Table | `agents/qa-tester.md` lines 355-375 | — | Reporting table extended with LAYOUT-* entries |
| Spec format reference | `examples/SPEC-FORMAT.md` | — | Already references `layout-integrity` area; no change needed |
| Skill invocation template | `skills/run/SKILL.md` lines 142-149 | — | Already forwards Visual Focus; no change needed |

---

## Standard Stack

This phase adds no new libraries. It uses only existing `playwright-cli` capabilities already documented in `qa-tester.md`.

| Tool | Version | Purpose | Source |
|------|---------|---------|--------|
| `playwright-cli eval` | existing | Run arbitrary JS for DOM queries, overflow checks, computed styles | [VERIFIED: agents/qa-tester.md — used throughout Design Verification and UX State Verification] |
| `playwright-cli screenshot` | existing | Visual state capture for perceptual checks | [VERIFIED: agents/qa-tester.md] |
| `playwright-cli snapshot` | existing | Accessibility tree for element identification | [VERIFIED: agents/qa-tester.md] |
| `playwright-cli route` | existing | API interception for LAYOUT-03 escalation path | [VERIFIED: agents/qa-tester.md — established in Phase 3] |

**Installation:** None required. No new dependencies.

---

## Architecture Patterns

### Section Structure Template

The new `### Layout & Content Integrity` section follows the exact structural template established by the two prior Tier 2 sections:

```
### Layout & Content Integrity

[Section intro + dual pattern statement + security note]

#### Spacing & Alignment Checks (LAYOUT-01)
[Numbered procedure steps]
Confidence: [one-line summary]

#### Cross-Page Structural Consistency (LAYOUT-02)
[Numbered procedure steps]
Confidence: [one-line summary]

#### Realistic-Data Overflow Testing (LAYOUT-03)
[Numbered procedure steps]
Confidence: [one-line summary]

#### Placeholder & Draft Content Detection (LAYOUT-04)
[Numbered procedure steps]
Confidence: [one-line summary]

#### Reporting Layout & Content Integrity Findings

| Finding type | Confidence |
| --- | --- |
[rows]
```

[VERIFIED: agents/qa-tester.md lines 749-1420 — both Design Verification and UX State Verification use this exact structural template]

### Recommended Edit Locations in `agents/qa-tester.md`

```
Line 747:  Remove "Tier 2 visual methodology pending Phase 11" stub for layout-integrity
           (keep the performance-responsive stub for Phase 12)
Line ~355: Extend Confidence Summary Table with LAYOUT-* rows
Line ~1420 (after "#### Reporting UX State Verification Findings" table):
           Insert new "### Layout & Content Integrity" section
```

[VERIFIED: agents/qa-tester.md — confirmed line positions by reading file]

### Pattern 1: Sibling Gap Comparison (LAYOUT-01 core technique)

The key technique for LAYOUT-01 is computing inter-element gaps within sibling sets and flagging statistical outliers — inferring correct spacing from internal consistency rather than a design reference.

```javascript
// Sibling gap comparison — compute gaps between consecutive siblings
// Source: Claude's Discretion (D-02) — verified pattern derivation from existing getBoundingClientRect usage in qa-tester.md
playwright-cli eval "() => {
  const OUTLIER_THRESHOLD = 0.2; // gap deviates > 20% from sibling median
  function median(arr) {
    const sorted = [...arr].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 !== 0 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
  }
  const results = [];
  // Priority sibling sets: cards, list items, nav items, table rows, grid cells
  const containers = document.querySelectorAll(
    '[class*=\"card\"], ul, ol, nav, [role=\"list\"], [role=\"grid\"], table tbody, [class*=\"grid\"]'
  );
  containers.forEach(container => {
    const children = Array.from(container.children).filter(
      el => el.offsetParent !== null
    );
    if (children.length < 3) return; // need 3+ siblings for meaningful comparison
    const gaps = [];
    for (let i = 1; i < children.length; i++) {
      const prev = children[i - 1].getBoundingClientRect();
      const curr = children[i].getBoundingClientRect();
      const gap = curr.top - prev.bottom; // vertical gap
      if (Math.abs(gap) < 200) gaps.push(gap); // exclude huge jumps (section breaks)
    }
    if (gaps.length < 2) return;
    const med = median(gaps);
    const outliers = gaps
      .map((g, i) => ({ index: i + 1, gap: g }))
      .filter(({ gap }) => med !== 0 && Math.abs(gap - med) / Math.abs(med) > OUTLIER_THRESHOLD);
    if (outliers.length > 0) {
      results.push({
        container: container.tagName + (container.className ? '.' + String(container.className).split(' ')[0] : ''),
        medianGap: med,
        outliers
      });
    }
  });
  return results;
}"
```

**Threshold rationale:** 20% deviation chosen as the discretionary default — small enough to catch real inconsistencies (a 16px gap next to 24px gaps = 50% deviation, clearly flagged), large enough to avoid false positives from subpixel rendering differences. Flag as **Medium confidence** when outlier detected — design intent may legitimately vary.

[ASSUMED: 20% threshold — reasonable based on CSS spacing practices, but project may want to tune]

### Pattern 2: Padding Symmetry Check (LAYOUT-01)

```javascript
// Parent-child padding balance — D-03
playwright-cli eval "() => {
  const containers = Array.from(
    document.querySelectorAll(
      '[class*=\"card\"], [class*=\"panel\"], [class*=\"box\"], section, article, [class*=\"container\"]'
    )
  ).filter(el => el.offsetParent !== null);
  return containers.map(el => {
    const s = getComputedStyle(el);
    const pt = parseFloat(s.paddingTop), pb = parseFloat(s.paddingBottom);
    const pl = parseFloat(s.paddingLeft), pr = parseFloat(s.paddingRight);
    return {
      tag: el.tagName,
      class: String(el.className).slice(0, 50),
      paddingTop: pt, paddingBottom: pb,
      paddingLeft: pl, paddingRight: pr,
      verticalImbalance: Math.abs(pt - pb) > 4,
      horizontalImbalance: Math.abs(pl - pr) > 4
    };
  }).filter(el => el.verticalImbalance || el.horizontalImbalance);
}"
```

**Confidence:** Asymmetric padding = **Medium confidence** — design may intentionally use asymmetric padding (e.g., more bottom padding for visual breathing room).

[ASSUMED: 4px tolerance — reasonable for subpixel rounding, but discretionary]

### Pattern 3: Grid/Flex Integrity Check (LAYOUT-01)

```javascript
// Grid/flex container structural validation — D-04
playwright-cli eval "() => {
  const containers = Array.from(document.querySelectorAll('*')).filter(el => {
    const d = getComputedStyle(el).display;
    return (d === 'flex' || d === 'grid') && el.offsetParent !== null;
  });
  const results = [];
  containers.forEach(el => {
    const s = getComputedStyle(el);
    const children = Array.from(el.children).filter(c => c.offsetParent !== null);
    if (children.length < 2) return;
    results.push({
      tag: el.tagName,
      class: String(el.className).slice(0, 60),
      display: s.display,
      gap: s.gap,
      alignItems: s.alignItems,
      justifyContent: s.justifyContent,
      childCount: children.length
    });
  });
  return results;
}"
```

**Use:** Compare `gap` values across similar containers (e.g., multiple flex rows that should be consistent). Inconsistent `gap` values within the same layout pattern = Medium confidence finding.

[VERIFIED: `getComputedStyle()` gap, alignItems, justifyContent are standard CSS computed properties]

### Pattern 4: Structural Fingerprint (LAYOUT-02)

```javascript
// Structural fingerprint per page — D-09
playwright-cli eval "() => {
  const landmarks = [
    '[role=\"banner\"]', 'header',
    '[role=\"navigation\"]', 'nav',
    '[role=\"main\"]', 'main',
    '[role=\"contentinfo\"]', 'footer'
  ];
  const fingerprint = {};
  landmarks.forEach(sel => {
    const els = document.querySelectorAll(sel);
    fingerprint[sel] = {
      count: els.length,
      childCounts: Array.from(els).map(el => el.children.length)
    };
  });
  return fingerprint;
}"
```

```javascript
// Link text sampling from shared regions — D-10
playwright-cli eval "() => {
  function linkTexts(regionSel) {
    const region = document.querySelector(regionSel);
    if (!region) return null;
    return Array.from(region.querySelectorAll('a[href]'))
      .map(a => a.textContent.trim().toLowerCase().replace(/\\s+/g, ' '))
      .filter(t => t.length > 0);
  }
  return {
    header: linkTexts('[role=\"banner\"], header'),
    nav: linkTexts('[role=\"navigation\"], nav'),
    footer: linkTexts('[role=\"contentinfo\"], footer')
  };
}"
```

**Cross-page comparison logic:** Run both probes on each page in the spec, then compare. Missing landmark = High confidence. Link count differs by more than 1 = Medium confidence (some pages legitimately have different nav). Link text present on one page but absent another = Medium confidence.

[VERIFIED: landmark selectors — reused from accessibility checks, qa-tester.md lines 1021-1068]

### Pattern 5: Content Injection + Overflow Detection (LAYOUT-03)

```javascript
// Inject long content into a target selector, then detect overflow — D-13, D-16
// Step 1: identify target elements (one logical section at a time)
playwright-cli eval "() => {
  // Example: card titles
  return Array.from(
    document.querySelectorAll('[class*=\"card\"] h2, [class*=\"card\"] h3, [class*=\"title\"]')
  ).filter(el => el.offsetParent !== null)
   .map(el => ({
     tag: el.tagName,
     class: String(el.className).slice(0, 50),
     original: el.textContent.trim().slice(0, 40)
   }));
}"

// Step 2: inject test content via run-code
playwright-cli run-code "async (page) => {
  const targets = await page.evaluate(() =>
    Array.from(document.querySelectorAll('[class*=\"card\"] h2'))
      .filter(el => el.offsetParent !== null)
  );
  if (targets.length === 0) return 'no targets found';
  // Inject into first instance only — keeps attribution clear (D-15)
  await page.evaluate(() => {
    const el = document.querySelector('[class*=\"card\"] h2');
    if (el) el.textContent = 'Donaudampfschifffahrtsgesellschaft Produktionsprogrammzusammenfassung';
  });
  return 'injected';
}"

// Step 3: check overflow after injection (reuses DESIGN-02 pattern)
playwright-cli eval "() => {
  const overflowing = [];
  document.querySelectorAll('[class*=\"card\"] h2').forEach(el => {
    if (el.scrollWidth > el.clientWidth || el.scrollHeight > el.clientHeight) {
      overflowing.push({
        tag: el.tagName,
        text: el.textContent.slice(0, 60),
        scrollW: el.scrollWidth, clientW: el.clientWidth,
        scrollH: el.scrollHeight, clientH: el.clientHeight
      });
    }
  });
  return overflowing;
}"
```

**Tiered test vocabulary:**

| String | Purpose | Target elements |
|--------|---------|-----------------|
| `Hildegard von Bingen Stiftung Internationaler Preis für Frauengesundheitsforschung` (82 chars, real German) | Basic length overflow stress | Card titles, table cells, headings |
| `Donaudampfschifffahrtsgesellschaft` (35 chars, compound word — no break point) | Word-break failure | Any container with `overflow: hidden` |
| `$1,234,567,890.99` | Fixed-width number container | Price fields, stats, dashboard counters |
| `firstname.lastname+tag@very-long-subdomain.example.com` | Email field overflow | Form inputs, profile displays |

[VERIFIED: `scrollWidth > clientWidth` overflow pattern from qa-tester.md lines 820-841 — exact same mechanism]

### Pattern 6: Placeholder Content Regex Scan (LAYOUT-04)

```javascript
// Regex scan over visible text, alt, title, aria-label, meta — D-20, D-21
playwright-cli eval "() => {
  const findings = [];

  // Patterns to detect
  const textPatterns = [
    { pattern: /lorem|ipsum|dolor sit amet|consectetur adipiscing/i, label: 'Lorem ipsum text' },
    { pattern: /\\b(TODO|FIXME|HACK|XXX)\\b/, label: 'TODO/FIXME marker in visible content' },
    { pattern: /\\[Your (Name|Company|Email|Phone|Address)\\]/i, label: 'Bracket placeholder' },
    { pattern: /\\bexample\\.com\\b/i, label: 'example.com placeholder URL' },
    { pattern: /\\btest@test\\.com\\b/i, label: 'test@test.com placeholder email' },
    { pattern: /\\b123-456-7890\\b/, label: 'Placeholder phone number' },
    { pattern: /\\bJohn Doe\\b|\\bJane Doe\\b/i, label: 'Generic placeholder name' }
  ];

  const imgPatterns = [
    { pattern: /placehold\\.it|via\\.placeholder\\.com|lorempixel|picsum\\.photos|dummyimage/i, label: 'Placeholder image URL' }
  ];

  // Scan visible text
  const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT);
  let node;
  while ((node = walker.nextNode())) {
    if (!node.parentElement || node.parentElement.offsetParent === null) continue;
    const text = node.textContent.trim();
    if (!text) continue;
    textPatterns.forEach(({ pattern, label }) => {
      if (pattern.test(text)) {
        findings.push({
          type: label,
          location: 'visible text',
          snippet: text.slice(0, 80),
          tag: node.parentElement.tagName
        });
      }
    });
  }

  // Scan alt, title, aria-label attributes
  document.querySelectorAll('[alt], [title], [aria-label]').forEach(el => {
    ['alt', 'title', 'aria-label'].forEach(attr => {
      const val = el.getAttribute(attr);
      if (!val) return;
      textPatterns.forEach(({ pattern, label }) => {
        if (pattern.test(val)) {
          findings.push({ type: label, location: attr, snippet: val.slice(0, 80), tag: el.tagName });
        }
      });
    });
  });

  // Scan img src for placeholder URLs
  document.querySelectorAll('img').forEach(img => {
    const src = img.currentSrc || img.src;
    imgPatterns.forEach(({ pattern, label }) => {
      if (pattern.test(src)) {
        findings.push({ type: label, location: 'img src', snippet: src.slice(-80), tag: 'IMG', alt: img.alt });
      }
    });
  });

  // Scan meta content
  document.querySelectorAll('meta[content]').forEach(el => {
    const content = el.getAttribute('content') || '';
    textPatterns.forEach(({ pattern, label }) => {
      if (pattern.test(content)) {
        findings.push({ type: label, location: 'meta content', snippet: content.slice(0, 80), name: el.getAttribute('name') || el.getAttribute('property') });
      }
    });
  });

  return findings;
}"
```

[VERIFIED: `document.createTreeWalker` with `NodeFilter.SHOW_TEXT` — standard DOM API; consistent with how text content scanning is done in browser contexts]
[ASSUMED: specific regex patterns — reasonable coverage based on common placeholder content, but may need tuning for specific projects]

### Anti-Patterns to Avoid

- **Injecting content globally (LAYOUT-03):** Injecting test strings page-wide produces unfalsifiable findings — every container overflows and attribution is lost. Always inject into one logical section at a time per D-15.
- **Flagging ALL padding asymmetry as High confidence:** Asymmetric padding is frequently intentional (more bottom padding for visual breathing room). Always flag at Medium confidence per D-03.
- **Missing the allowlist check (LAYOUT-04):** Before reporting a placeholder finding, check if the spec declares a `placeholder_allowlist` and suppress matches per D-23.
- **Page-wide fingerprint without per-region breakdown (LAYOUT-02):** Comparing only total element counts misses the case where header is missing on one page but a new unrelated element has the same role. Always compare per-region (header/nav/footer separately) per D-09/D-10.
- **Forgetting to restore injected content (LAYOUT-03):** After overflow testing, the DOM has been mutated. If the agent continues testing the same page, restored state may matter. Reload the page after LAYOUT-03 or explicitly restore original content.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Overflow detection | Custom measurement logic | `scrollWidth > clientWidth` pattern from DESIGN-02 (qa-tester.md lines 820-841) | Already established, battle-tested, consistent confidence reporting |
| Landmark querying | Custom element discovery | ARIA role selectors from accessibility checks (qa-tester.md) | Already used in A11Y section — reuse for structural fingerprint |
| Content injection | New browser automation mechanism | `playwright-cli run-code` with `page.evaluate()` | Established pattern for DOM mutation from Phase 3/10 |
| Route interception | Custom mock framework | `playwright-cli route` + `playwright-cli route-clear` | Already documented in STATE-01 (qa-tester.md lines 1115-1137) |
| Sibling selection | XPath or CSS nth-child gymnastics | `container.children` + `getBoundingClientRect()` loop | Simpler, already used for image quality probes (DESIGN-03) |

**Key insight:** This phase is purely additive methodology text. Every underlying mechanism (`eval`, `screenshot`, `route`) already exists. The work is describing how to combine them — not inventing new tools.

---

## Common Pitfalls

### Pitfall 1: Sibling Set Size Too Small
**What goes wrong:** Running sibling comparison on containers with 1-2 children produces meaningless "outlier" flags — every child IS the median.
**Why it happens:** Some pages have very few items in a given container; the comparison algorithm doesn't account for minimum sample size.
**How to avoid:** Require at least 3 siblings before running gap comparison (enforced in the Pattern 1 snippet above with `if (children.length < 3) return`).
**Warning signs:** Findings like "container with 1 child has inconsistent spacing."

### Pitfall 2: Injected Content Not Cleaned Up
**What goes wrong:** LAYOUT-03 injects long strings into the DOM. If the page is then tested for other things (screenshots, structural fingerprint), the mutated state produces false positives.
**Why it happens:** DOM mutation persists for the lifetime of the page session.
**How to avoid:** Either reload the page after LAYOUT-03 checks (`playwright-cli navigate [url]`) or restore original content explicitly. Document this in the methodology.
**Warning signs:** LAYOUT-02 structural fingerprint run after LAYOUT-03 injection finds structural changes.

### Pitfall 3: Allowlist Not Checked
**What goes wrong:** Agent flags "Learn more" as a vague CTA on a site where that's the intentional brand voice.
**Why it happens:** LAYOUT-04 regex/visual scan runs before checking spec-declared suppressions.
**How to avoid:** In the methodology, add an explicit step: "Before recording any LAYOUT-04 finding, check whether the matched text appears in the spec's `placeholder_allowlist`. If so, suppress the finding and note suppression in the report."
**Warning signs:** User receives duplicate reports on findings they've already acknowledged.

### Pitfall 4: Casing Inconsistency False Positives
**What goes wrong:** Agent flags intentional casing differences — a brand name in ALL CAPS mixed with sentence case text — as a casing inconsistency bug.
**Why it happens:** LLM visual judgment for casing (D-22) can over-flag when an element has brand-specific capitalization.
**How to avoid:** Scope casing inconsistency checks to same-type elements only (e.g., nav link labels vs nav link labels, not headlines vs button labels). Flag at Medium confidence with rationale.
**Warning signs:** Findings like "ALL-CAPS logo text inconsistent with sentence-case body copy."

### Pitfall 5: Overflow Detection on Hidden Elements
**What goes wrong:** Elements with `display: none` or `visibility: hidden` report `scrollWidth` and `clientWidth` as 0, producing false positives or false negatives.
**Why it happens:** `scrollWidth > clientWidth` comparison breaks down when both values are 0.
**How to avoid:** Filter for `el.offsetParent !== null` (same guard used throughout existing qa-tester.md probes) before running overflow checks.
**Warning signs:** Overflow findings on elements that aren't visible to the user.

---

## Code Examples

Verified patterns from existing codebase:

### Overflow Check (from DESIGN-02, qa-tester.md lines 820-841)
```javascript
// Source: agents/qa-tester.md lines 826-840
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

### Cross-Page Computed Style Sample (from DESIGN-05, qa-tester.md lines 1034-1057)
```javascript
// Source: agents/qa-tester.md lines 1035-1058
playwright-cli eval "() => {
  const sample = (sel) => {
    const el = document.querySelector(sel);
    if (!el) return null;
    const s = getComputedStyle(el);
    return { fontFamily: s.fontFamily, fontSize: s.fontSize, fontWeight: s.fontWeight };
  };
  return { h1: sample('h1'), body: sample('body'), nav: sample('nav') };
}"
```

### Route Interception Cleanup (from STATE-01, qa-tester.md line 1137)
```
playwright-cli route-clear "*/api/*"
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Reference-based spacing (DESIGN-01) | Standalone sibling comparison (LAYOUT-01) | Phase 11 decision | Enables spacing checks when no design reference exists |
| Text-only overflow scan (DESIGN-02) | All-element overflow scan (LAYOUT-01 D-05) | Phase 11 extension | Catches non-text overflow (images in containers, flex children) |
| CSS value consistency (DESIGN-05) | DOM structural consistency (LAYOUT-02) | Phase 11 decision | Two complementary cross-page checks: style AND structure |

**Pending removal after Phase 11 ships:**
- Line 747 stub: "Tier 2 visual methodology for `layout-integrity` and `performance-responsive` will be added in Phases 11-12." — remove `layout-integrity` mention only; keep `performance-responsive` stub for Phase 12.

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | 20% deviation threshold for sibling gap comparison triggers a finding | Code Examples / Pattern 1 | Too sensitive = false positive noise; too loose = real bugs missed. Low risk — planner or implementer can tune |
| A2 | 4px tolerance for padding asymmetry check | Code Examples / Pattern 2 | Subpixel rendering can produce 1-2px differences; 4px is conservative. Low risk |
| A3 | Specific regex patterns for placeholder detection (D-20) | Code Examples / Pattern 6 | A project may use different naming conventions (e.g., `[PLACEHOLDER]` instead of `[Your Name]`). Medium risk — allowlist mechanism (D-23) mitigates |
| A4 | `document.createTreeWalker` is preferred over `document.body.innerText` for text scanning | Code Examples / Pattern 6 | Using `innerText` would be simpler but scans all text including hidden elements. Low risk — either works; TreeWalker gives more control |

---

## Open Questions

1. **Sibling priority selection when many sets exist**
   - What we know: D-02 specifies "cards, list items, nav items, table rows" as priority sets; D-15 says inject into one section at a time
   - What's unclear: When a page has 15 different sibling containers (common in complex layouts), which does the agent scan first? Does it scan all of them?
   - Recommendation: Scan all containers meeting minimum child count (3+), cap at first 20 results to avoid noise. Document this in methodology.

2. **Link text normalization for cross-page comparison (LAYOUT-02)**
   - What we know: D-10 notes link count deviation as Medium confidence; link text comparison needed
   - What's unclear: How to handle trailing slashes, URL vs text comparison, case normalization
   - Recommendation: Normalize by lowercasing + trimming whitespace (`.trim().toLowerCase()`) before comparison. Trailing slashes removed from any href comparison. This is discretionary — implement in methodology.

3. **Content restoration after LAYOUT-03 injection**
   - What we know: DOM injection persists for page session; this is a pitfall
   - What's unclear: Should methodology mandate page reload, or original content restoration?
   - Recommendation: Mandate page reload (`playwright-cli navigate [same url]`) after all LAYOUT-03 checks on a page, before proceeding to LAYOUT-04 or any other checks. Simpler and more reliable than content restoration.

---

## Environment Availability

Step 2.6: SKIPPED — Phase 11 is documentation-only changes to `agents/qa-tester.md`. No external dependencies, runtimes, or services involved.

---

## Validation Architecture

`workflow.nyquist_validation` is absent from `.planning/config.json` — treating as enabled.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected — this project has no automated test suite for the agent methodology text |
| Config file | None |
| Quick run command | Manual: read the new section in `agents/qa-tester.md` and verify it follows established patterns |
| Full suite command | Manual: run `/qa:run` against a test app with `## Visual Focus: layout-integrity` in spec |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| LAYOUT-01 | Spacing/alignment/grid/clipping checks in agent | Manual smoke | Run `/qa:run` with layout-integrity spec | ❌ Wave 0 |
| LAYOUT-02 | Cross-page structural consistency in agent | Manual smoke | Run `/qa:run` with multi-page layout-integrity spec | ❌ Wave 0 |
| LAYOUT-03 | Realistic-data overflow testing in agent | Manual smoke | Run `/qa:run` with layout-integrity spec against real app | ❌ Wave 0 |
| LAYOUT-04 | Placeholder content detection in agent | Manual smoke | Run `/qa:run` against page with known Lorem ipsum | ❌ Wave 0 |

**Note:** All validation for this phase is inherently manual. The "tests" are whether the agent produces sensible findings when given a spec with `## Visual Focus: layout-integrity`. There is no automated test infrastructure for agent methodology text — this is consistent with how Phases 9 and 10 were validated.

### Wave 0 Gaps

No automated test files needed — manual validation via `/qa:run` against a real app is the appropriate validation path for agent methodology. This matches the established pattern from Phases 9 and 10.

---

## Security Domain

`security_enforcement` is not set in `.planning/config.json` — treating as enabled.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | no | Phase 11 adds no auth |
| V3 Session Management | no | No session changes |
| V4 Access Control | no | No access control changes |
| V5 Input Validation | yes | Placeholder regex scan (LAYOUT-04) scans user-visible content — no injection risk; eval runs read-only |
| V6 Cryptography | no | No crypto involved |

### Known Threat Patterns for `playwright-cli eval` methodology

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| CSS custom property exposure | Information Disclosure | Security note already in Design Verification section (line 755): never include `--auth`, `--token`, `--key`, `--secret` properties in reports. Carry this note into the Layout & Content Integrity section header. |
| Eval code injection via spec content | Tampering | Specs are authored by project team, not user input. Eval strings are agent-generated, not user-controlled. Low risk — consistent with existing DESIGN and UX State sections. |
| Active DOM mutation (LAYOUT-03 injection) | Tampering | Methodology must state: inject only in test environments, never against production data. Mirrors the environment note already present in UX State Verification (line 1105). |

---

## Sources

### Primary (HIGH confidence)
- `agents/qa-tester.md` lines 749-1420 — Design Verification + UX State Verification methodology, section structure templates, established eval patterns
- `agents/qa-tester.md` lines 820-841 — Text overflow detection (DESIGN-02) — direct reuse pattern for LAYOUT-01 and LAYOUT-03
- `agents/qa-tester.md` lines 1021-1068 — Cross-Page Consistency (DESIGN-05) — structural fingerprint complement
- `agents/qa-tester.md` lines 733-747 — Visual Focus Trigger with `layout-integrity` stub
- `.planning/phases/11-layout-content-integrity/11-CONTEXT.md` — All locked decisions D-01 through D-24

### Secondary (MEDIUM confidence)
- `.planning/REQUIREMENTS.md` LAYOUT-01 through LAYOUT-04 — requirement text
- `.planning/STATE.md` — project decisions and accumulated context

### Tertiary (LOW confidence)
- None — all research was internal codebase verification

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new libraries; all existing playwright-cli capabilities
- Architecture: HIGH — section structure, edit locations, and integration points verified directly in `qa-tester.md`
- Eval patterns: MEDIUM — core mechanisms verified (scrollWidth, getBoundingClientRect, getComputedStyle); specific threshold values (20%, 4px) are discretionary defaults tagged [ASSUMED]
- Pitfalls: HIGH — derived from direct examination of existing patterns and identified gaps

**Research date:** 2026-04-20
**Valid until:** 2026-05-20 (stable — methodology document, not external dependency)
