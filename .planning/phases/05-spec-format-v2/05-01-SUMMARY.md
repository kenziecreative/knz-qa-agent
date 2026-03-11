---
phase: 05-spec-format-v2
plan: 01
subsystem: testing
tags: [qa, spec-format, dependencies, data-driven, tags, environments, accessibility, agent]

# Dependency graph
requires:
  - phase: 04-structured-accessibility
    provides: Tier 2 accessibility methodology in qa-tester agent
provides:
  - v2 spec format documentation with depends_on, Data Sets, tags, Environments, Accessibility Focus
  - Sample spec demonstrating all v2 features in a realistic login flow
  - Agent prompt with v2 spec interpretation instructions for all five features
affects:
  - 05-spec-format-v2-plan-02
  - 05-spec-format-v2-plan-03
  - skills/qa-run.md
  - skills/qa-gen.md

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "depends_on references scenarios by exact heading name — single-level chaining"
    - "Data Sets use {variable} placeholder substitution in steps and expected results"
    - "Standard tags (smoke, critical, regression, a11y) have defined execution behaviors"
    - "Environment profiles fall back to Base URL when absent (v1 backward compatibility)"
    - "Accessibility Focus is a depth toggle: absent=Tier 1, present=Tier 2 (partial or full)"

key-files:
  created:
    - examples/SPEC-FORMAT.md
  modified:
    - examples/sample-login-spec.md
    - agents/qa-tester.md
    - .gitignore

key-decisions:
  - "Single-level dependencies resolved top-to-bottom in spec order"
  - "Data-driven variants all run even if one fails — partial failure reporting"
  - "Tags inform agent behavior (thoroughness, speed, emphasis) but agent does not filter"
  - "Environment secrets use placeholder values or env var references ($STAGING_PASSWORD)"
  - "Accessibility Focus controls Tier 2 depth with four named areas"
  - "All v2 fields are optional — full backward compatibility with v1 specs"

patterns-established:
  - "Spec-level sections (Environments, Accessibility Focus) go after Viewports, before Test Scenarios"
  - "Scenario-level fields (tags, depends_on) go immediately after the scenario heading"
  - "Data Sets subsection lives within a scenario block with ## heading"

# Metrics
duration: 3min
completed: 2026-03-11
---

# Phase 5 Plan 01: v2 Spec Format Definition and Agent Interpretation Summary

**v2 spec format with depends_on, data-driven scenarios, tags, environment profiles, and accessibility depth — documented in SPEC-FORMAT.md, demonstrated in sample spec, and interpreted by qa-tester agent**

## Performance

- **Duration:** ~3 min
- **Started:** 2026-03-11T08:50:09Z
- **Completed:** 2026-03-11T09:02:12Z
- **Tasks:** 2
- **Files created:** 1 (examples/SPEC-FORMAT.md)
- **Files modified:** 3 (sample-login-spec.md, qa-tester.md, .gitignore)

## Accomplishments

- Created comprehensive SPEC-FORMAT.md documenting all five v2 additions with syntax, examples, behavior descriptions, and a Backward Compatibility section
- Updated the Basic Structure example to show where v2 sections fit in a spec
- Added Section Reference entries for Environments, Accessibility Focus, Tags, Scenario Dependencies, and Data-Driven Scenarios
- Updated sample-login-spec.md with all five v2 features: tags on every scenario, depends_on on Session Persistence and Logout Flow, data-driven Failed Login with three credential variants, Environments section (local/staging), and Accessibility Focus (form-accessibility)
- Added "Spec Format v2 Features" section to qa-tester agent prompt with complete interpretation and execution instructions for all five features
- Added .gitignore exception for examples/SPEC-FORMAT.md (was blocked by SPEC-*.md pattern)

## Task Commits

Each task was committed atomically:

1. **Task 1: Define v2 spec format sections in SPEC-FORMAT.md and update sample spec** - `7a8cca2` (feat)
2. **Task 2: Update qa-tester agent to understand and act on v2 spec sections** - `0fa3002` (feat)

## Files Created/Modified

- `examples/SPEC-FORMAT.md` - Complete v2 spec format documentation: Basic Structure with v2 sections, Section Reference for all five features, Tips updated with tag/dependency guidance, Backward Compatibility statement, full syntax examples for each feature
- `examples/sample-login-spec.md` - Added Environments (local/staging), Accessibility Focus (form-accessibility), tags on all 6 scenarios, depends_on on scenarios 5 and 6, data-driven variant on Failed Login with wrong-credentials/expired-account/locked-account data sets
- `agents/qa-tester.md` - Added "Spec Format v2 Features" section (between Session Structure and Memory & Persistence) with Scenario Dependencies skip behavior, Data-Driven per-variant execution, Tags informational context, Environment Profiles override chain, Accessibility Focus Tier 1/Tier 2 depth toggle
- `.gitignore` - Added `!examples/SPEC-FORMAT.md` exception to the `SPEC-*.md` ignore pattern

## Decisions Made

- Single-level dependencies with top-to-bottom resolution. No transitive resolution needed — the spec author controls ordering. Dependencies reference scenarios by exact heading name.
- Data-driven variants continue on failure — all variants run even if one fails, matching common test runner behavior and maximizing information per run.
- Tags are informational to the agent: `critical` means extra thoroughness, `smoke` means be fast, `a11y` means emphasize accessibility, `regression` means pay attention to the specific previously-broken behavior. Filtering happens in qa-run skill, not the agent.
- Environment credentials use `$ENV_VAR` placeholder syntax. Real secrets come from environment variables or `.qa/env.local` (gitignored).
- Accessibility Focus is a depth toggle with four named areas that map to Structured Accessibility subsections. Baseline Tier 1 always runs regardless.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] .gitignore blocking SPEC-FORMAT.md**

- **Found during:** Task 1 commit
- **Issue:** The existing `SPEC-*.md` gitignore pattern prevented `examples/SPEC-FORMAT.md` from being tracked
- **Fix:** Added `!examples/SPEC-FORMAT.md` negation rule to .gitignore
- **Files modified:** .gitignore
- **Commit:** 7a8cca2

## Issues Encountered

None beyond the gitignore blocking issue (resolved as deviation above).

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- SPEC-FORMAT.md and agent prompt are the foundation that Plan 02 (skill updates) and Plan 03 (README/documentation) build on
- Plan 02 (already completed) references the v2 sections defined here for --tag/--env flag behavior
- Plan 03 should document the v2 format in the README for end users

---
*Phase: 05-spec-format-v2*
*Completed: 2026-03-11*
