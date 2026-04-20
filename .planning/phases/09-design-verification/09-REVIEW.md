---
phase: 09-design-verification
reviewed: 2026-04-19T00:00:00Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - agents/qa-tester.md
finding_count: 4
severity:
  critical: 0
  high: 1
  medium: 2
  low: 1
  info: 0
status: findings
---

# Phase 09: Code Review Report

**Reviewed:** 2026-04-19
**Depth:** standard
**Files Reviewed:** 1 (`agents/qa-tester.md` — Design Verification section, lines 749–1097)
**Status:** findings

## Summary

Phase 09 added the `### Design Verification` section with five subsections (DESIGN-01 through DESIGN-05) plus a confidence table. The overall methodology is solid: the dual eval + visual pattern is well-structured, security filtering is present in all three locations it was required, the confidence table row counts match the plan spec (6 High, 10 Medium), and the dark mode run-code pattern is architecturally correct.

Four findings: one high (division-by-zero producing false positives in the image distortion probe), two medium (querySelector mismatch in hover-then-eval; confidence table description inconsistency), one low (regex case-sensitivity gap in the security filter).

---

## High

### H-01: Division by zero in image distortion probe produces false `distorted: true`

**File:** `agents/qa-tester.md:871-874`

**Issue:** The `distorted` computation divides by `img.naturalHeight` and by `img.getBoundingClientRect().height`. Either can be zero — an SVG with no intrinsic height, a display:none image briefly in the DOM, or a zero-height container. When the denominator is zero, the ratio becomes `Infinity`, and `Math.abs(Infinity - anything) > 0.05` is always `true`. The agent would flag every such image as distorted with High confidence (after object-fit follow-up), even when the image is fine.

The same division-by-zero applies to the rendered height in the `upscaled` check: if `getBoundingClientRect().height === 0`, the comparison `img.getBoundingClientRect().height > img.naturalHeight` is `0 > naturalHeight` which is false — so `upscaled` doesn't have the same problem. The distortion ratio is the only path that divides.

**Fix:**

```javascript
distorted: img.naturalWidth > 0 && img.naturalHeight > 0 && (() => {
  const rect = img.getBoundingClientRect();
  return rect.width > 0 && rect.height > 0 && Math.abs(
    (img.naturalWidth / img.naturalHeight) -
    (rect.width / rect.height)
  ) > 0.05;
})(),
```

Guard both `img.naturalHeight > 0` and `rect.height > 0` before the ratio comparison. The IIFE avoids calling `getBoundingClientRect()` a third time inside this property.

---

## Medium

### M-01: Hover-then-eval uses `querySelector('button')` rather than the hovered element — may evaluate the wrong element

**File:** `agents/qa-tester.md:950-964`

**Issue:** Step 5 of DESIGN-04 instructs the agent to get an element ref from `playwright-cli snapshot`, then hover that ref via `playwright-cli hover <element-ref>`. But both the pre-hover and post-hover eval snippets hardcode `document.querySelector('button')`. If the page has multiple buttons (nav button, form submit button, icon button), the hovered ref may not correspond to the first `<button>` in the DOM. The pre- and post-hover reads are consistent with each other (both query the same element), but they may not reflect the element that was actually hovered — making the comparison meaningless for any button that isn't the first in DOM order.

**Fix:** Revise the procedure to query by a stable attribute of the target element rather than a positional selector. One practical approach is to read the element's accessible name from the snapshot output, then use `querySelector` with a more specific selector. An alternative the agent can always apply: add a note that if `querySelector('button')` returns a different element than the one hovered (visually verifiable via snapshot), the agent should use a more specific selector (e.g., `button[aria-label="..."]`, `button.primary`, or query by text content).

Suggested prose addition after step 5b:

> Note: `document.querySelector('button')` targets the first `<button>` in DOM order. If the hovered element is not the first button on the page, use a more specific selector — e.g., `button[aria-label="Submit"]` or `document.querySelector('.btn-primary')` — to ensure pre- and post-hover reads target the same element that was hovered.

### M-02: Confidence table describes `scrollWidth > clientWidth` only, but the probe also detects vertical overflow (`scrollHeight > clientHeight`)

**File:** `agents/qa-tester.md:1084` (confidence table row) vs. `agents/qa-tester.md:827` (probe condition)

**Issue:** The confidence table row reads: `Text overflowing container (scrollWidth > clientWidth) | High`. But the probe at line 827 flags elements where `el.scrollWidth > el.clientWidth || el.scrollHeight > el.clientHeight` — it detects both horizontal AND vertical overflow. The table description is incomplete, which means an agent reading the confidence table as its reference would undercount findings: a vertically overflowing element that doesn't overflow horizontally would be detected by the probe but the agent might not know to report it at High confidence because the table only mentions `scrollWidth > clientWidth`.

**Fix:** Update the confidence table row to reflect both axes:

```
| Text overflowing container (scrollWidth > clientWidth or scrollHeight > clientHeight) | High |
```

---

## Low

### L-01: Security filter regex is case-sensitive — CSS custom properties with uppercase prefix variants bypass the filter

**File:** `agents/qa-tester.md:927` and `agents/qa-tester.md:991`

**Issue:** The security filter `!prop.match(/^--(auth|token|key|secret)/)` uses no `i` flag, so it only matches lowercase prefix variations. A custom property named `--Auth-Header`, `--TOKEN`, or `--Secret-Key` would pass through the filter and appear in reports. CSS custom property names are case-sensitive by spec, so `--Auth` and `--auth` are technically distinct properties — a developer could define both. The filter as written only covers the lowercase forms.

In practice the risk is low: CSS convention strongly favors lowercase kebab-case for custom properties, and these properties are read from the live page (not injected). But the plan's threat model (T-09-01) states the intent is to prevent sensitive property names from appearing in reports — the current filter doesn't fully achieve that.

**Fix:** Add the `i` flag:

```javascript
!prop.match(/^--(auth|token|key|secret)/i)
```

Apply identically at both line 927 and line 991.

---

_Reviewed: 2026-04-19_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
