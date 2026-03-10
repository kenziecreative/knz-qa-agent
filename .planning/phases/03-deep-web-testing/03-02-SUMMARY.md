---
phase: 03
plan: 02
subsystem: qa-agent
tags: [viewport, responsive, multi-persona, auth, playwright]

dependency-graph:
  requires: ["03-01"]
  provides: ["multi-viewport-orchestration", "multi-persona-orchestration"]
  affects: ["03-03", "03-04"]

tech-stack:
  added: []
  patterns:
    - "Persona-first orchestration: run all viewports per persona to minimize auth cycles"
    - "Touch target threshold: 44x44px minimum flagged via Playwright bounding boxes"
    - "Viewport set: mobile 375x812, tablet 768x1024, desktop 1280x800 (spec-overridable)"

key-files:
  created: []
  modified:
    - agents/qa-tester.md
    - skills/qa-run.md
    - skills/qa-gen.md

decisions:
  - id: "03-02-A"
    decision: "Default viewports are mobile/tablet/desktop; specs narrow or extend"
    rationale: "Covers the most common breakpoints without forcing every project to configure from scratch"
  - id: "03-02-B"
    decision: "Orchestration order: all viewports per persona A, then persona B"
    rationale: "Minimizes login/logout cycles — auth flow overhead is the expensive operation"
  - id: "03-02-C"
    decision: "Unauthorized access to restricted content = High confidence finding (security)"
    rationale: "Certainty × severity — permission bypass is definitively real and definitively impactful"

metrics:
  duration: "~1 minute"
  completed: "2026-03-10"
---

# Phase 3 Plan 02: Multi-Viewport and Multi-Persona Orchestration Summary

**One-liner:** Systematic viewport iteration (375/768/1280px) and persona switching with auth flow handling, embedded in agent methodology and spec templates.

## What Was Built

Added two new methodology sections to the QA tester agent and wired them through to the skill files that orchestrate and generate test specs.

### agents/qa-tester.md

Two sections added after Network Intelligence:

**Multi-Viewport Testing** — Establishes default viewport set (mobile 375x812, tablet 768x1024, desktop 1280x800), explains spec override mechanism, and defines what to check at each size: responsive breakpoints, 44x44px touch targets via Playwright bounding boxes, horizontal scrolling (document width vs viewport width), navigation pattern changes, and content reflow. Reporting guidance directs the agent to note the specific viewport where each issue appears.

**Multi-Persona Testing** — Defines persona iteration with auth flow handling (log out, clear state, log in as next persona), scope guidance (not every scenario needs every persona), what to verify per persona (role-based visibility, permission-based actions), and orchestration order (all viewports per persona A before switching to persona B).

### skills/qa-run.md

- Added steps 4 and 5 to the spec execution flow: viewport iteration with fallback defaults, and persona iteration with auth handling
- Added two bullets to the agent instructions template: viewport testing (responsive issues, touch targets, horizontal scroll) and persona testing (login/logout between switches, role-based content verification)

### skills/qa-gen.md

Added Viewports and Personas sections to the spec output template, between Base URL and Test Scenarios. Both sections include `[TODO: ...]` annotations so generated specs prompt users to customize credentials and auth methods before running.

## Decisions Made

| ID | Decision | Rationale |
|----|----------|-----------|
| 03-02-A | Default viewports: mobile/tablet/desktop; specs narrow or extend | Covers common breakpoints without forcing configuration from scratch |
| 03-02-B | Orchestration: all viewports per persona before switching | Minimizes auth cycles — login/logout is the expensive operation |
| 03-02-C | Unauthorized access to restricted content = High confidence | Permission bypass is definitively real and definitively impactful |

## Verification

All plan verification criteria passed:

- `grep -c "Multi-Viewport" agents/qa-tester.md` → 1
- `grep -c "Multi-Persona" agents/qa-tester.md` → 1
- `grep "44x44" agents/qa-tester.md` → match
- `grep "375" agents/qa-tester.md` → match
- `grep "viewport" skills/qa-run.md` → matches
- `grep "persona" skills/qa-run.md` → matches
- `grep "Viewports" skills/qa-gen.md` → match
- `grep "Personas" skills/qa-gen.md` → match

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] MD032 linting: lists not surrounded by blank lines**

- **Found during:** Task 1, triggered by IDE diagnostics after edit
- **Issue:** Bold subheadings followed immediately by list items violated MD032 (blanks-around-lists) at lines 187, 194, 208, 214
- **Fix:** Added blank line between each bold heading and its following list in the two new sections
- **Files modified:** agents/qa-tester.md
- **Commit:** included in task 1 commit (5d05d22)

## Next Phase Readiness

Plan 03-03 (Error State Testing) and 03-04 (Active Route Interception) can proceed. The viewport/persona framework is now part of the agent's core methodology and will be referenced by specs generated going forward.
