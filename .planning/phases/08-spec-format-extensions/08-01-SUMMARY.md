---
phase: 08-spec-format-extensions
plan: "01"
subsystem: spec-format
tags: [spec-format, visual-testing, browser-engines, design-reference, documentation]
dependency_graph:
  requires: []
  provides:
    - Visual Focus spec section vocabulary and triggers
    - Design Reference spec section vocabulary and agent behavior
    - Browsers spec section vocabulary and skill orchestration
  affects:
    - examples/SPEC-FORMAT.md
    - agents/qa-tester.md
    - skills/run/SKILL.md
    - examples/visual-testing-spec.md
tech_stack:
  added: []
  patterns:
    - Tier toggle pattern (absent/present-empty/present-with-items) mirrored from Accessibility Focus
    - Viewport-keyed subsection pattern for Design Reference (mirrored from Environments)
    - Conditional forwarding block pattern in agent invocation template
key_files:
  created:
    - examples/visual-testing-spec.md
  modified:
    - examples/SPEC-FORMAT.md
    - agents/qa-tester.md
    - skills/run/SKILL.md
decisions:
  - Visual Focus mirrors Accessibility Focus pattern exactly (depth toggle, 4 phase-aligned areas)
  - Design Reference uses viewport-keyed subsections with state-path bullet items
  - Browsers uses simple bullet list matching Viewports pattern
  - Phase 8 installs vocabulary only — Tier 2 methodology is placeholder for Phases 9-12
  - Agent labels all findings with active engine name ([webkit] Scenario X PASS)
metrics:
  duration: "~3 minutes"
  completed: "2026-04-20"
  tasks_completed: 3
  files_modified: 3
  files_created: 1
---

# Phase 8 Plan 1: Spec Format Extensions Summary

**One-liner:** Added Visual Focus, Design Reference, and Browsers spec sections as vocabulary and triggers for v0.3 visual/UX methodology in Phases 9-12.

## What Was Built

Three new optional spec format sections were added across all relevant files:

### `## Visual Focus`
Controls visual verification depth using the same tier-toggle pattern as `## Accessibility Focus`. Four phase-aligned areas: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive`. When absent, Tier 1 visual observations only. When present, Tier 2 runs (with placeholder wording pointing to Phases 9-12 for actual methodology).

### `## Design Reference`
Provides mockup image paths per viewport using keyed subsections (`### desktop`, `### mobile`) with state-path bullet items. Agent skips visual diff for absent viewport subsections and notes "no design reference for [viewport]" in the report.

### `## Browsers`
Lists browser engines to test. Skill runs one agent invocation per engine. Agent labels all findings with the active engine name (`[webkit] Login form — PASS`). Defaults to Chromium when absent.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Add three Section Reference entries to SPEC-FORMAT.md | 8618fe2 | examples/SPEC-FORMAT.md |
| 2 | Add Visual Focus Trigger, Design Reference, Browsers to qa-tester.md | 87d56d5 | agents/qa-tester.md |
| 3 | Add forwarding blocks to SKILL.md and create sample spec | 9a2049f | skills/run/SKILL.md, examples/visual-testing-spec.md |

## Decisions Made

1. **Tier toggle pattern reused** — Visual Focus mirrors Accessibility Focus exactly: absent = Tier 1 only, present-empty = full Tier 2, present-with-items = selective Tier 2. Consistency over novelty.
2. **Phase-aligned area names** — `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive` map 1:1 to Phases 9-12 requirement groups, making the spec vocabulary and implementation phases directly traceable.
3. **Methodology is placeholder only** — Tier 2 visual methodology not implemented. Agent instructed to note "full methodology pending Phase [N]" if Visual Focus is present before those phases ship. No risk of partial/incorrect behavior.
4. **No combined example block in SPEC-FORMAT.md** — Each of the three new sections has its own independent example. Combined examples create maintenance drift risk (D-13).
5. **Agent receives one engine per invocation** — Skill loops and spawns one agent per engine; agent is stateless per run. Agent's only job is to label output with the engine name passed in the test assignment header.

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

- `agents/qa-tester.md` Visual Focus Trigger contains placeholder text: "Visual Focus requested for [areas] — full methodology pending Phase [N]." This is intentional — the actual Tier 2 visual methodology ships in Phases 9-12. The stub is documented and expected.

## Threat Flags

None — this phase makes zero changes to runtime execution paths. All deliverables are static Markdown documentation and AI instruction edits. Design Reference paths are relative project paths, not secrets.

## Self-Check: PASSED

All created/modified files confirmed present. All task commits confirmed in git log.

| Item | Status |
|------|--------|
| examples/SPEC-FORMAT.md | FOUND |
| agents/qa-tester.md | FOUND |
| skills/run/SKILL.md | FOUND |
| examples/visual-testing-spec.md | FOUND |
| 08-01-SUMMARY.md | FOUND |
| commit 8618fe2 (Task 1) | FOUND |
| commit 87d56d5 (Task 2) | FOUND |
| commit 9a2049f (Task 3) | FOUND |
