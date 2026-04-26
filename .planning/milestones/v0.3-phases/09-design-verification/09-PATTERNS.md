# Phase 9: Design Verification - Pattern Map

**Mapped:** 2026-04-19
**Files analyzed:** 1 (single file modified — `agents/qa-tester.md`)
**Analogs found:** 1 / 1

---

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `agents/qa-tester.md` | agent (methodology doc) | event-driven (spec trigger → methodology execution) | `agents/qa-tester.md` lines 186-372 (Structured Accessibility + Confidence Table) | exact — same file, same pattern extended |

---

## Pattern Assignments

### `agents/qa-tester.md` — Design Verification methodology sections

**What changes:** Three distinct edits to this one file:
1. Add `### Design Verification` section (five subsections + confidence table) after line 747
2. Remove placeholder phrase at line 747: `"Visual Focus requested for [areas] — full methodology pending Phase [N]."`
3. Remove placeholder phrase at line 763: `"The actual comparison methodology (how to compare live screenshots against reference images) is implemented in Phase 9."`
4. Extend the Accessibility confidence table (lines 351-372) with design verification rows — OR add a separate design verification confidence table inside the new section

---

## Pattern 1: Tier 2 Section Structure

**Analog:** `agents/qa-tester.md`, lines 186-372 — `## Structured Accessibility`

This is the exact structural pattern to mirror. The Design Verification section is the visual equivalent of Structured Accessibility.

**Section header pattern** (lines 186-188):
```markdown
## Structured Accessibility

Tier 2 accessibility testing — deeper, structured methodology that goes beyond the baseline checks above. These checks are performed on every page/flow unless the spec explicitly narrows scope. All testing uses only Playwright's built-in APIs (no external a11y libraries).
```

**Apply to Design Verification as:**
```markdown
### Design Verification

Tier 2 visual design verification — runs when `design-verification` is listed in the spec's `## Visual Focus` section. All five areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).

If no `## Design Reference` section was forwarded in the test assignment, skip DESIGN-01 and note "no design reference provided — mockup comparison skipped." Continue to DESIGN-02 through DESIGN-05.
```

---

## Pattern 2: Numbered Procedure Steps per Check

**Analog:** `agents/qa-tester.md`, lines 192-204 — Modal/Dialog Focus (A11Y-01, A11Y-02, A11Y-03)

The canonical procedure format — numbered steps, specific commands, immediate confidence assignment:

```markdown
#### Modal/Dialog Focus (A11Y-01, A11Y-02, A11Y-03)

When testing any modal, dialog, or drawer:

1. Open the modal (click trigger element)
2. Verify focus moved INTO the modal — press Tab once and check that `document.activeElement` is inside an element with `role="dialog"` (or the modal container). If focus stayed on the trigger or body, flag as High confidence issue.
3. Verify focus is TRAPPED — press Tab enough times to cycle through all focusable elements in the modal AT LEAST TWICE. After each Tab, check that `document.activeElement` is still inside the modal. If focus escapes to elements outside the modal, flag as High confidence issue.
4. Close the modal (press Escape or click close button)
5. Verify focus RETURNED to the trigger element — use `toBeFocused()` on the trigger. If focus went to body or another element, flag as Medium confidence issue.
```

**Apply to each DESIGN subsection as:** numbered steps, `playwright-cli eval` command inline, confidence level stated per finding, common patterns listed at end of step.

---

## Pattern 3: Visual Judgment with Confidence Assignment

**Analog:** `agents/qa-tester.md`, lines 277-284 — Color as Sole Indicator (A11Y-11)

The canonical visual judgment confidence pattern — when eval can't give a deterministic answer, the agent uses screenshot + judgment and frames as Medium:

```markdown
#### Color as Sole Indicator (A11Y-11)

When testing state changes (hover, active, selected, error, disabled):

- Take screenshots of the element in both states
- Check whether the state change is communicated by MORE than just color — look for: text labels, icons, underlines, borders, opacity changes, shape changes
- This is a visual judgment call — Playwright cannot compute color differences programmatically without axe-core
- Frame findings as Medium confidence observations
- Common patterns to flag: links distinguished from text only by color (no underline), error states shown only by red text (no icon or border), selected tabs with only a color change

Confidence: Color-only state indicator = Medium confidence (requires visual judgment, cannot be mechanically verified).
```

**Apply to DESIGN-01 mockup comparison** — all five categories (typography, spacing, color, imagery, layout) are visual judgment = Medium confidence unless an eval probe confirms deviation (High).

---

## Pattern 4: emulateMedia for Media State Changes

**Analog:** `agents/qa-tester.md`, lines 286-297 — Reduced Motion (A11Y-12)

The canonical `emulateMedia` + screenshot + reset pattern:

```markdown
#### Reduced Motion (A11Y-12)

On pages with animations, carousels, or loading spinners:

1. Use `page.emulateMedia({ reducedMotion: 'reduce' })` to set the preference
2. Take a screenshot to visually confirm animations are suppressed
3. Optionally check computed styles on animated elements: `animation-duration` and `transition-duration` should be `0s` or very short
4. Important: Check the VISUALLY ANIMATED element specifically, not its parent container
5. After checking, reset with `page.emulateMedia({ reducedMotion: 'no-preference' })` or `null`
6. If page has no visible animations, skip this check (not applicable)

Confidence: Animations still running with reduced motion = High confidence.
```

**Apply to DESIGN-04 dark mode** — replace `reducedMotion: 'reduce'` with `colorScheme: 'dark'`, add step to read CSS variables before/after via `playwright-cli run-code`, reset at end. Note: use `run-code` (not `eval`) because `emulateMedia` is a Playwright API, not a browser API.

---

## Pattern 5: Depth Toggle + Section Absent Behavior

**Analog:** `agents/qa-tester.md`, lines 719-731 — Accessibility Focus Trigger

The canonical absent/present/present-with-areas behavior description — Design Verification already has its trigger stub at lines 733-747 using this exact pattern. New methodology sections go inside that trigger's scope.

```markdown
### Accessibility Focus Trigger

The `## Accessibility Focus` section in a spec controls accessibility testing depth:

- **Section ABSENT:** Run only baseline Tier 1 checks...
- **Section PRESENT with no items listed:** Run FULL Tier 2 structured accessibility checks...
- **Section PRESENT with specific areas listed:** Run Tier 2 checks for only the listed areas.

**This is a depth toggle, not an on/off switch.**
```

**Apply to Design Verification:** already implemented in Visual Focus Trigger (lines 733-747). No changes needed to the trigger itself — only the methodology body after it.

---

## Pattern 6: Confidence Summary Table

**Analog:** `agents/qa-tester.md`, lines 351-372 — Reporting Accessibility Findings

The canonical two-column confidence table format:

```markdown
### Reporting Accessibility Findings

| Finding type | Confidence |
| --- | --- |
| Focus not moving into modal | High |
| Focus escaping modal (trap broken) | High |
| No h1 or multiple h1s | High |
| ...
| Color as sole state indicator | Medium |
```

**Apply to Design Verification:** Add a `#### Design Verification Confidence Table` subsection inside the new `### Design Verification` section. Rows to include:

| Finding type | Confidence |
| --- | --- |
| Broken image (complete=true, naturalWidth=0) | High |
| Font not loaded (document.fonts.check() returns false) | High |
| Text overflowing container (scrollWidth > clientWidth) | High |
| CSS custom property value differs from expected token | High |
| Image upscaled beyond natural resolution | High |
| Aspect ratio distortion >5% | Medium |
| Visual spacing imbalance vs reference | Medium |
| Typography mismatch vs reference (visual judgment) | Medium |
| Color mismatch vs reference (visual judgment) | Medium |
| Imagery mismatch vs reference (visual judgment) | Medium |
| Layout deviation vs reference (visual judgment) | Medium |
| Cross-page computed style deviation | Medium |
| Hover/focus color state not matching expected | Medium |
| Dark mode CSS variables not switching | Medium |
| Lazy-load placeholder absent | Medium |

---

## Shared Patterns

### eval-first, screenshot-supplement (dual-mode)

**Source:** Decisions D-15 + existing Tier 1/Tier 2 model throughout `agents/qa-tester.md`
**Apply to:** All five DESIGN subsections

Every DESIGN check follows this sequence:
1. `playwright-cli eval` for deterministic DOM/CSS probe → High confidence if deviation found
2. `playwright-cli screenshot` → agent visual judgment → Medium confidence for perceptual findings

Confidence assignment rule: "eval result differs from expected = High confidence (deterministic). Visual judgment shows inconsistency = Medium confidence (perceptual)."

### hover-then-eval sequencing

**Source:** CONTEXT.md decision D-11; analog `agents/qa-tester.md` line 200 (activeElement check after Tab press — same principle: trigger state first, read after)
**Apply to:** DESIGN-04 hover/focus color verification

```
Step 1: playwright-cli hover <element-ref>
Step 2 (immediately after, no navigation): playwright-cli eval "() => ({ ... getComputedStyle ... })"
Step 3: playwright-cli screenshot
```

Never read computed styles before triggering the interaction state.

### run-code vs eval distinction

**Source:** RESEARCH.md pitfall 3; existing `page.emulateMedia` pattern at `agents/qa-tester.md` line 290
**Apply to:** Any methodology step using Playwright API (not browser API)

Rule: Use `playwright-cli eval` for browser/DOM APIs (`getComputedStyle`, `naturalWidth`, `document.fonts`). Use `playwright-cli run-code` for Playwright API calls (`emulateMedia`, `route`, viewport resize).

### Graceful skip on missing optional data

**Source:** `agents/qa-tester.md` lines 759-761 — Design Reference section absent behavior; line 296 — A11Y-12 "if page has no visible animations, skip this check"
**Apply to:** DESIGN-01 (no Design Reference), DESIGN-05 (single-page spec)

Pattern: "If [prerequisite] is absent, skip [check] and note '[reason] — [check name] skipped.' Continue to next check."

---

## No Analog Found

No files in this phase lack a codebase analog. All methodology patterns derive directly from `agents/qa-tester.md` Structured Accessibility section (lines 186-372) and the Visual Focus Trigger stub (lines 733-763).

---

## Edit Map (Precise Line Targets)

| Edit | Location | Action |
|------|----------|--------|
| Remove "pending Phase N" note | `agents/qa-tester.md` line 747 | Delete the sentence: `"Visual Focus requested for [areas] — full methodology pending Phase [N]."` |
| Add `### Design Verification` section | After line 747, before line 749 | Insert new section (five subsections + confidence table) |
| Remove "implemented in Phase 9" phrase | `agents/qa-tester.md` line 763 | Delete the sentence: `"The actual comparison methodology (how to compare live screenshots against reference images) is implemented in Phase 9."` Replace with pointer to the `### Design Verification` section above |

---

## Metadata

**Analog search scope:** `agents/qa-tester.md` (entire file — single primary artifact for this project)
**Files scanned:** 1 (the only file being modified)
**Pattern extraction date:** 2026-04-19
