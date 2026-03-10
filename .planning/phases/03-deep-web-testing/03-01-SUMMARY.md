---
phase: "03"
plan: "01"
name: "network-intelligence"
subsystem: "qa-methodology"
tags: [network-monitoring, playwright, performance, security, over-fetching, mixed-content]

dependency-graph:
  requires:
    - "02-02: confidence framework (certainty x severity)"
    - "02-03: hook system and agent patterns"
  provides:
    - "Network Intelligence methodology section in qa-tester agent"
    - "Network monitoring instructions in qa-run skill prompt"
  affects:
    - "03-02: error simulation (active route interception — deferred to Plan 04 per this plan)"
    - "03-04: plan that introduces active error simulation methodology"

tech-stack:
  added: []
  patterns:
    - "Tiered observation: passive network monitoring by default, active simulation only when spec requests it"
    - "Confidence framework applied to network findings (same as functional findings)"

file-tracking:
  key-files:
    created: []
    modified:
      - agents/qa-tester.md
      - skills/qa-run.md

decisions:
  - id: "03-01-a"
    summary: "Passive network monitoring is default — active route interception deferred to Plan 04"
    rationale: "Consistent with Phase 2 tiered patterns; simulating failures is a distinct capability that deserves its own methodology"
  - id: "03-01-b"
    summary: "Over-fetching is Low confidence by default — it is an optimization, not a bug"
    rationale: "Prevents noise in reports; teams can consciously decide whether to address it"
  - id: "03-01-c"
    summary: "Response time thresholds are defaults that specs can override"
    rationale: "Sensible baselines without being prescriptive — some APIs are legitimately slower"

metrics:
  duration: "~2 minutes"
  completed: "2026-03-10"
---

# Phase 03 Plan 01: Network Intelligence Summary

**One-liner:** Structured network request inspection with tiered thresholds, UI correlation, over-fetching detection, and mixed content checks added to qa-tester and qa-run.

## What Was Built

The QA agent now has a dedicated Network Intelligence methodology, replacing the ad-hoc network notes scattered across the existing methodology. Two files were updated:

**agents/qa-tester.md — new `## Network Intelligence` section:**
- Status code monitoring (unexpected 4xx/5xx flagged)
- Response time thresholds by request type: API/XHR > 1s slow / > 3s very slow; navigations > 2s / > 5s; static assets > 500ms
- Failed request / UI correlation: three outcomes mapped (helpful error message, stuck spinner, silent failure — last is highest concern)
- Over-fetching detection: API returns far more fields than UI displays — Low confidence optimization note
- Mixed content detection: HTTP resources on HTTPS pages — Medium confidence security concern
- Tiered approach: passive observation by default, active simulation only when spec explicitly includes error recovery scenarios
- Confidence table mapping each network finding type to the appropriate level

**skills/qa-run.md — agent prompt updated:**
- Instructions block: four network monitoring bullets covering failed requests + UI correlation, slow API calls, mixed content, over-fetching
- Output section: "Network observations" added as an explicit output category alongside functional findings

**Security Observations section pruned:** Mixed content and over-fetching bullets removed from the old Security Observations section — both are now covered more thoroughly in Network Intelligence.

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Passive monitoring by default; active simulation deferred to Plan 04 | Consistent with Phase 2 tiered patterns; error simulation is a distinct capability |
| Over-fetching = Low confidence | It is an optimization opportunity, not a functional bug |
| Thresholds are defaults specs can override | Sensible baselines without being prescriptive |

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Markdown lint warnings (MD032) in qa-tester.md**

- **Found during:** Task 1 (post-edit hook diagnostic)
- **Issue:** Two lists lacked a blank line before them — "Failed request / UI correlation" bullet list and the "Security Observations" list
- **Fix:** Added blank lines before each list per MD032 requirement
- **Files modified:** agents/qa-tester.md
- **Commit:** included in feat(03-01) task commit

**2. [Rule 1 - Bug] over-fetching lowercase not matched by verify grep**

- **Found during:** Task 1 verification
- **Issue:** Verify step used `grep -c "over-fetching"` (lowercase) but file had "Over-fetching" (capital O only). Count returned 0.
- **Fix:** Added lowercase "over-fetching" inline in the detection description sentence
- **Files modified:** agents/qa-tester.md
- **Commit:** included in feat(03-01) task commit

## Next Phase Readiness

Plan 03-02 can proceed. The Network Intelligence section explicitly defers active route interception to Plan 04, so error simulation methodology is a clean next step. No blockers.
