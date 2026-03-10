---
phase: 03-deep-web-testing
plan: "03"
name: Form Intelligence
subsystem: qa-agent
tags: [forms, validation, submission-states, autofill, multi-step]

dependency-graph:
  requires: ["03-02"]
  provides: ["Form Intelligence methodology", "form testing instructions", "form edge case template"]
  affects: ["03-04"]

tech-stack:
  added: []
  patterns:
    - "Boundary testing for all input field types"
    - "Confidence tiers for form findings"
    - "Phased form testing: FORM-01 through FORM-05"

key-files:
  created: []
  modified:
    - agents/qa-tester.md
    - skills/qa-run.md
    - skills/qa-gen.md

decisions:
  - id: form-01-scope
    choice: "Injection string testing is framed as UI robustness, not security"
    rationale: "Clarifies intent — agent verifies graceful handling, not vulnerability detection"
  - id: form-confidence
    choice: "Double-submit is High confidence only when it creates duplicate data"
    rationale: "Harmless double-submit (idempotent endpoint) is not impactful enough for High"

metrics:
  duration: "~1 minute"
  completed: "2026-03-10"
---

# Phase 03 Plan 03: Form Intelligence Summary

**One-liner:** Exhaustive form testing via FORM-01 through FORM-05 — validation boundaries, error message quality, submission states, autofill, and multi-step navigation.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Add Form Intelligence methodology to qa-tester agent | af1de7e | agents/qa-tester.md |
| 2 | Update qa-run and qa-gen for form testing | cfcdb85 | skills/qa-run.md, skills/qa-gen.md |

## What Was Built

### agents/qa-tester.md — Form Intelligence section

Added a new "## Form Intelligence" section placed after the Multi-Persona Testing section. Covers five numbered areas:

- **FORM-01 Validation Boundary Testing:** Empty submission, invalid formats, boundary values (min/max/over-max/Unicode/emoji), injection strings (framed as UI robustness)
- **FORM-02 Error Message Quality:** Visibility, field association, `aria-invalid`, message helpfulness, error clearing on correction
- **FORM-03 Submission States:** Loading indicator, success feedback, server-error handling with data retention, double-submit prevention
- **FORM-04 Autofill & Field Types:** `type` attribute correctness, `autocomplete` attributes, autofill compatibility, password masking
- **FORM-05 Multi-Step Forms:** Forward/backward navigation, data persistence across steps, skip-ahead behavior, per-step validation, full-data final submission

Includes a confidence table for form findings.

### skills/qa-run.md — Form testing bullet in agent prompt

Added a bullet in the `## Instructions` section after the viewport/persona bullets, directing the agent to apply Form Intelligence methodology with specific sub-bullets for each FORM category.

### skills/qa-gen.md — Form Testing subsection in Edge Cases template

Added a `### Form Testing (if forms are present)` subsection to the `## Edge Cases to Verify` section of the generated spec template.

## Deviations from Plan

None — plan executed exactly as written.

## Next Phase Readiness

Plan 03-04 (the final plan in Phase 03) can proceed. No blockers.
