---
phase: 11-layout-content-integrity
plan: 01
subsystem: testing
tags: [qa-agent, playwright-cli, layout, spacing, alignment, dom, structural-consistency]

# Dependency graph
requires:
  - phase: 10-ux-state-verification
    provides: UX State Verification section structure and dual eval+visual pattern this mirrors
  - phase: 09-design-verification
    provides: Design Verification dual pattern, DESIGN-05 cross-page consistency pattern, DESIGN-02 overflow detection
provides:
  - Layout & Content Integrity section header with security and environment notes
  - LAYOUT-01 Spacing & Alignment Checks (sibling gap comparison, padding symmetry, grid/flex integrity, content clipping, screenshot supplement)
  - LAYOUT-02 Cross-Page Structural Consistency (structural fingerprint, link text sampling, screenshot supplement, single-page fallback)
affects: [11-02-PLAN, layout-integrity trigger in specs, qa-tester agent behavior]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Sibling gap comparison via getBoundingClientRect() with 20% outlier threshold"
    - "Padding symmetry check via getComputedStyle() with 4px imbalance threshold"
    - "Grid/flex integrity via computed display/gap/alignItems/justifyContent properties"
    - "All-element content clipping via scrollWidth > clientWidth (extends DESIGN-02 text-only check)"
    - "Structural fingerprint via ARIA landmark + semantic HTML element count/childCount"
    - "Link text normalization via .toLowerCase().trim().replace(whitespace) for cross-page comparison"

key-files:
  created: []
  modified:
    - agents/qa-tester.md

key-decisions:
  - "LAYOUT-01 operates WITHOUT a design reference — infers correctness from internal consistency (sibling median comparison, padding symmetry)"
  - "LAYOUT-02 complements DESIGN-05 (CSS style values) not duplicates it — LAYOUT-02 is structural/DOM consistency"
  - "Content clipping check extends DESIGN-02 from text elements only to ALL visible elements (scrollWidth > clientWidth)"
  - "Sibling gap outlier threshold set at 20% deviation from sibling median (OUTLIER_THRESHOLD = 0.2)"
  - "Padding asymmetry flagged at Medium confidence — asymmetric padding is frequently intentional design"
  - "Missing landmark region across pages is High confidence — structural absence is deterministic"
  - "Link count deviation > 1 is Medium confidence — some pages legitimately have different nav items"
  - "Stub removal for layout-integrity deferred to Plan 02 — LAYOUT-03/04 not yet implemented"

patterns-established:
  - "Pattern 1: Sibling gap comparison — query container children, compute gaps via getBoundingClientRect(), flag outliers >20% from median"
  - "Pattern 2: Padding symmetry — getComputedStyle() paddingTop/Bottom/Left/Right comparison with 4px threshold"
  - "Pattern 3: Grid/flex integrity — computed display/gap/alignItems/justifyContent on flex/grid containers"
  - "Pattern 4: Structural fingerprint — landmark role count + childCounts per page for cross-page comparison"
  - "Pattern 5: Link text sampling — extract, normalize, compare nav/header/footer link arrays across pages"

requirements-completed: [LAYOUT-01, LAYOUT-02]

# Metrics
duration: ~10min
completed: 2026-04-20
---

# Phase 11 Plan 01: Layout & Content Integrity (LAYOUT-01 + LAYOUT-02) Summary

**Layout & Content Integrity methodology section added to qa-tester.md with LAYOUT-01 (spacing/alignment/clipping via sibling gap comparison and all-element overflow detection) and LAYOUT-02 (cross-page structural consistency via landmark fingerprinting and link text sampling)**

## Performance

- **Duration:** ~10 min
- **Started:** 2026-04-20T10:36:00Z
- **Completed:** 2026-04-20T10:46:07Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Added `### Layout & Content Integrity` section header with `layout-integrity` trigger, security note (CSS custom property exclusion), and environment note (DOM mutation warning for LAYOUT-03)
- Added `#### Spacing & Alignment Checks (LAYOUT-01)` with 5 procedure steps: sibling gap comparison (20% outlier threshold), padding symmetry (4px imbalance threshold), grid/flex integrity, all-element content clipping, and screenshot supplement
- Added `#### Cross-Page Structural Consistency (LAYOUT-02)` with 4 procedure steps: structural landmark fingerprint, link text sampling with normalization, screenshot supplement, and single-page fallback
- Both subsections have Confidence summary lines; section correctly positioned between UX State Verification Findings table and Design Reference section

## Task Commits

Each task was committed atomically:

1. **Task 1: Layout & Content Integrity section header + LAYOUT-01** - `90018dd` (feat)
2. **Task 2: Cross-Page Structural Consistency (LAYOUT-02)** - `90018dd` (feat — same commit, both tasks in same file edit session)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `agents/qa-tester.md` - Added Layout & Content Integrity section (lines 1421-1611) with LAYOUT-01 and LAYOUT-02 subsections

## Decisions Made

- LAYOUT-01 infers correctness from internal consistency rather than comparing to a design reference (D-07) — distinct from DESIGN-01 which requires a reference image
- LAYOUT-02 complements DESIGN-05 rather than duplicating it — DESIGN-05 compares CSS computed style values, LAYOUT-02 compares structural DOM presence and nav link content
- All-element content clipping (LAYOUT-01 step 4) extends the DESIGN-02 text-only overflow pattern to images, flex children, and all visible elements
- Stub update (removing `layout-integrity` from the Phase 11-12 pending note at line 747) deferred to Plan 02 — the full `layout-integrity` methodology (LAYOUT-03 and LAYOUT-04) is not complete until Plan 02 ships

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- LAYOUT-01 and LAYOUT-02 are complete and ready for use by specs listing `layout-integrity` in their Visual Focus section
- Plan 02 (11-02) implements LAYOUT-03 (realistic-data overflow testing) and LAYOUT-04 (placeholder/draft content detection) to complete the `layout-integrity` methodology
- The stub at line 747 should be updated in Plan 02 after LAYOUT-03 and LAYOUT-04 are added

---
*Phase: 11-layout-content-integrity*
*Completed: 2026-04-20*
