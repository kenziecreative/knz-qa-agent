---
phase: 11-layout-content-integrity
reviewed: 2026-04-20T06:15:00Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - agents/qa-tester.md
findings:
  critical: 0
  warning: 4
  info: 3
  total: 7
status: issues_found
---

# Phase 11: Code Review Report

**Reviewed:** 2026-04-20T06:15:00Z
**Depth:** standard
**Files Reviewed:** 1
**Status:** issues_found

## Summary

Reviewed `agents/qa-tester.md` (1881 lines), the core QA agent system prompt. This is an agent prompt file (Markdown with embedded code snippets), not executable source code -- so the review focuses on correctness of instructions the agent will follow, inconsistencies that could cause agent confusion or test failures, and code quality in the embedded JavaScript eval snippets.

The file is well-structured with clear methodology, consistent confidence frameworks, and thorough coverage across accessibility, design verification, UX state, and the newly added layout/content integrity sections. The new Phase 11 content (LAYOUT-01 through LAYOUT-04, lines 1421-1801) follows the established patterns from earlier phases cleanly.

Four warnings were found: a command naming inconsistency between the reference section and the methodology body that could cause the agent to use a non-existent command, a bug in the cursor correctness eval that flags `input`/`select`/`textarea` elements despite the prose saying to exclude them, a missing `run-code` command in the Core Commands reference, and a command name inconsistency where `navigate` is used in one place but the documented command is `goto`. Three info-level items were also noted.

## Warnings

### WR-01: Command Name Inconsistency -- `unroute` vs `route-clear`

**File:** `agents/qa-tester.md:73` and `agents/qa-tester.md:1137`
**Issue:** The Core Commands reference section (line 73) documents `playwright-cli unroute [pattern]` as the command to remove route interceptions. However, every usage in the methodology body (lines 1137, 1152, 1168, 1689) uses `playwright-cli route-clear` instead. When the agent follows the methodology sections (which is how it executes tests), it will use `route-clear`. If it instead references the Core Commands section for the cleanup command, it will use `unroute`. One of these is wrong -- the agent will issue a command that does not exist, causing route mocks to leak between test states.
**Fix:** Determine which command name `playwright-cli` actually implements and make both the reference and methodology consistent. If `route-clear` is correct:
```markdown
# Line 73, change:
- `playwright-cli route-list` / `playwright-cli unroute [pattern]`
# to:
- `playwright-cli route-list` / `playwright-cli route-clear [pattern]`
```

### WR-02: Cursor Correctness Eval Includes Elements the Prose Excludes

**File:** `agents/qa-tester.md:1260-1277`
**Issue:** The eval code on lines 1260-1273 queries `input, select, textarea` and flags them as incorrect when their cursor is not `pointer`. Line 1277 then says "Exception: `input`, `select`, `textarea` expect `text` or `auto` cursor for their body -- only check for `pointer` on buttons, links, and custom interactive roles." The eval code contradicts the prose exception -- it will produce false positive findings for every text input, select, and textarea on the page. The agent is instructed to flag these as **High confidence** issues, creating noise in reports.
**Fix:** Filter out form field elements in the eval itself so the agent does not need to reconcile contradictory instructions:
```javascript
playwright-cli eval "() => {
  const interactive = Array.from(
    document.querySelectorAll('button, [role=\"button\"], a[href], input, select, textarea, [tabindex]:not([tabindex=\"-1\"])')
  ).filter(el => {
    const r = el.getBoundingClientRect();
    return r.width > 0 && r.height > 0;
  });
  const formFields = new Set(['INPUT', 'SELECT', 'TEXTAREA']);
  return interactive.map(el => ({
    tag: el.tagName,
    role: el.getAttribute('role'),
    text: el.textContent.trim().slice(0, 30),
    cursor: getComputedStyle(el).cursor,
    correct: formFields.has(el.tagName)
      ? ['text', 'auto', 'default', 'pointer'].includes(getComputedStyle(el).cursor)
      : getComputedStyle(el).cursor === 'pointer'
  })).filter(el => !el.correct);
}"
```

### WR-03: `run-code` Command Used But Not Documented in Core Commands

**File:** `agents/qa-tester.md:34-78` (Core Commands section)
**Issue:** The `playwright-cli run-code` command is used extensively in the methodology sections (lines 978, 980, 1014, 1114, 1171, 1296, 1327, 1645) for operations requiring `async (page) =>` patterns (dark mode toggling, cookie clearing, scroll testing, content injection). However, `run-code` does not appear anywhere in the Core Commands reference (lines 34-78). The agent may not know this command exists when generating test steps from the reference section, or may be confused about its syntax since it lacks a reference entry.
**Fix:** Add `run-code` to the Core Commands reference:
```markdown
**JavaScript:**
- `playwright-cli eval <expression> [element]` — evaluate JS on page or element
- `playwright-cli run-code <code>` — execute async code with page object (for operations requiring await, like emulateMedia, clearCookies, waitForTimeout)
```

### WR-04: `navigate` Command Used But Does Not Exist -- Should Be `goto`

**File:** `agents/qa-tester.md:1681`
**Issue:** Line 1681 instructs the agent to "reload the page with `playwright-cli navigate [same url]`" after LAYOUT-03 injection checks. The documented navigation command is `playwright-cli goto <url>` (line 37). There is no `navigate` command in the Core Commands reference. If the agent follows this instruction literally, the command will fail and the page will not be reloaded, leaving injected test content in the DOM and potentially causing false positives in LAYOUT-04 placeholder detection.
**Fix:**
```markdown
# Line 1681, change:
reload the page with `playwright-cli navigate [same url]`
# to:
reload the page with `playwright-cli goto [same url]` (or `playwright-cli reload`)
```

## Info

### IN-01: `document.querySelector('button')` Hardcoded in DESIGN-04 Hover/Focus Example

**File:** `agents/qa-tester.md:951-964`
**Issue:** The hover/focus color state verification examples (DESIGN-04 steps 5b-5e) use `document.querySelector('button')` as a hardcoded selector. The surrounding prose says "For key interactive elements (buttons, links, inputs)" and refers to using snapshot element refs, but the eval code always queries the first button on the page. This is a code example rather than a reusable snippet -- the agent will likely adapt it, but the mismatch between prose ("get element ref from snapshot") and code ("hardcode `button`") could cause the agent to check only the first button rather than sweeping all key interactive elements.
**Fix:** Add a comment or use a placeholder selector to clarify the agent should substitute:
```javascript
// Replace '[target-selector]' with the actual element selector from snapshot
const el = document.querySelector('[target-selector]');
```
This is consistent with the STATE-02 pattern on line 1212 which correctly uses `'[target-selector]'`.

### IN-02: Content Clipping Check Queries All Elements -- May Produce Noisy Results

**File:** `agents/qa-tester.md:1532-1546`
**Issue:** The LAYOUT-01 content clipping check (step 4) runs `document.querySelectorAll('*')` and flags any visible element where `scrollWidth > clientWidth`. Many elements legitimately have overflow by design (scrollable containers, code blocks, carousels, elements with `overflow-x: auto` or `overflow-x: scroll`). The DESIGN-02 text overflow check (lines 825-841) scopes to text elements only. Querying all elements will produce significantly more false positives. The methodology does not mention filtering out elements with intentional `overflow: auto|scroll` styles.
**Fix:** Consider adding an `overflow` style filter to reduce noise:
```javascript
if (el.scrollWidth > el.clientWidth || el.scrollHeight > el.clientHeight) {
  const s = getComputedStyle(el);
  // Skip elements with intentional scrollable overflow
  if (s.overflowX === 'auto' || s.overflowX === 'scroll' ||
      s.overflowY === 'auto' || s.overflowY === 'scroll') return;
  overflowing.push({ ... });
}
```

### IN-03: Security Note About CSS Custom Properties Repeated Three Times

**File:** `agents/qa-tester.md:755`, `agents/qa-tester.md:1103`, `agents/qa-tester.md:1425`
**Issue:** The security note "When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports" appears identically three times (in Design Verification, UX State Verification, and Layout & Content Integrity). The regex filter `!prop.match(/^--(auth|token|key|secret)/)` is also duplicated in the eval snippets. This is not a bug -- the repetition ensures the agent sees the note regardless of which section it enters -- but it could be consolidated into a single "Security Rules for Eval Probes" section referenced by all three, reducing file size and maintenance burden.
**Fix:** Optional consolidation. Add a named security rule once (e.g., after the "Security Observations" section around line 630) and reference it from each Tier 2 section. Current approach is safe if slightly verbose.

---

_Reviewed: 2026-04-20T06:15:00Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
