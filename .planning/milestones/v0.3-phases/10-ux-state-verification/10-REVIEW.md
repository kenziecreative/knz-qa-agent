---
phase: 10-ux-state-verification
reviewed: 2026-04-20T05:45:49Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - agents/qa-tester.md
findings:
  critical: 2
  warning: 2
  info: 1
  total: 5
status: issues_found
---

# Phase 10: Code Review Report

**Reviewed:** 2026-04-20T05:45:49Z
**Depth:** standard
**Files Reviewed:** 1
**Status:** issues_found

## Summary

Reviewed `agents/qa-tester.md` (1498 lines) with focus on the UX State Verification section added in Phase 10 (lines 1099-1419). The new content adds four structured verification procedures (STATE-01 through STATE-04) following the same dual-pattern methodology established in Phase 9's Design Verification. The writing quality, methodology structure, confidence calibration, and security notes are consistent with the existing agent document.

Five issues found: two command name mismatches that will cause runtime failures when the agent follows the instructions, one code/prose inconsistency that will produce false positive findings, one undocumented command used across multiple sections, and one missing null guard in template code.

## Critical Issues

### CR-01: `route-clear` command does not exist -- should be `unroute`

**File:** `agents/qa-tester.md:1137`
**Issue:** The STATE-01 cleanup steps use `playwright-cli route-clear "*/api/*"` on lines 1137, 1152, and 1168. However, the Core Commands reference (line 73) documents the cleanup command as `playwright-cli unroute [pattern]`. There is no `route-clear` command. When the agent follows these instructions, the cleanup will fail silently or error, leaving route mocks active. Per the STATE-01 cleanup mandate (step 7, line 1198), failing to clean up between state checks causes state bleed -- e.g., an empty-state route mock persisting into the loading state test.
**Fix:**
Replace all three occurrences:
```
- Cleanup: `playwright-cli route-clear "*/api/*"` (per D-04)
+ Cleanup: `playwright-cli unroute "*/api/*"` (per D-04)
```
Lines affected: 1137, 1152, 1168.

### CR-02: `localstorage-clear` command does not exist

**File:** `agents/qa-tester.md:1114`
**Issue:** Lines 1114 and 1171 use `playwright-cli localstorage-clear` to clear local storage as part of empty state and first-run state triggering. The Core Commands reference (line 67) documents only `localstorage-list`, `localstorage-get`, and `localstorage-set` -- there is no `localstorage-clear` command. The agent will encounter an error when attempting to clear storage, causing the empty state and first-run state tests to run against stale storage data rather than a clean slate.
**Fix:**
Either add `localstorage-clear` to the Core Commands reference if it exists in the playwright-cli implementation, or replace with a workaround using `run-code`:
```
- `playwright-cli run-code "async (page) => { await page.context().clearCookies(); }"` + `playwright-cli localstorage-clear`
+ `playwright-cli run-code "async (page) => { await page.context().clearCookies(); await page.evaluate(() => localStorage.clear()); }"`
```
Lines affected: 1114, 1171.

## Warnings

### WR-01: Cursor correctness eval does not implement its own exception rule

**File:** `agents/qa-tester.md:1260-1277`
**Issue:** The cursor correctness probe (STATE-03, step 1) queries `input, select, textarea` elements in its selector (line 1262) and marks them as incorrect when their cursor is not `pointer` (line 1272: `correct: getComputedStyle(el).cursor === 'pointer'`). However, line 1277 explicitly states these elements are exceptions that expect `text` or `auto` cursor. The code contradicts the prose -- every `<input>`, `<select>`, and `<textarea>` on the page will be reported as having an "incorrect" cursor, generating false positives at High confidence. This will produce noisy reports and erode trust in the agent's findings.
**Fix:**
Update the eval to exclude form input elements from the pointer-cursor check:
```javascript
playwright-cli eval "() => {
  const interactive = Array.from(
    document.querySelectorAll('button, [role=\"button\"], a[href], input, select, textarea, [tabindex]:not([tabindex=\"-1\"])')
  ).filter(el => {
    const r = el.getBoundingClientRect();
    return r.width > 0 && r.height > 0;
  });
  const formInputTags = ['INPUT', 'SELECT', 'TEXTAREA'];
  return interactive.map(el => ({
    tag: el.tagName,
    role: el.getAttribute('role'),
    text: el.textContent.trim().slice(0, 30),
    cursor: getComputedStyle(el).cursor,
    correct: formInputTags.includes(el.tagName)
      ? ['text', 'auto', 'default'].includes(getComputedStyle(el).cursor)
      : getComputedStyle(el).cursor === 'pointer'
  })).filter(el => !el.correct);
}"
```

### WR-02: STATE-02 default baseline eval has no null guard on querySelector

**File:** `agents/qa-tester.md:1213-1214`
**Issue:** The template code `document.querySelector('[target-selector]')` on line 1213 is a placeholder the agent must substitute with the actual selector. If the element does not exist on the page (selector mismatch, element removed by SPA routing, etc.), `querySelector` returns `null` and `getComputedStyle(null)` on line 1214 throws `TypeError: Failed to execute 'getComputedStyle' on 'Window': parameter 1 is not of type 'Element'`. The eval will crash instead of returning a meaningful "element not found" result. While the agent should substitute a valid selector, defensive coding in the template prevents opaque runtime errors.
**Fix:**
Add a null guard:
```javascript
playwright-cli eval "() => {
  const el = document.querySelector('[target-selector]');
  if (!el) return { error: 'element not found', selector: '[target-selector]' };
  const s = getComputedStyle(el);
  return {
    bg: s.backgroundColor, color: s.color, opacity: s.opacity,
    border: s.borderColor, outline: s.outlineColor,
    cursor: s.cursor, transform: s.transform,
    boxShadow: s.boxShadow, textDecoration: s.textDecoration
  };
}"
```

## Info

### IN-01: `run-code` command used but not documented in Core Commands

**File:** `agents/qa-tester.md:978`
**Issue:** `playwright-cli run-code` is used on lines 978, 980, 1014, 1114, 1171, 1296, and 1327 to execute async Playwright page API code (cookie clearing, media emulation, waitForTimeout, mouse.wheel). This command is distinct from `playwright-cli eval` (which runs synchronous expressions) but is not listed in the Core Commands reference (lines 34-78). This predates Phase 10 -- it was introduced in Phase 9's Design Verification section -- but the UX State Verification section adds more usages, making the gap more prominent. The agent may fail to use `run-code` correctly for novel cases since it has no documented syntax or semantics to reference.
**Fix:**
Add `run-code` to the Core Commands section under JavaScript:
```markdown
**JavaScript:**
- `playwright-cli eval <expression> [element]` -- evaluate JS on page or element
- `playwright-cli run-code <async-function>` -- run async Playwright page API code (e.g., emulateMedia, waitForTimeout, mouse.wheel)
```

---

_Reviewed: 2026-04-20T05:45:49Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
