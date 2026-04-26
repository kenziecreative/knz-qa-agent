# Phase 11: Layout & Content Integrity - Pattern Map

**Mapped:** 2026-04-20
**Files analyzed:** 1 (agents/qa-tester.md — single-file phase, methodology additions only)
**Analogs found:** 4 / 4 (all capability areas have direct analogs in existing methodology sections)

---

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `agents/qa-tester.md` (Layout & Content Integrity section) | methodology | eval + screenshot dual pattern | `agents/qa-tester.md` lines 1099-1420 (UX State Verification) | exact |
| `agents/qa-tester.md` (Confidence Summary Table extension) | methodology | reporting table | `agents/qa-tester.md` lines 1398-1420 (Reporting UX State Verification Findings) | exact |
| `agents/qa-tester.md` line 747 (stub removal) | methodology | trigger config | `agents/qa-tester.md` line 742 (`layout-integrity` area definition) | exact |

---

## Pattern Assignments

### Layout & Content Integrity Section (new section after line 1420)

**Analog:** `agents/qa-tester.md` lines 1099-1420 (UX State Verification section)

---

#### Section Header Pattern

**Source:** `agents/qa-tester.md` lines 1099-1106

```markdown
### UX State Verification

Tier 2 UX state verification — runs when `ux-states` is listed in the spec's `## Visual Focus` section. All four areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).

**Security note:** When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports.

**Environment note:** Active state triggering (storage clearing, cookie deletion, route mocking) manipulates application state. Use only in test environments — never against production data.
```

**Apply as:** Replace `UX State Verification`/`ux-states` identifiers and adjust the environment note for DOM mutation (LAYOUT-03 injection, not storage clearing). The security note carries over unchanged. The section intro sentence follows the same "All four areas use the same dual pattern" structure.

---

#### LAYOUT-01: Spacing & Alignment Checks

**Analog:** `agents/qa-tester.md` lines 820-850 (Text Overflow Detection, DESIGN-02)

**Core overflow detection pattern** (lines 824-841):

```javascript
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

**LAYOUT-01 extends this to all visible elements** (not just text tags) and adds the `el.offsetParent !== null` visibility guard — the same guard used throughout `agents/qa-tester.md` (confirmed at lines 1127, 1235, 1264).

**Sibling gap comparison pattern** (from RESEARCH.md Pattern 1 — Claude's Discretion):

```javascript
playwright-cli eval "() => {
  const OUTLIER_THRESHOLD = 0.2; // gap deviates > 20% from sibling median
  function median(arr) {
    const sorted = [...arr].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 !== 0 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
  }
  const results = [];
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
      const gap = curr.top - prev.bottom;
      if (Math.abs(gap) < 200) gaps.push(gap); // exclude section breaks
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

**Padding symmetry pattern** (from RESEARCH.md Pattern 2):

```javascript
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

**Grid/flex integrity pattern** (from RESEARCH.md Pattern 3):

```javascript
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

**Confidence line format** (copy from line 850):
```
Confidence: eval-detected overflow = High. Visual readability judgment = Medium.
```

For LAYOUT-01:
```
Confidence: eval-confirmed overflow (scrollWidth > clientWidth) = High. Sibling gap outlier (>20% deviation from median) = Medium. Asymmetric padding = Medium (design intent may be intentional). Grid/flex gap inconsistency = Medium.
```

---

#### LAYOUT-02: Cross-Page Structural Consistency

**Analog:** `agents/qa-tester.md` lines 1021-1074 (Cross-Page Consistency, DESIGN-05)

**Cross-page sampling procedure pattern** (lines 1031-1068):

```javascript
// DESIGN-05 analog — CSS style sampling across pages
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
    link: sample('a'),
    nav: sample('nav'),
    input: sample('input')
  };
}"
```

LAYOUT-02 uses the same multi-page visit pattern but queries structural/DOM properties instead of CSS style values.

**Structural fingerprint pattern** (from RESEARCH.md Pattern 4):

```javascript
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

**Link text sampling pattern** (from RESEARCH.md Pattern 4):

```javascript
playwright-cli eval "() => {
  function linkTexts(regionSel) {
    const region = document.querySelector(regionSel);
    if (!region) return null;
    return Array.from(region.querySelectorAll('a[href]'))
      .map(a => a.textContent.trim().toLowerCase().replace(/\s+/g, ' '))
      .filter(t => t.length > 0);
  }
  return {
    header: linkTexts('[role=\"banner\"], header'),
    nav: linkTexts('[role=\"navigation\"], nav'),
    footer: linkTexts('[role=\"contentinfo\"], footer')
  };
}"
```

**Single-page fallback pattern** (line 1066-1068):
```markdown
7. If the spec covers only one URL, cross-page comparison is not possible. Note: "Cross-page consistency requires multiple pages — [structural fingerprint] sampled on this page for reference." Record sampled values in the report without flagging any findings.
```

**Confidence line:**
```
Confidence: Missing landmark region (eval-confirmed) = High. Link count deviation > 1 = Medium. Link text present on one page but absent another = Medium. Single-page spec = no findings (observation only).
```

---

#### LAYOUT-03: Realistic-Data Overflow Testing

**Analog:** `agents/qa-tester.md` lines 1113-1137 (STATE-01 route interception + cleanup pattern)

**Route interception + cleanup pattern** (lines 1115, 1137):

```
playwright-cli route "*/api/*" "{\"body\": \"[]\", \"contentType\": \"application/json\"}"
# ... test ...
playwright-cli route-clear "*/api/*"
```

LAYOUT-03 uses the same route pattern as the escalation path (D-18), but the default mechanism is DOM textContent injection via `run-code`, not route interception.

**Content injection + overflow detection pattern** (from RESEARCH.md Pattern 5):

```javascript
// Step 1: identify target elements (one logical section at a time — D-15)
playwright-cli eval "() => {
  return Array.from(
    document.querySelectorAll('[class*=\"card\"] h2, [class*=\"card\"] h3, [class*=\"title\"]')
  ).filter(el => el.offsetParent !== null)
   .map(el => ({
     tag: el.tagName,
     class: String(el.className).slice(0, 50),
     original: el.textContent.trim().slice(0, 40)
   }));
}"

// Step 2: inject test content
playwright-cli run-code "async (page) => {
  await page.evaluate(() => {
    const el = document.querySelector('[class*=\"card\"] h2');
    if (el) el.textContent = 'Donaudampfschifffahrtsgesellschaft Produktionsprogrammzusammenfassung';
  });
  return 'injected';
}"

// Step 3: overflow check (reuses DESIGN-02 scrollWidth pattern — lines 824-841)
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

// Step 4: page reload to restore state (mandatory after LAYOUT-03 per pitfall 2)
playwright-cli navigate [same url]
```

**Tiered test vocabulary** (from RESEARCH.md Pattern 5):

| String | Purpose | Target elements |
|--------|---------|-----------------|
| `Hildegard von Bingen Stiftung Internationaler Preis für Frauengesundheitsforschung` | Basic length overflow | Card titles, table cells, headings |
| `Donaudampfschifffahrtsgesellschaft` | Word-break failure (no break point) | Any `overflow: hidden` container |
| `$1,234,567,890.99` | Fixed-width number container | Price fields, stats, dashboard counters |
| `firstname.lastname+tag@very-long-subdomain.example.com` | Email field overflow | Form inputs, profile displays |

**Cleanup mandate pattern** (analog: STATE-01 line 1137 and STATE-01 line 1198):
```
After EACH active state trigger, the route mock or storage change MUST be cleaned up before proceeding.
```
For LAYOUT-03: After all injection checks on a page, `playwright-cli navigate [same url]` before proceeding to LAYOUT-04 or other checks.

**Confidence line:**
```
Confidence: eval-confirmed overflow after injection = High. Screenshot visual layout break after injection = Medium.
```

---

#### LAYOUT-04: Placeholder & Draft Content Detection

**Analog:** `agents/qa-tester.md` lines 1159-1167 (STATE-01 error indicator pattern — regex + DOM scan combo)

**Error text scan pattern** (line 1163 — regex over `innerText`):

```javascript
const hasErrorText = document.body.innerText.match(/something went wrong|error|failed to load|try again|oops/i);
```

LAYOUT-04 extends this regex scanning approach to a comprehensive set of patterns and a wider scan scope (innerText, alt, title, aria-label, meta content).

**Full placeholder scan pattern** (from RESEARCH.md Pattern 6):

```javascript
playwright-cli eval "() => {
  const findings = [];

  const textPatterns = [
    { pattern: /lorem|ipsum|dolor sit amet|consectetur adipiscing/i, label: 'Lorem ipsum text' },
    { pattern: /\b(TODO|FIXME|HACK|XXX)\b/, label: 'TODO/FIXME marker in visible content' },
    { pattern: /\[Your (Name|Company|Email|Phone|Address)\]/i, label: 'Bracket placeholder' },
    { pattern: /\bexample\.com\b/i, label: 'example.com placeholder URL' },
    { pattern: /\btest@test\.com\b/i, label: 'test@test.com placeholder email' },
    { pattern: /\b123-456-7890\b/, label: 'Placeholder phone number' },
    { pattern: /\bJohn Doe\b|\bJane Doe\b/i, label: 'Generic placeholder name' }
  ];

  const imgPatterns = [
    { pattern: /placehold\.it|via\.placeholder\.com|lorempixel|picsum\.photos|dummyimage/i, label: 'Placeholder image URL' }
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
          type: label, location: 'visible text',
          snippet: text.slice(0, 80), tag: node.parentElement.tagName
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

**Allowlist check mandate** (from CONTEXT.md D-23): Before recording any LAYOUT-04 finding, check whether the matched text appears in the spec's `placeholder_allowlist`. If so, suppress the finding and note suppression in the report.

**Spec-level allowlist declaration** (spec format):
```yaml
placeholder_allowlist: [TODO, Learn more]
```

**Visual supplement pattern** — LLM screenshot judgment for ambiguous cases follows the same "screenshot + visual assessment at Medium confidence" structure used throughout DESIGN-01 (lines 761-782) and STATE-02 (line 1228):
```
Take a viewport screenshot and visually assess:
- Vague CTAs ("Click here", "Learn more", "Submit" without context)
- Casing inconsistency across same-type elements (nav link labels vs nav link labels only)
- Placeholder images (gray boxes, stock photo watermarks)
```

**Confidence line:**
```
Confidence: eval regex match in visible text/attributes = High. LLM screenshot judgment (vague CTA, casing inconsistency, placeholder image) = Medium. Allowlist-suppressed match = not reported.
```

---

#### Reporting Layout & Content Integrity Findings

**Analog:** `agents/qa-tester.md` lines 1398-1420 (Reporting UX State Verification Findings)

**Reporting table pattern** (lines 1402-1420):

```markdown
#### Reporting UX State Verification Findings

Present UX state verification findings grouped by confidence level, consistent with the existing design verification and accessibility findings format.

| Finding type | Confidence |
| --- | --- |
| Empty state missing (list renders zero items, no empty state messaging) | High |
...
| First-run/onboarding state not detected on fresh session | Observation |
```

**Apply as:** Same structure — section header, one-sentence format note, confidence table. Add these rows:

| Finding type | Confidence |
| --- | --- |
| Content clipping detected (scrollWidth > clientWidth, any element) | High |
| Overflow confirmed after content injection (scrollWidth > clientWidth) | High |
| Placeholder regex match in visible text or attribute | High |
| Placeholder image URL in img src | High |
| Missing landmark region across pages (no header/nav/footer) | High |
| Sibling gap outlier (>20% deviation from sibling median) | Medium |
| Asymmetric container padding (>4px imbalance) | Medium |
| Grid/flex gap inconsistency across similar containers | Medium |
| Nav link count deviation across pages (>1 link difference) | Medium |
| Nav link text present on one page but absent another | Medium |
| Visual layout break after injection (screenshot judgment) | Medium |
| Vague CTA detected (screenshot visual judgment) | Medium |
| Casing inconsistency across same-type elements (screenshot) | Medium |
| Placeholder image visual detection (screenshot) | Medium |

---

#### Line 747 Stub Removal

**Source:** `agents/qa-tester.md` line 747

```markdown
**Note:** Tier 2 visual methodology for `layout-integrity` and `performance-responsive` will be added in Phases 11-12. If a Visual Focus section requests those areas before their phases ship, note in the report: "Visual Focus requested for [areas] — full methodology pending Phase [N]."
```

**Change:** Remove `layout-integrity` from the pending list. Keep `performance-responsive`. Result:

```markdown
**Note:** Tier 2 visual methodology for `performance-responsive` will be added in Phase 12. If a Visual Focus section requests that area before Phase 12 ships, note in the report: "Visual Focus requested for performance-responsive — full methodology pending Phase 12."
```

---

## Shared Patterns

### Dual Eval + Screenshot Pattern
**Source:** `agents/qa-tester.md` lines 749-752, 1099-1101
**Apply to:** All four LAYOUT-* subsections

```markdown
All four areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).
```

### Visibility Guard Pattern
**Source:** Throughout `agents/qa-tester.md` (lines 1127, 1235, 1264, 1380)
**Apply to:** Every eval probe across all four LAYOUT-* areas

```javascript
.filter(el => el.offsetParent !== null)
```

Every eval probe that queries DOM elements must filter with `el.offsetParent !== null` to exclude hidden elements before running any measurement.

### Security Note
**Source:** `agents/qa-tester.md` lines 755, 1103
**Apply to:** Layout & Content Integrity section header (same as Design Verification and UX State Verification)

```markdown
**Security note:** When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports.
```

### Environment Note
**Source:** `agents/qa-tester.md` line 1105
**Apply to:** Layout & Content Integrity section header

Adapt the existing environment note to cover DOM mutation instead of storage clearing:

```markdown
**Environment note:** DOM content injection (LAYOUT-03) mutates live page state. Use only in test environments — never against production data. Reload the page after all LAYOUT-03 checks before proceeding.
```

### Overflow Detection Mechanism
**Source:** `agents/qa-tester.md` lines 826-841 (DESIGN-02 Text Overflow Detection)
**Apply to:** LAYOUT-01 content clipping (D-05), LAYOUT-03 post-injection overflow check (D-16)

The `scrollWidth > clientWidth || scrollHeight > clientHeight` check is the canonical overflow detection pattern in this codebase. LAYOUT-01 extends the element selector from text tags only to all visible elements. LAYOUT-03 reuses it verbatim after injection.

### Numbered Procedure Steps
**Source:** `agents/qa-tester.md` lines 761-791 (DESIGN-01), 1111-1198 (STATE-01)
**Apply to:** All four LAYOUT-* subsections

Each subsection uses numbered steps (1. 2. 3. ...) for the procedure, with inline code blocks for eval commands. Sub-steps use letters (a. b. c. ...). Confidence summary appears as a standalone line at the end of each subsection.

### Route Cleanup Mandate
**Source:** `agents/qa-tester.md` line 1137, line 1198
**Apply to:** LAYOUT-03 (page reload after injection), any route interception used in LAYOUT-03 escalation path

After any active DOM manipulation or route interception, cleanup is mandatory before proceeding to the next check.

---

## No Analog Found

No files in this phase have zero analog coverage. All patterns map to existing methodology in `agents/qa-tester.md`. The only discretionary work (exact JS eval snippets, threshold values, regex patterns) is fully specified in RESEARCH.md and ready for the planner to use directly.

---

## Metadata

**Analog search scope:** `agents/qa-tester.md` (lines 749-1420 — Design Verification + UX State Verification)
**Files scanned:** 1 primary analog (`agents/qa-tester.md`)
**Key sections read:** lines 749-1097 (Design Verification), lines 1099-1420 (UX State Verification), lines 340-374 (Accessibility Reporting table), lines 730-748 (Visual Focus Trigger)
**Pattern extraction date:** 2026-04-20
```
