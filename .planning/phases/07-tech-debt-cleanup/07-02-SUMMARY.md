---
phase: 07-tech-debt-cleanup
plan: 02
subsystem: hooks, plugin, agent
tags: [hooks, plugin.json, stop-hook, qa-run, claude-code]

# Dependency graph
requires:
  - phase: 02-agent-architecture
    provides: hooks system and plugin architecture established
  - phase: 07-tech-debt-cleanup
    provides: Plan 07-01 context for audit-driven fixes
provides:
  - Stop hook with correct spec path (.qa/*.md) and correct skill syntax (/qa:run)
  - Complete plugin.json manifest with hooks field referencing hooks/hooks.json
  - Clean agent without orphaned --background documentation
affects: []

# Tech tracking
tech-stack:
  added: []
  patterns: []

key-files:
  created: []
  modified:
    - hooks/stop.md
    - .claude-plugin/plugin.json
    - agents/qa-tester.md

key-decisions:
  - "Remove --background section from agent rather than adding flag to skills (scope: dead docs removed, not new feature added)"
  - "Stop hook checks .qa/*.md (glob pattern) not .qa/specs/ (wrong subdirectory) for spec detection"
  - "/qa:run uses colon syntax matching Claude Code skill conventions"

patterns-established: []

# Metrics
duration: 2min
completed: 2026-03-11
---

# Phase 7 Plan 02: Tech Debt Cleanup (Bugs + Dead Code) Summary

**Fixed two Stop hook bugs (wrong spec path, wrong skill syntax), added hooks field to plugin.json manifest, and removed unreachable --background documentation from agent**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-03-11T21:20:36Z
- **Completed:** 2026-03-11T21:22:48Z
- **Tasks:** 2/2
- **Files modified:** 3

## Accomplishments

- Stop hook now checks `.qa/*.md` instead of the non-existent `.qa/specs/` directory — spec detection actually works
- Stop hook now suggests `/qa:run` (colon syntax) instead of `/qa-run` (hyphen) — correct Claude Code skill invocation
- `plugin.json` now declares `"hooks": "hooks/hooks.json"` — complete plugin manifest
- `agents/qa-tester.md` no longer documents `--background` flag that no skill exposes — no dead code confusion

## Task Commits

Each task was committed atomically:

1. **Task 1: Fix Stop hook spec path and skill syntax** - `7863836` (fix)
2. **Task 2: Add hooks to plugin.json and remove --background from agent** - `6ba1bc7` (fix)

## Files Created/Modified

- `hooks/stop.md` - Fixed `ls .qa/specs/` → `ls .qa/*.md` and `/qa-run` → `/qa:run`
- `.claude-plugin/plugin.json` - Added `"hooks": "hooks/hooks.json"` field
- `agents/qa-tester.md` - Removed Background Execution section (13 lines of dead docs)

## Decisions Made

- **Remove vs. add:** Removed `--background` docs from agent rather than adding the flag to a skill. Background execution would be a feature addition (scope creep for a tech debt plan). Dead documentation removed cleanly.
- **Glob pattern for spec check:** Used `ls .qa/*.md` (glob) rather than just `ls .qa/` to specifically verify markdown spec files exist, not just that the `.qa/` directory itself exists.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Critical] Fixed MD032 lint warning (blank line before list)**

- **Found during:** Task 1 (Stop hook edits triggered IDE diagnostics)
- **Issue:** After editing line 79 in stop.md, the list at lines 79-80 lacked a blank line before it (MD032 violation)
- **Fix:** Added blank line between "Wait for the user's response:" and the bullet list
- **Files modified:** hooks/stop.md
- **Verification:** IDE diagnostics cleared
- **Committed in:** 7863836 (Task 1 commit)

---

**Total deviations:** 1 auto-fixed (1 style/lint)
**Impact on plan:** Minor formatting fix, no scope creep.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

Phase 7 (Tech Debt Cleanup) is now complete. Both plans executed:
- 07-01: Cleaned up duplicate qa-check hook, fixed hooks.json format
- 07-02: Fixed Stop hook bugs, completed plugin.json, removed dead agent docs

All 7 phases complete. The GSD integration flow is now fully functional:
code changes → Stop hook → correct `.qa/*.md` spec detection → correct `/qa:run` suggestion.

---
*Phase: 07-tech-debt-cleanup*
*Completed: 2026-03-11*
