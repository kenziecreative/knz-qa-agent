---
phase: 12-performance-responsive
plan: "01"
subsystem: agents/qa-tester.md
tags: [performance, web-vitals, cross-browser, tier2, methodology]
dependency_graph:
  requires: []
  provides: [PERF-01-methodology, PERF-02-methodology, performance-responsive-section]
  affects: [agents/qa-tester.md]
tech_stack:
  added: []
  patterns: [PerformanceObserver-injection, getComputedStyle-diff, engine-tagged-findings, lab-value-labeling]
key_files:
  created: []
  modified: [agents/qa-tester.md]
decisions:
  - All Web Vitals labeled as lab values, not field/CrUX data — prevents misleading conclusions in reports
  - PERF-02 skips on single-engine runs — comparison requires Chromium baseline to be meaningful
  - INP=0 with no interactions reported as "not measured" not "Good" — avoids false passing scores
  - visibilitychange dispatch chosen for CLS finalization over page.close() — preserves session
  - Intentional divergence exemption list prevents false positives on native controls and vendor prefixes
metrics:
  duration: ~5 minutes
  completed_date: "2026-04-20"
  tasks_completed: 2
  files_modified: 1
requirements: [PERF-01, PERF-02]
---

# Phase 12 Plan 01: Performance & Responsive (PERF-01/PERF-02) Summary

**One-liner:** PerformanceObserver-based Web Vitals measurement with element attribution and getComputedStyle cross-browser CSS diff against Chromium baseline.

## What Was Built

Added the `### Performance & Responsive` Tier 2 section to `agents/qa-tester.md`, completing the first half of the performance-responsive methodology. This enables the QA agent to:

1. Measure Lab LCP, Lab CLS, and Lab INP using raw PerformanceObserver (no external libraries)
2. Attribute poor scores to specific DOM elements via entry.element and entry.sources
3. Compare computed CSS properties across browser engines against a Chromium baseline
4. Flag cross-browser divergences using engine-tagged finding format

## Tasks

### Task 1: Remove stub note and add Performance & Responsive section header + PERF-01 Core Web Vitals

**Commit:** 556ba98

**Changes:**
- Deleted "pending Phase 12" stub note (2 lines) from Visual Focus depth-toggle section
- Inserted `### Performance & Responsive` section with dual-pattern intro, security note, and lab-environment note
- Added `#### Core Web Vitals (PERF-01)` subsection with full PerformanceObserver lifecycle:
  - Inject observers for LCP, CLS, INP using `window.__qa*` globals with try/catch safety
  - LCP finalization via user click (browser requirement)
  - CLS finalization via `visibilitychange` dispatch
  - Read globals and evaluate against thresholds
  - Element attribution from entry.element/entry.sources
  - Firefox caveats (LCP/CLS not supported, INP via `event` type)
- Extended Confidence Summary Table with 6 new PERF rows

### Task 2: Add PERF-02 Cross-Browser Rendering Comparison subsection

**Commit:** 27a6a66

**Changes:**
- Inserted `#### Cross-Browser Rendering Comparison (PERF-02)` immediately after PERF-01 confidence line
- 18-property target list covering typography, layout, and rendering categories
- `getComputedStyle` eval probe querying key structural selectors
- Chromium-as-baseline comparison with format-only difference normalization (rgb vs hex, 700 vs bold)
- Intentional divergence exemption list (native form controls, scrollbars, system fonts, -webkit- props)
- Engine-tagged finding format: `[webkit] [selector] — [property]: "value" vs chromium "value"`
- Screenshot supplement for perceptual differences eval probes miss
- Single-engine skip guard with note

## Deviations from Plan

None - plan executed exactly as written.

## Known Stubs

None. All methodology is fully specified with concrete procedures, thresholds, and code snippets.

## Threat Surface Scan

No new network endpoints, auth paths, file access patterns, or schema changes introduced. The security note for `--auth`/`--token`/`--key`/`--secret` CSS custom properties is included in the Performance & Responsive section header, matching the T-12-01 mitigation from the threat model.

## Self-Check: PASSED

- `agents/qa-tester.md` exists and was modified: confirmed
- Commit 556ba98 exists: confirmed (Task 1)
- Commit 27a6a66 exists: confirmed (Task 2)
- `### Performance & Responsive` count = 1: confirmed
- `#### Core Web Vitals (PERF-01)` count = 1: confirmed
- `#### Cross-Browser Rendering Comparison (PERF-02)` count = 1: confirmed
- Stub note removed (0 matches for "Tier 2 visual methodology for `performance-responsive` will be added in Phase 12"): confirmed
- `Lab LCP` count >= 2: confirmed (3)
- `Lab CLS` count >= 2: confirmed (2)
- `Lab INP` count >= 2: confirmed (3)
- `### Performance & Responsive` appears before `### Design Reference` (line 1807 vs 2007): confirmed
- `**Security note:**` in Performance & Responsive section: confirmed
- `**Lab-environment note:**` count = 1: confirmed
- Confidence table contains `Lab LCP > 2.5s (PerformanceObserver entry) | High`: confirmed
- `Cross-browser CSS property divergence (getComputedStyle diff) | High` in confidence table: confirmed
