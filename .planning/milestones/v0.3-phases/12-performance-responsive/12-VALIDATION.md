---
phase: 12
slug: performance-responsive
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-20
---

# Phase 12 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification via playwright-cli |
| **Config file** | none — this phase modifies agent methodology (markdown), not executable code |
| **Quick run command** | `grep -c "### Performance & Responsive" agents/qa-tester.md` |
| **Full suite command** | `grep -c "PERF-01\|PERF-02\|PERF-03\|PERF-04" agents/qa-tester.md` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run quick run command (verify section exists)
- **After every plan wave:** Run full suite command (verify all PERF requirements referenced)
- **Before `/gsd-verify-work`:** Full suite must be green
- **Max feedback latency:** 1 second

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 12-01-01 | 01 | 1 | PERF-01 | — | N/A | grep | `grep "Core Web Vitals" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 12-01-02 | 01 | 1 | PERF-01 | — | N/A | grep | `grep "PerformanceObserver\|__qaLCP\|__qaCLS\|__qaINP" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 12-02-01 | 02 | 1 | PERF-02 | — | N/A | grep | `grep "cross-browser\|getComputedStyle" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 12-03-01 | 03 | 1 | PERF-03 | — | N/A | grep | `grep "breakpoint\|viewport sweep" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 12-04-01 | 04 | 1 | PERF-04 | — | N/A | grep | `grep "anti-pattern\|modal.*close\|cookie.*banner\|opt-in\|infinite.*scroll" agents/qa-tester.md` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. This phase modifies markdown methodology in qa-tester.md — no test framework installation needed.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| PerformanceObserver JS snippets produce valid metrics | PERF-01 | Requires live browser page with real content | Run /qa:run against a real site with `## Visual Focus: performance-responsive` |
| Cross-browser diff detects real divergences | PERF-02 | Requires multi-engine Playwright session | Run /qa:run with `## Browsers: chromium, webkit` on a page with known rendering differences |
| Breakpoint sweep catches real overflow | PERF-03 | Requires page with responsive layout | Run /qa:run on a responsive page and verify overflow findings |
| UX anti-pattern detection on real pages | PERF-04 | Requires pages exhibiting anti-patterns | Run /qa:run on pages with known modals, cookie banners, etc. |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 1s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
