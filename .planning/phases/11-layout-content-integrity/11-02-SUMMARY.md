---
phase: 11-layout-content-integrity
plan: 02
subsystem: testing
tags: [qa-agent, playwright-cli, layout, overflow, placeholder-detection, dom-injection, content-integrity]

# Dependency graph
requires:
  - phase: 11-01
    provides: LAYOUT-01 and LAYOUT-02 subsections, Layout & Content Integrity section header — insertion point for LAYOUT-03
  - phase: 10-ux-state-verification
    provides: Reporting table pattern (mirrored by Reporting Layout & Content Integrity Findings)
provides:
  - LAYOUT-03 Realistic-Data Overflow Testing (tiered injection, overflow check, page reload, route escalation)
  - LAYOUT-04 Placeholder & Draft Content Detection (regex scan, allowlist, screenshot judgment)
  - Reporting Layout & Content Integrity Findings table (16 rows, 6 High + 10 Medium)
  - Visual Focus stub note updated — layout-integrity removed from pending, performance-responsive retained for Phase 12
affects: [layout-integrity trigger completeness, qa-tester methodology, placeholder detection capability]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "DOM textContent injection via playwright-cli run-code for overflow stress testing"
    - "Tiered test content vocabulary: German compounds, long Latin, large currency, long email"
    - "Mandatory page reload after LAYOUT-03 injection via playwright-cli navigate [same url]"
    - "Route interception escalation path for locale/API-driven overflow (spec-driven, uses playwright-cli route + route-clear)"
    - "TreeWalker-based visible text scan for placeholder regex detection"
    - "placeholder_allowlist spec field for suppressing known false positives"
    - "LLM screenshot judgment for vague CTAs, casing inconsistency, placeholder images"

key-files:
  created: []
  modified:
    - agents/qa-tester.md

key-decisions:
  - "LAYOUT-03 uses DOM textContent injection as default path — route interception is spec-driven escalation only"
  - "Tiered vocabulary covers four failure modes: basic length (German long phrase), word-break (Donaudampfschifffahrtsgesellschaft), number container ($1,234,567,890.99), form field (long email)"
  - "Page reload after LAYOUT-03 is mandatory — prevents false positives in LAYOUT-04 placeholder scan from injected content"
  - "LAYOUT-04 regex scan extended to alt/title/aria-label/meta — catches placeholders not visible on screen"
  - "placeholder_allowlist defers to spec author for known intentional values (e.g., a product literally named TODO)"
  - "LLM screenshot judgment for casing scoped to same-type elements — avoids false positives from intentional typographic hierarchy"
  - "Reporting table rows ordered: 6 High (deterministic eval results) then 10 Medium (screenshot/judgment-based)"
  - "Visual Focus stub note updated: layout-integrity removed from pending, performance-responsive retained for Phase 12"

# Metrics
duration: ~5min
completed: 2026-04-20
---

# Phase 11 Plan 02: Layout & Content Integrity (LAYOUT-03 + LAYOUT-04 + Reporting + Stub Update) Summary

**Completed the Layout & Content Integrity methodology with LAYOUT-03 (realistic-data overflow testing via tiered DOM injection), LAYOUT-04 (placeholder/draft content detection via regex scan + LLM screenshot judgment), a 16-row reporting table, and updated the Visual Focus stub note to remove layout-integrity from pending.**

## Performance

- **Duration:** ~5 min
- **Started:** 2026-04-20T10:46:07Z
- **Completed:** 2026-04-20T10:51:20Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Added `#### Realistic-Data Overflow Testing (LAYOUT-03)` with 7 numbered steps: injection target identification, tiered content injection (4-string vocabulary table), overflow detection via scrollWidth/scrollHeight after injection, screenshot supplement, section-type repeat loop, mandatory page reload mandate, and route interception escalation path
- Added `#### Placeholder & Draft Content Detection (LAYOUT-04)` with 3 numbered steps: comprehensive TreeWalker-based regex scan over visible text and alt/title/aria-label/meta attributes, `placeholder_allowlist` check before reporting, and LLM screenshot judgment for vague CTAs, casing inconsistency, and placeholder images
- Added `#### Reporting Layout & Content Integrity Findings` table with 16 rows (6 High confidence: clipping, injection overflow, regex match, placeholder image URL, missing landmark, missing shared region; 10 Medium confidence: sibling gap outlier, asymmetric padding, grid/flex inconsistency, nav link count deviation, nav link text absence, visual layout break, vague CTA, casing inconsistency, placeholder image visual, alignment/spacing screenshot judgment)
- Updated Visual Focus stub note at line 747 — removed `layout-integrity` from pending list, updated text to reference only `performance-responsive` and Phase 12

## Task Commits

Each task was committed atomically:

1. **Task 1 + Task 2 (combined):** - `69ec2b7` — feat(11-02): add LAYOUT-03, LAYOUT-04, reporting table, update stub note

## Files Created/Modified

- `agents/qa-tester.md` — Added LAYOUT-03 (lines 1612-1692), LAYOUT-04 (lines 1693-1779), Reporting table (lines 1780-1801), updated stub note at line 747 (192 insertions, 1 substitution)

## Decisions Made

- LAYOUT-03 DOM injection is the default path; route interception is escalation-only (spec must declare it) — avoids unintended API mocking
- Page reload after all LAYOUT-03 checks on a page is mandatory per T-11-04 threat mitigation — prevents injected content from contaminating LAYOUT-04 placeholder scan
- LAYOUT-04 regex scan scope extended to alt, title, aria-label, and meta[content] attributes per D-21 — placeholder content in these attributes is visible in tooltips, screen readers, and search results even when not visually rendered
- `placeholder_allowlist` spec field defers authority to the spec author for known-intentional values (product named TODO, CTA "Learn more" that is intentional)
- LLM casing check scoped to same-type elements per D-22 to prevent false positives from intentional typographic hierarchy (e.g., H1 uses different case than nav links by design)
- Reporting table rows reflect the same confidence model used throughout qa-tester.md: eval-confirmed results = High, screenshot/judgment results = Medium

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Layout & Content Integrity methodology is complete: LAYOUT-01 through LAYOUT-04 all implemented
- Visual Focus stub note correctly references only `performance-responsive` and Phase 12
- Phase 12 (performance-responsive) can now proceed with the same dual pattern structure established in Phases 9-11

## Self-Check

- [x] `#### Realistic-Data Overflow Testing (LAYOUT-03)` — 1 occurrence in agents/qa-tester.md
- [x] `#### Placeholder & Draft Content Detection (LAYOUT-04)` — 1 occurrence
- [x] `#### Reporting Layout & Content Integrity Findings` — 1 occurrence
- [x] `Hildegard von Bingen` tiered string — present
- [x] `Donaudampfschifffahrtsgesellschaft` tiered string — present
- [x] `$1,234,567,890.99` tiered string — present
- [x] `firstname.lastname+tag@very-long-subdomain.example.com` tiered string — present
- [x] `playwright-cli navigate [same url]` page reload mandate — present
- [x] `route-clear` cleanup mandate — present
- [x] `placeholder_allowlist` check step — present
- [x] `layout-integrity` no longer in stub note pending phrase — confirmed (0 matches)
- [x] `performance-responsive.*Phase 12` reference — present (2 occurrences: area list + stub note)
- [x] Section ordering: LAYOUT-01 (1429) → LAYOUT-02 (1554) → LAYOUT-03 (1612) → LAYOUT-04 (1693) → Reporting (1780) → Design Reference (1803)
- [x] Commit `69ec2b7` exists

## Self-Check: PASSED

---
*Phase: 11-layout-content-integrity*
*Completed: 2026-04-20*
