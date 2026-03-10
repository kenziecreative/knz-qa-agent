---
phase: 02-agent-architecture
plan: 03
subsystem: hooks
tags: [claude-hooks, stop-hook, session-start, qa-context, git-diff, proactive-qa]

# Dependency graph
requires:
  - phase: 02-agent-architecture
    plan: 01
    provides: .qa/memory/ conventions, .qa/reports/ directory structure
provides:
  - hooks/hooks.json declaring Stop and SessionStart hooks
  - Stop hook that detects relevant code changes and prompts for QA
  - SessionStart hook that loads last QA report and known issues
  - Soft GSD awareness for users without orchestration tooling
affects:
  - 02-04-skills (hooks reference skill files that skills may extend)
  - Users running any project with Claude Code

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "hooks.json plugin hook declaration — Stop and SessionStart types"
    - "Git diff heuristics for QA relevance detection"
    - "Silent exit pattern — hooks produce zero noise when not relevant"
    - "One-time suggestion pattern — gsd-suggested marker prevents repeat prompts"

key-files:
  created:
    - hooks/hooks.json
    - hooks/stop.md
    - hooks/session-start.md
  modified: []

key-decisions:
  - "Hooks work independently of GSD — no orchestration tooling required"
  - "Stop hook uses git diff heuristics: source/style/config changes trigger QA, docs/tests/planning-only do not"
  - "GSD suggestion is soft and one-time — tracked via .qa/memory/gsd-suggested marker file"
  - "SessionStart shows full report content on findings — never truncates (consistent with 02-02 decision)"
  - "Silent exit when no .qa/ directory — zero noise for projects not using this agent"

patterns-established:
  - "Hook skill pattern: markdown files with YAML frontmatter, imperative step-by-step instructions"
  - "Relevance filtering: classify files before deciding to prompt user"

# Metrics
duration: ~2min
completed: 2026-03-10
---

# Phase 2 Plan 03: Hook System Summary

**Stop hook detects relevant code changes via git diff and prompts "Run QA? [yes / skip]"; SessionStart hook loads full last QA report and known issues from .qa/reports/ and .qa/memory/ at session start**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-03-10T12:51:53Z
- **Completed:** 2026-03-10T12:53:11Z
- **Tasks:** 2
- **Files created:** 3

## Accomplishments

- Created `hooks/hooks.json` declaring Stop and SessionStart hooks for the Claude Code plugin
- Created `hooks/stop.md` with git diff heuristics to detect relevant code changes (source, styles, config) vs. noise (docs, tests, planning)
- Stop hook prompts user directly: "Run QA? [yes / skip]" — not a passive nudge
- Soft GSD awareness in Stop hook — one-time suggestion for sessions with 5+ relevant changes and no orchestration tooling, tracked via `.qa/memory/gsd-suggested` marker
- Created `hooks/session-start.md` to load full last QA report from `.qa/reports/` at session start
- SessionStart reads known issues from `.qa/memory/` markdown files, skipping entries older than 30 days
- Both hooks exit silently when not relevant — zero noise design

## Task Commits

Each task was committed atomically:

1. **Task 1: Create hooks.json and Stop hook** - `13cabfa` (feat)
2. **Task 2: Create SessionStart hook** - `47e1ffd` (feat)

**Plan metadata:** (see final commit below)

## Files Created/Modified

- `hooks/hooks.json` — Declares Stop and SessionStart hooks pointing to skill files
- `hooks/stop.md` — Stop hook: git diff change detection, QA prompt, soft GSD awareness
- `hooks/session-start.md` — SessionStart hook: load .qa/reports/ and .qa/memory/ context

## Decisions Made

- **Hooks are GSD-independent:** Both hooks work for any Claude Code user regardless of whether they use GSD or any orchestration tooling. GSD is mentioned softly (once) as an option, never as a requirement.
- **Stop hook change classification:** Source code, styles, API routes, and runtime config trigger QA. Docs-only, tests-only, planning-only, and lock files do not. Rationale: reduces false positives so the prompt is meaningful.
- **GSD suggestion = one-time and conditional:** Only surfaces when `.planning/` is absent AND 5+ relevant files changed AND `.qa/memory/gsd-suggested` doesn't exist. Three conditions prevent it from being annoying.
- **SessionStart shows full report:** Consistent with the 02-02 decision — full report is shown because users need full context. Clean passes get a one-liner, findings get the full content.
- **Silent exit by default:** If `.qa/` doesn't exist, hooks exit with zero output. Respects users who haven't opted into QA for a given project.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — hooks activate automatically when `hooks/hooks.json` is recognized by Claude Code.

## Next Phase Readiness

- ARCH-03 (hooks.json) complete: declares Stop and SessionStart hooks
- ARCH-04 (Stop hook) complete: detects code changes, prompts directly, includes soft GSD awareness
- ARCH-05 (SessionStart hook) complete: loads full last QA report and known issues
- Memory paths (.qa/reports/, .qa/memory/) consistent with 02-01 conventions
- No blockers or concerns for remaining phase 2 plans

---
*Phase: 02-agent-architecture*
*Completed: 2026-03-10*
