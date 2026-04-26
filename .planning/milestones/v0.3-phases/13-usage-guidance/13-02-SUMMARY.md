---
phase: 13-usage-guidance
plan: "02"
status: complete
started: 2026-04-20T20:30:00Z
completed: 2026-04-20T20:45:00Z
---

# Plan 13-02 Summary

## Objective

Create the /qa:guide skill and update README.md + CLAUDE.md for discoverability.

## What Was Built

### skills/guide/SKILL.md (~150 lines)
A file-read-only skill that inspects project QA state from disk and returns context-aware guidance on which QA command to run next. Reads `.qa/` state files and `docs/ORCHESTRATION.md` to produce 2-4 bullet narrative recommendations. No agent spawning — purely reads and recommends.

Five state checks: `.qa/` existence, spec count, report count, HISTORY.md line count, and `.monitor-alert` presence. Priority ordering determines primary recommendation.

### README.md Updates
- Added `/qa:guide - Usage Guidance` section with bash examples
- Added `guide/SKILL.md` and `docs/ORCHESTRATION.md` to project structure tree
- Added ORCHESTRATION.md pointer in Development Notes
- Added v0.3.0 version history entry
- Updated skill list in "What This Is" paragraph

### CLAUDE.md Updates
- Added `/qa:guide` to skill list in "What This Is" section
- Added `guide/SKILL.md` to skills listing in Project Structure
- Added `docs/ORCHESTRATION.md` to Project Structure
- Added `guide` to "Skills that don't spawn the agent" line

## Key Decisions

- Followed report/SKILL.md pattern exactly for structure
- No Agent tool in allowed-tools per D-04 (file-read-only skill)
- Output is narrative bullets, not checklist or table per D-06

## Self-Check: PASSED

## Key Files

key-files:
  created:
    - skills/guide/SKILL.md
  modified:
    - README.md
    - CLAUDE.md
