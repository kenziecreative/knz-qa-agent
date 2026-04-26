# Phase 10: UX State Verification - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 10-ux-state-verification
**Areas discussed:** State existence detection, Interaction state matrix, Feedback & scroll behavior, Animation & transition quality

---

## State Existence Detection

| Option | Description | Selected |
|--------|-------------|----------|
| Active triggering (Recommended) | Agent manipulates app state to force each UI state (empty, loading, error, first-run) then verifies rendering. Falls back to passive DOM probing when no trigger guidance exists. | ✓ |
| Passive probing only | Agent queries DOM for state-indicator patterns without triggering states. Faster but only verifies markup exists, not that states actually fire. | |

**User's choice:** Active triggering
**Notes:** Per-state trigger heuristics: empty = clear storage + mock empty API; loading = intercept + delay; error = 5xx route; first-run = clear all storage + cookies. Passive probing as fallback.

---

## Interaction State Matrix

| Option | Description | Selected |
|--------|-------------|----------|
| Interaction-first (Recommended) | Trigger states via hover/click/Tab like a user, eval to read computed styles. Active state captured via screenshot only. Extends the existing DESIGN-04 pattern. | ✓ |
| Programmatic injection | Force CSS classes / aria attributes via eval to hold states open. Tests injected DOM rather than real app behavior. | |

**User's choice:** Interaction-first
**Notes:** Components selected from accessibility tree snapshot. Active (mousedown) state is visual-judgment-only due to timing constraints.

---

## Feedback & Scroll Behavior

### Cursor Correctness
No options presented — `getComputedStyle(el).cursor` eval probe is the only viable option (screenshots don't render cursors in headless Playwright).

### Toast/Notification Timing

| Option | Description | Selected |
|--------|-------------|----------|
| Presence + disappearance (Recommended) | Assert toast appears after trigger, then auto-dismisses. Duration measurement only when spec explicitly declares a timing threshold. | ✓ |
| Always measure duration | Use performance.now() to bracket toast on-screen time against a default threshold. More precise but inherently flaky in headless environments. | |

**User's choice:** Presence + disappearance
**Notes:** Duration measurement opt-in only when spec declares threshold.

### Sticky Header
No options presented — two-step eval (computed position + scroll + bounding rect) is the standard approach, screenshot as supplement.

---

## Animation & Transition Quality

| Option | Description | Selected |
|--------|-------------|----------|
| CSS inference + loading check (Recommended) | Eval probes for transition/animation properties, layout-triggering warnings, duration consistency. Screenshots only for confirming loading animations exist during async ops. Honest about Playwright's ceiling. | ✓ |
| Add screenshot-based smoothness judgment | Attempt to capture mid-animation screenshots for visual smoothness assessment on top of CSS inference. Higher noise, unreliable timing. | |

**User's choice:** CSS inference + loading check
**Notes:** No FPS API in Playwright — structural quality checks are the ceiling. Complements existing reduced-motion checks.

---

## Claude's Discretion

- Exact JS evaluate snippets for state detection heuristics
- Threshold values for sticky header position tolerance
- Methodology section structure and ordering in qa-tester.md
- Interactive element prioritization in state matrix sweep
- State cleanup implementation details

## Deferred Ideas

None — discussion stayed within phase scope
