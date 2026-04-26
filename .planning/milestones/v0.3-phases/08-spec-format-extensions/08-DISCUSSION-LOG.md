# Phase 8: Spec Format Extensions - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-19
**Phase:** 08-spec-format-extensions
**Areas discussed:** Visual Focus tiering, Design Reference format, Browser engine selection, Spec doc & backward compat

---

## Visual Focus Tiering

| Option | Description | Selected |
|--------|-------------|----------|
| Phase-aligned | 4 named areas mapped 1:1 to Phase 9-12 clusters: design-verification, ux-states, layout-integrity, performance-responsive | ✓ |
| Mode-aligned | 4 areas named by verification mode: design-reference, ui-states, layout-content, responsive-perf | |

**User's choice:** Phase-aligned (Recommended)
**Notes:** Mirrors Accessibility Focus pattern exactly. Area names map to requirements groups for clean traceability.

---

## Design Reference Format

| Option | Description | Selected |
|--------|-------------|----------|
| Keyed subsections per viewport | Subsections per viewport (### desktop, ### mobile), state paths as nested bullet items — mirrors ## Environments pattern | ✓ |
| Flat path list with inline labels | Simple bullet list with freeform labels like "desktop default: path.png" | |

**User's choice:** Keyed subsections (Recommended)
**Notes:** Unambiguous viewport-to-image mapping. Missing viewport = skip, never block.

---

## Browser Engine Selection

| Option | Description | Selected |
|--------|-------------|----------|
| Bullet list | Simple bullet list of engine names (chromium, webkit, firefox) — matches ## Viewports pattern | ✓ |
| Key-value pairs | engines: chromium, webkit — could carry per-engine config | |

**User's choice:** Bullet list (Recommended)
**Notes:** Maps directly to playwright-cli --browser flag. Absent = Chromium only.

---

## Spec Doc & Backward Compatibility

| Option | Description | Selected |
|--------|-------------|----------|
| Follow v2 pattern exactly | Individual Section Reference entries + examples per section + one-line Backward Compatibility update | ✓ |
| v2 pattern + combined example | Same as above, plus combined example block showing all three sections together | |

**User's choice:** Follow v2 pattern (Recommended)
**Notes:** Sections are independently useful. Combined example creates maintenance drift risk.

---

## Claude's Discretion

- Example content wording in SPEC-FORMAT.md
- Ordering of new sections among existing sections
- Whether to add a sample spec file demonstrating new sections

## Deferred Ideas

None — discussion stayed within phase scope

## Additional Context

User shared a transcript from a live project where the QA agent failed to catch visual spacing issues (button flush against container bottom edge). This gap is what Phases 9-11 methodology will address; Phase 8's spec format infrastructure enables it to be spec-driven.
