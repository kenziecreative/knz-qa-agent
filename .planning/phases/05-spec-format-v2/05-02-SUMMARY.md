---
phase: 05-spec-format-v2
plan: 02
subsystem: testing
tags: [qa, skill, tag-filtering, environment-profiles, accessibility, spec-format-v2]

# Dependency graph
requires:
  - phase: 05-spec-format-v2-plan-01
    provides: v2 spec format with tags, depends_on, Environments, and Accessibility Focus sections
provides:
  - qa-run skill with --tag flag for scenario filtering (OR logic, comma-separated)
  - qa-run skill with --env flag for environment profile selection with backward compatibility
  - qa-run dependency auto-inclusion after tag filtering
  - qa-run v2 context passed to agent (Active Environment, Tag Filter, Execution Notes blocks)
  - qa-gen skill with --a11y-depth flag (baseline/deep)
  - qa-gen generates tags on every scenario, depends_on hints, Environments template, Accessibility Focus section
affects:
  - 05-spec-format-v2-plan-03
  - examples/SPEC-FORMAT.md
  - README

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Tag filtering uses OR logic for multiple tags — scenario matches if ANY requested tag present"
    - "Environment resolution falls back gracefully: --env profile > first profile > Base URL (v1 compat)"
    - "Dependency auto-inclusion: filtered-out dependencies are automatically included and labeled"
    - "a11y-depth flag controls Tier 1 vs Tier 2 accessibility depth in generated specs"

key-files:
  created: []
  modified:
    - skills/qa-run.md
    - skills/qa-gen.md

key-decisions:
  - "Tag OR logic: comma-separated tags match if ANY listed tag present on scenario"
  - "Environment backward compatibility: specs without Environments section use Base URL as before"
  - "Dependency auto-inclusion after tag filtering: silently included, agent informed of additions"
  - "a11y-depth baseline is the default, deep is opt-in via explicit flag"
  - "Tag Assignment Guide added to qa-gen as standalone section for clarity"

patterns-established:
  - "Execution Notes block in agent prompt signals dependency/data-driven behavior to qa-tester"
  - "Active Environment block overrides spec Base URL when --env flag is active"

# Metrics
duration: 3min
completed: 2026-03-11
---

# Phase 5 Plan 02: Skill Updates for v2 Spec Features Summary

**qa-run and qa-gen wired to v2 spec format: --tag/--env flags for filtering and profiles, --a11y-depth for accessibility depth, with full backward compatibility**

## Performance

- **Duration:** ~3 min
- **Started:** 2026-03-11T08:50:08Z
- **Completed:** 2026-03-11T08:53:10Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments

- qa-run skill updated with Modes 5/6/7, --tag and --env argument parsing, environment resolution (with v1 fallback), tag filtering with OR logic, dependency auto-inclusion, and structured v2 context blocks passed to qa-tester agent
- qa-gen skill updated with --a11y-depth flag, v2 steps added to all three generation modes, Environments section template, scenario template with tags and depends_on fields, Accessibility Focus section (conditional on --a11y-depth deep), and Tag Assignment Guide
- All existing content preserved — both skills maintain full backward compatibility with v1 usage

## Task Commits

Each task was committed atomically:

1. **Task 1: Add --tag and --env flags to qa-run skill with v2 execution flow** - `9321685` (feat)
2. **Task 2: Add --a11y-depth flag and v2 section generation to qa-gen skill** - `752573d` (feat)

**Plan metadata:** (pending)

## Files Created/Modified

- `skills/qa-run.md` - Added Modes 5/6/7, --tag/--env argument parsing, environment resolution with backward compat, tag filtering with OR logic, dependency auto-inclusion, and v2 context blocks in agent prompt template
- `skills/qa-gen.md` - Added --a11y-depth flag, v2 generation steps across all modes, Environments and Accessibility Focus sections to output template, scenario tags/depends_on, and Tag Assignment Guide

## Decisions Made

- Tag OR logic: multiple tags via `--tag smoke,critical` match scenarios containing ANY listed tag. Consistent with most test runner conventions and easiest to reason about.
- Environment backward compatibility: if no `--env` flag and no `## Environments` section, skill falls back to `## Base URL` — v1 specs work unchanged.
- Dependency auto-inclusion: when tag filtering removes a scenario that another filtered scenario depends on, the dependency is silently re-added and the agent is informed. This prevents silent test failures from missing setup.
- `--a11y-depth baseline` is the default so existing `/qa:gen` calls are unchanged; `deep` is explicit opt-in.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

Minor markdown linting warnings (MD029 ordered list numbering) appeared when the 2a/2b/2c substeps were initially inserted as unordered list items mixed into an ordered list. Fixed by renumbering all steps sequentially (1-9) in the execution flow section.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Both skills updated and committed. Ready for Plan 03 (likely README documentation for v2 spec features).
- The v2 agent prompt blocks (Active Environment, Tag Filter, Execution Notes) in qa-run.md link to behavior the qa-tester agent will need to understand — Plan 03 may want to update agents/qa-tester.md to acknowledge these new context blocks.

---
*Phase: 05-spec-format-v2*
*Completed: 2026-03-11*
