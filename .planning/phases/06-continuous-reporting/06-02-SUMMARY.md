---
phase: 06-continuous-reporting
plan: "02"
subsystem: testing
tags: [qa-agent, reporting, trend-analysis, github-issues, gh-cli, cross-session, history]

# Dependency graph
requires:
  - phase: 06-01
    provides: qa-monitor skill that writes .qa/reports/ and .qa/HISTORY.md — the files qa-report now reads
provides:
  - /qa:report skill with file-based reading from .qa/reports/ as primary data source
  - Trend analysis from .qa/HISTORY.md with stability score, recurring failures, regression timeline
  - Recurring failure detection with 3+ run threshold and RECURRING escalation flag
  - GitHub issue creation via gh CLI with duplicate detection for High confidence findings
  - --since flag for historical window control (date or duration)
  - --create-issues flag for optional issue creation
  - Cross-session reporting: works without prior conversation context
affects: [users-of-qa-report, ci-cd-integrations, future-qa-phases]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "File-first data pattern: disk files are always the primary source, conversation context is fallback only"
    - "Two-question report structure: latest run (what happened) then trends (is it a pattern)"
    - "Graceful degradation: features skip with informative notes when prerequisites are absent"
    - "Duplicate-aware issue creation: search before create prevents noise"

key-files:
  created: []
  modified:
    - skills/qa-report.md

key-decisions:
  - "File-based reading is PRIMARY — conversation context is exceptional fallback only for empty .qa/reports/ with same-session test run"
  - "Report leads with latest run (immediate answer) before trends (pattern context)"
  - "3+ failure threshold for RECURRING flag — 2 failures listed but not escalated"
  - "Medium confidence findings listed but NOT auto-created as issues — requires manual review"
  - "Graceful gh auth failure: skips issue creation with instructions, does not fail report generation"
  - "JSON format includes trends as structured objects for CI/CD pipeline consumption"

patterns-established:
  - "Cross-session pattern: all QA skills should read from .qa/ files, not session state"
  - "Trend window default: last 5 runs or 7 days, whichever is less — scoped but meaningful"

# Metrics
duration: 2min
completed: 2026-03-11
---

# Phase 6 Plan 02: Continuous Reporting - Report Skill Summary

**Cross-session /qa:report skill reading from .qa/reports/ with trend analysis, recurring failure detection (3+ run threshold), and GitHub issue creation via gh CLI with duplicate checking.**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-03-11T20:57:11Z
- **Completed:** 2026-03-11T20:58:50Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Rewrote /qa:report to read from `.qa/reports/` directory as primary data source — works in any new session without conversation context
- Added trend analysis from `.qa/HISTORY.md`: stability score, recurring failures, regression timeline, improvement trends
- Added recurring failure detection with 3+ run threshold for RECURRING escalation and 2+ run listing
- Added `--create-issues` flag: creates GitHub issues from High confidence findings via `gh issue create` with duplicate search before creation
- Added `--since` flag supporting both date (e.g., `2026-03-01`) and duration (e.g., `7d`, `14d`) formats
- JSON output format extended to include structured trend data for CI/CD consumption
- Graceful degradation throughout: missing history skips trends with informative note; gh auth failure skips issues without breaking report

## Task Commits

Each task was committed atomically:

1. **Task 1: Upgrade qa-report with file-based reading and trend analysis** - `15f752c` (feat)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `skills/qa-report.md` — Rewritten skill with 472 lines: file-based reading, trend analysis, recurring failure detection, GitHub issue creation, --since and --create-issues flags

## Decisions Made

- File-based reading is PRIMARY — conversation context is an exceptional fallback only when .qa/reports/ is empty AND a test ran in the current session. This distinction matters for cross-session reliability.
- Report leads with "Latest Run" section before "Trend Analysis" — users want immediate answers first, then longitudinal context.
- 3+ failure runs triggers RECURRING flag with elevated display; 2 failures are listed but not escalated. Threshold prevents noise from intermittent failures.
- Medium confidence findings are listed for --create-issues but NOT auto-created. Only High confidence findings warrant automatic issue creation; medium require human review.
- gh auth failures skip issue creation gracefully — the report itself is still generated. Auth errors shouldn't prevent seeing test results.
- JSON format includes trends as structured objects, making the output useful for CI/CD pipelines and custom reporting tools.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required beyond what users already have (gh CLI if using --create-issues).

## Next Phase Readiness

Phase 6 is now complete. Both plans delivered:
- 06-01: /qa:monitor skill for continuous automated testing with regression detection
- 06-02: /qa:report skill for cross-session reporting with trend analysis and issue creation

The full continuous reporting loop is in place:
- Monitor writes to .qa/reports/ and .qa/HISTORY.md
- Report reads from those files in any new session
- Recurring failures surface across sessions
- Critical findings flow to GitHub issues

No blockers for future phases. The agent now has persistent intelligence across sessions.

---
*Phase: 06-continuous-reporting*
*Completed: 2026-03-11*
