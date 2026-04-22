# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v0.3 — Visual & UX Design Verification

**Shipped:** 2026-04-22
**Phases:** 7 | **Plans:** 13

### What Was Built
- Tier 2 visual verification methodology: design reference comparison, typography, image/media quality, color/theme consistency, cross-page consistency
- UX state verification: empty/loading/error state triggering, 8-state interaction matrix, cursor/toast/sticky feedback, animation quality
- Layout & content integrity: spacing/alignment, structural consistency, realistic-data overflow, placeholder detection
- Performance & responsive: Core Web Vitals measurement, cross-browser rendering comparison, breakpoint sweep, UX anti-pattern detection
- Orchestration guidance: 601-line ORCHESTRATION.md reference + /qa:guide context-aware skill
- Documentation gap closure: all v0.3 capabilities documented across ORCHESTRATION.md, SPEC-FORMAT.md, README.md, CLAUDE.md

### What Worked
- Parallel phase execution (Phases 9-13 independent after Phase 8) — enabled fast throughput
- Tier 1/Tier 2 framing from accessibility carried cleanly to visual verification — consistent mental model
- Eval probes over screenshots for measurement — precise CSS values, computed styles, DOM state
- Milestone audit catching documentation gaps before close — Phase 14 created and closed all 6
- `playwright-cli run-code` for page-level APIs (emulateMedia, PerformanceObserver) — clean separation from element-level commands

### What Was Inefficient
- ROADMAP.md checkbox tracking didn't auto-update for Phases 8 and 9 — required manual correction
- REQUIREMENTS.md traceability checkboxes never updated during execution — all remained `[ ] Pending`
- Most SUMMARY.md files missing `requirements-completed` frontmatter — executor agents inconsistent
- `audit-open` CLI command has a runtime bug (`output is not defined`) — couldn't run pre-close audit

### Patterns Established
- Visual Focus / Accessibility Focus parallel: both use Tier 1-always / Tier 2-opt-in with identical three-case behavior
- Eval probe methodology: `getComputedStyle`, `scrollWidth > clientWidth`, `document.fonts.check()` patterns reused across subsections
- Confidence calibration: High = definitely real AND impactful, Medium = likely real OR less impactful
- `playwright-cli run-code` for page-object APIs vs `playwright-cli eval` for element-level queries
- Security filter for CSS variable reading: `!prop.match(/^--(auth|token|key|secret)/)`
- Lab-environment labeling for synthetic metrics (Web Vitals)

### Key Lessons
1. Documentation gaps compound — shipping methodology without updating user-facing docs creates a discoverability wall. Milestone audit caught this, but better to document as you build.
2. Tracking file hygiene (checkboxes, frontmatter) is low-value busywork when VERIFICATION.md is the authoritative source. Consider simplifying tracking requirements.
3. Methodology-only phases (no executable code) verify cleanly through file inspection — no test runner needed, but spot-checks on line numbers and grep counts are essential.

### Cost Observations
- Model mix: ~20% opus (orchestration, verification), ~80% sonnet (execution, review)
- Notable: Documentation-only phases execute fastest (~3-5 min per plan). Methodology phases with eval code blocks take slightly longer (~5-8 min per plan).

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Phases | Plans | Key Change |
|-----------|--------|-------|------------|
| v0.2 | 7 | 18 | Established plugin architecture, agent methodology patterns |
| v0.3 | 7 | 13 | Parallel phase execution, Tier 1/Tier 2 visual framing, milestone audit gap closure |

### Top Lessons (Verified Across Milestones)

1. Tier-based methodology (always-on baseline + opt-in deep checks) scales well across domains (accessibility, visual)
2. Spec-driven behavior — new capabilities should be toggleable via spec sections, not hardcoded
3. Documentation is a deliverable, not an afterthought — catches in audit cost an extra phase
