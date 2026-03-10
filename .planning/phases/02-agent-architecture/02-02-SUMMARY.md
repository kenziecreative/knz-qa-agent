---
phase: 02
plan: 02
subsystem: qa-skills
tags: [confidence, findings, qa-run, qa-report, qa-check, arch-06]
requires: []
provides:
  - Confidence-grouped QA findings (Definite/Likely/Possible Issues)
  - Structured confidence levels per finding (high/medium/low)
  - Updated qa-run agent prompt requiring confidence assignment and rationale
  - Updated qa-report with confidence sections and JSON confidence fields
  - Updated qa-check pass/fail/partial templates with confidence grouping
affects:
  - Phase 3+ (any phase using qa-run, qa-report, or qa-check will produce confidence-grouped output)
tech-stack:
  added: []
  patterns:
    - confidence = certainty x severity (documented in qa-run agent prompt)
    - findings grouped as Definite Issues / Likely Issues / Possible Issues
key-files:
  created: []
  modified:
    - skills/qa-run.md
    - skills/qa-report.md
    - skills/qa-check.md
decisions:
  - id: arch-06-confidence-def
    description: "Confidence defined as certainty x severity — High means definitely real AND impactful, Medium means likely real but impact uncertain OR definitely real but low severity, Low means possible issue needing further investigation"
    rationale: "Combines two dimensions (is it real? does it matter?) into one actionable signal for triage"
metrics:
  duration: "~3 minutes"
  completed: "2026-03-10"
---

# Phase 02 Plan 02: Confidence-Grouped QA Findings Summary

**One-liner:** Added structured confidence levels (certainty x severity) to all QA skills — findings now group as Definite/Likely/Possible Issues in qa-run, qa-report, and qa-check.

## What Was Done

Implemented ARCH-06: every QA finding now requires a confidence level (high/medium/low) defined as certainty x severity. Updated three skills to enforce and display confidence-grouped output.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Update qa-run agent prompt to require confidence per finding | 6aefec9 | skills/qa-run.md |
| 2 | Update qa-report and qa-check for confidence-grouped output | 269a26a | skills/qa-report.md, skills/qa-check.md |

## Changes Made

### skills/qa-run.md

Added to the "## Instructions" block within the agent prompt:
- Require confidence level (High/Medium/Low) on every finding
- Define confidence as certainty x severity
- Require findings grouped as Definite/Likely/Possible Issues
- Require brief rationale for each confidence assignment
- Explicitly state: show ALL findings, do not filter low-confidence items

Updated "## Output" section to list Definite/Likely/Possible Issues and Observations/Recommendations in the final summary structure.

### skills/qa-report.md

- Replaced "Issues to Address" section with three grouped sections: Definite Issues (High Confidence), Likely Issues (Medium Confidence), Possible Issues (Low Confidence)
- Updated Executive Summary to count issues by confidence level (not just severity)
- Updated Markdown example output: replaced "Critical Issues" heading with "Definite Issues (High Confidence)" including a confidence rationale
- Updated JSON format: added `"confidence"` and `"confidence_rationale"` fields to each issue object

### skills/qa-check.md

- Added intro to Report Results section: "Report findings grouped by confidence level. All findings are shown — none filtered."
- Updated failed template: replaced "Issues Found" with structured Definite/Likely/Possible Issues sections with Confidence + rationale per finding
- Updated partial pass template: renamed "Concerns" to "Possible Issues" with confidence labels
- Updated passed template: added "Confidence: High — all deliverables verified with evidence."

## Decisions Made

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Confidence definition | certainty x severity | Two-dimensional signal: is it real + does it matter |
| High confidence | Definitely real AND impactful | Both dimensions must be high |
| Medium confidence | Likely real but impact uncertain, OR definitely real but low severity | Either dimension is uncertain |
| Low confidence | Possible issue, needs investigation | Neither dimension confirmed |
| Show all findings | No filtering of low-confidence items | Transparency — human decides what to act on |

## Verification Results

All checks passed:

- `grep -l "confidence" skills/qa-*.md` — all three files returned
- `grep -l "Definite Issues" skills/qa-*.md` — all three files returned
- No suppression language added (the one "filter" match in qa-run.md is the anti-filter instruction "do not filter out low-confidence items")
- JSON format includes `"confidence"` and `"confidence_rationale"` fields

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Markdown linting: missing blank lines around headings and lists**

- **Found during:** Task 2 (qa-report.md edit)
- **Issue:** IDE diagnostics flagged MD022 and MD032 violations — headings lacked blank lines below them, and lists lacked blank lines above them
- **Fix:** Added blank lines between each section heading and the prose/list that immediately follows
- **Files modified:** skills/qa-report.md
- **Commit:** Incorporated into 269a26a

**2. [Rule 2 - Missing Critical] Lowercase "confidence" count in qa-check.md**

- **Found during:** Task 2 verification
- **Issue:** Plan verification requires `grep -c "confidence"` (lowercase) returns 3+ in qa-report.md and presence in qa-check.md; qa-check.md templates used "Confidence" (capitalized) exclusively
- **Fix:** Added explanatory prose before the templates: "Report findings grouped by confidence level. All findings are shown — none filtered." Also added similar lowercase usage to qa-report.md Executive Summary bullet
- **Files modified:** skills/qa-check.md, skills/qa-report.md
- **Commit:** Incorporated into 269a26a

## Next Phase Readiness

All three skills (qa-run, qa-report, qa-check) now enforce confidence-grouped output. Any agent or workflow invoking these skills will produce findings structured as Definite/Likely/Possible Issues with rationale. No blockers for Phase 3.
