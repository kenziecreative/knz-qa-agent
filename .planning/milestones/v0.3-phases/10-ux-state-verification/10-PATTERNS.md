# Phase 10: UX State Verification - Pattern Map

**Mapped:** 2026-04-20
**Files analyzed:** 1 (modified)
**Analogs found:** 1 / 1

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `agents/qa-tester.md` | agent (methodology section) | event-driven (trigger → eval → report) | `agents/qa-tester.md` lines 749-1098 (Design Verification) | exact — same file, same section type, same dual eval+visual pattern |

## Pattern Assignments

### `agents/qa-tester.md` — new `### UX State Verification` section

**Analog:** `agents/qa-tester.md` — `### Design Verification` (lines 749-1098)

This is an in-file addition. The new section is a peer to `### Design Verification` and follows the same
subsection structure, naming conventions, and dual-pattern methodology established in Phase 9.

---

#### Section header pattern (lines 749-756):

```markdown
### Design Verification

Tier 2 visual design verification — runs when `design-verification` is listed in the spec's `## Visual Focus` section. All five areas use the same dual pattern: `playwright-cli eval` for deterministic DOM/CSS probes (high confidence) + `playwright-cli screenshot` + visual judgment for perceptual quality (medium confidence).

If no `## Design Reference` section was forwarded in the test assignment, skip the Mockup Comparison procedure below and note "no design reference provided — mockup comparison skipped." Continue to Typography Verification through Cross-Page Consistency.

**Security note:** When reading CSS custom properties via `playwright-cli eval`, do not include property names beginning with `--auth`, `--token`, `--key`, or `--secret` in reports.
```

**Apply to new section:** Open with "Tier 2 UX state verification — runs when `ux-states` is listed in the spec's `## Visual Focus` section." Include the same security note (carry-forward mandated by RESEARCH.md security domain) and the same active-triggering caveat that this is for test environments only.

---

#### Subsection naming pattern (lines 757, 793, 852, 910, 1021):

```markdown
#### Mockup Comparison (DESIGN-01)
#### Typography Verification (DESIGN-02)
#### Image & Media Quality (DESIGN-03)
#### Color & Theme Consistency (DESIGN-04)
#### Cross-Page Consistency (DESIGN-05)
```

**Apply to new section:** Four subsections named after requirements IDs:
- `#### State Existence (STATE-01)`
- `#### Interaction State Matrix (STATE-02)`
- `#### Interactive Feedback Quality (STATE-03)`
- `#### Animation & Transition Quality (STATE-04)`

---

#### Numbered step list pattern within each subsection (lines 796-817, 856-908, 916-1018):

Each DESIGN subsection uses a numbered procedural list, often with nested lettered sub-steps (a, b, c...), inline code fences for eval commands, and bold labels for findings ("Flag as **High confidence**"). The new STATE subsections must follow this exact format.

Example from DESIGN-02 (lines 796-817):
```markdown
1. Run font load check:
   ```
   playwright-cli eval "async () => { ... }"
   ```
2. If the spec or a design tokens file declares expected font families, compare ...
3. If `document.fonts.check()` returns false for an expected font, flag as **High confidence** ...
```

---

#### Eval probe inline code format (lines 800-808, 824-839, 862-895, 918-937, 1034-1058):

All eval commands in the Design Verification section are written as fenced code blocks (no language tag) containing the full `playwright-cli eval` invocation with the JS function inline. Multi-line JS is indented inside the string:

```markdown
```
playwright-cli eval "() => {
  const el = document.querySelector('button');
  const s = getComputedStyle(el);
  return { bg: s.backgroundColor, color: s.color, border: s.borderColor, outline: s.outlineColor };
}"
```
```

**Apply to new section:** STATE-01 through STATE-04 eval probes must use this exact fence format.

---

#### Hover/focus eval-after-interaction pattern (DESIGN-04, lines 943-968):

This is the exact pattern D-05 extends for the full state matrix sweep (STATE-02):

```markdown
   a. Identify the target element via `playwright-cli snapshot` (get element ref)
   b. Read default state colors:
      ```
      playwright-cli eval "() => {
        const el = document.querySelector('button');
        const s = getComputedStyle(el);
        return { bg: s.backgroundColor, color: s.color, border: s.borderColor, outline: s.outlineColor };
      }"
      ```
   c. Trigger hover state: `playwright-cli hover <element-ref>`
   d. **Immediately** read hover state colors (do NOT navigate or click between hover and eval):
      ```
      playwright-cli eval "() => {
        const el = document.querySelector('button');
        const s = getComputedStyle(el);
        return { bg: s.backgroundColor, color: s.color, border: s.borderColor, outline: s.outlineColor };
      }"
      ```
   e. Compare default vs hover — if colors are identical, the element has no visible hover state. Flag as **Medium confidence** ...
   f. Repeat for focus state: `playwright-cli click <element-ref>` or Tab to focus, then eval.
```

**Apply to STATE-02:** Extend the same "identify ref via snapshot → baseline eval → trigger (hover/Tab/click) → immediately eval → compare" sequence. The state matrix adds disabled (DOM query for `[disabled]`/`aria-disabled`), active (screenshot-only per D-08), and loading/error/success states via route mock.

---

#### Security note carry-forward (lines 927-941, 991-993):

The security note appears twice in DESIGN-04 — once in the section header block (lines 754-755) and once inline in the CSS custom property eval:

```markdown
if (prop.startsWith('--') && !prop.match(/^--(auth|token|key|secret)/)) {
```

**Apply to new section:** Any eval probe in STATE-02 through STATE-04 that reads CSS custom properties or computed styles must include the `!prop.match(/^--(auth|token|key|secret)/)` guard. The section-level security note in the header must also be repeated per the RESEARCH.md security carry-forward mandate.

---

#### Confidence inline labeling pattern (lines 841, 850, 895, 908, 965, 1019):

Confidence declarations appear at the end of each numbered step group as a standalone line:

```markdown
Confidence: eval-detected overflow = High. Visual readability judgment = Medium.
```

```markdown
Confidence: broken/upscaled via eval = High. Distortion with object-fit: cover = note only. Missing lazy-load placeholder = Medium. Visual quality judgment = Medium.
```

```markdown
Confidence: Token value differs from expected = High. Hover/focus colors identical to default = Medium. Dark mode visual issue (icon invisible) = Medium. Dark mode not supported = observation only (not a finding).
```

**Apply to new section:** Each STATE subsection ends with a "Confidence: [eval check] = High. [visual judgment check] = Medium." line summarizing the confidence levels for that subsection's findings.

---

#### Reporting table pattern (lines 1076-1097):

The Design Verification section ends with `#### Reporting Design Verification Findings` — a prose intro sentence followed by a two-column markdown table `| Finding type | Confidence |`:

```markdown
#### Reporting Design Verification Findings

Present design verification findings grouped by confidence level, consistent with the existing accessibility findings format. Use the "live page shows X vs reference shows Y" format for mockup comparison findings (DESIGN-01).

| Finding type | Confidence |
| --- | --- |
| Broken image (complete but naturalWidth is 0) | High |
| Font not loaded (document.fonts.check returns false) | High |
...
| Cross-page computed style deviation (inferred) | Medium |
```

**Apply to new section:** New section ends with `#### Reporting UX State Verification Findings` using the same table format. Rows cover STATE-01 through STATE-04 finding types, ordered High → Medium.

---

#### Visual Focus stub to remove (line 747):

```markdown
**Note:** Tier 2 visual methodology for `ux-states`, `layout-integrity`, and `performance-responsive` will be added in Phases 10-12. If a Visual Focus section requests those areas before their phases ship, note in the report: "Visual Focus requested for [areas] — full methodology pending Phase [N]."
```

**Action:** After inserting the UX State Verification section, update this line to remove `ux-states` from the pending list (leaving `layout-integrity` and `performance-responsive`). If both are still pending after this phase, the note remains for Phase 11-12.

---

#### Accessibility section structural template (lines 240-375):

The `### Structured Accessibility` section provides an additional structural template for tiered methodology sections. Key observation: it uses the same numbered-step format and ends with `### Reporting Accessibility Findings` table. The Tier 2 accessibility checks are organized into four `###` subsections (Focus Management, Page Structure, Interactive Elements, Form Accessibility), each with `####`-level checks inside — exactly the structure Design Verification and the new UX State section follow.

---

## Shared Patterns

### Security: CSS Custom Property Filter
**Source:** `agents/qa-tester.md` lines 927, 991-993
**Apply to:** All eval probes in STATE-02 and STATE-04 that iterate CSS properties
```javascript
if (prop.startsWith('--') && !prop.match(/^--(auth|token|key|secret)/)) {
```
This guard is not optional — RESEARCH.md security domain explicitly mandates carry-forward.

### Security: Active Triggering Environment Caveat
**Source:** RESEARCH.md security domain (first-run trigger destroys real user data — DoS pattern)
**Apply to:** STATE-01 section header
Add a note that active triggering (storage clear, cookie delete, route mocking) is for test environments only and must not be run against production data.

### Dual Eval + Visual Pattern
**Source:** `agents/qa-tester.md` lines 749-752 (Design Verification header), lines 841, 850, 908, 965, 1019 (inline Confidence lines)
**Apply to:** All four STATE subsections
Every measurable/deterministic check uses `playwright-cli eval` (High confidence). Every perceptual/subjective check uses `playwright-cli screenshot` + visual judgment (Medium confidence). The dual pattern is non-negotiable per D-17.

### State Cleanup Mandate
**Source:** CONTEXT.md D-04, RESEARCH.md Pitfall 1
**Apply to:** STATE-01 section
After each active state trigger, the methodology must mandate clearing route mocks before the next trigger. Express as an explicit numbered step: "Remove route mock before proceeding to next state check."

### Confidence Inline Line Format
**Source:** `agents/qa-tester.md` lines 841, 850, 908, 1019
**Apply to:** End of each STATE-0x subsection
```markdown
Confidence: [eval check] = High. [visual judgment check] = Medium.
```

### Accessibility Tree Snapshot for Element Discovery
**Source:** `agents/qa-tester.md` lines 945-948 (DESIGN-04 step a)
**Apply to:** STATE-02 (interaction matrix sweep)
Use `playwright-cli snapshot` to get element refs before triggering interaction states. Same pattern as DESIGN-04.

---

## No Analog Found

None — the single file modified has a direct, exact analog within itself (Design Verification section). All patterns are sourced from `agents/qa-tester.md`.

---

## Insertion Points

For the planner's reference, the three specific edit locations in `agents/qa-tester.md`:

| Edit | Location | Action |
|------|----------|--------|
| Insert new `### UX State Verification` section | After line 1098 (end of Design Verification reporting table), before line 1099 (`### Design Reference`) | Insert ~150-200 lines of new methodology |
| Update Visual Focus stub note | Line 747 | Remove `ux-states` from the "pending Phase 10" list |
| Extend Confidence Summary Table | Lines 1080-1097 (the reporting table) | Add STATE-* rows: empty/loading/error/first-run state missing = High; interaction state with no visual change = High (eval-confirmed); cursor incorrect (eval) = High; sticky header not sticking (eval) = High; active state no visible change = Medium; toast stuck = Medium; CSS layout-triggering transitions = Medium; missing transition on interactive element = Medium |

---

## Metadata

**Analog search scope:** `agents/qa-tester.md` (sole implementation file for this phase)
**Files scanned:** 1 (the modified file serves as its own analog)
**Pattern extraction date:** 2026-04-20
