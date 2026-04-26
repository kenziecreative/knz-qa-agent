---
phase: 11
slug: layout-content-integrity
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-20
---

# Phase 11 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification (methodology documentation phase) |
| **Config file** | none — no automated test suite for agent methodology text |
| **Quick run command** | `grep -c "### Layout & Content Integrity" agents/qa-tester.md` |
| **Full suite command** | `grep -c "LAYOUT-0[1-4]" agents/qa-tester.md` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run `grep -c "### Layout & Content Integrity" agents/qa-tester.md`
- **After every plan wave:** Run `grep -c "LAYOUT-0[1-4]" agents/qa-tester.md`
- **Before `/gsd-verify-work`:** Full suite must be green
- **Max feedback latency:** 1 second

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 11-01-01 | 01 | 1 | LAYOUT-01 | — | N/A | grep | `grep "Spacing & Alignment" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 11-01-02 | 01 | 1 | LAYOUT-02 | — | N/A | grep | `grep "Cross-Page Structural" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 11-02-01 | 02 | 1 | LAYOUT-03 | — | N/A | grep | `grep "Realistic-Data Overflow" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 11-02-02 | 02 | 1 | LAYOUT-04 | — | N/A | grep | `grep "Placeholder.*Content" agents/qa-tester.md` | ❌ W0 | ⬜ pending |
| 11-03-01 | 03 | 1 | LAYOUT-01,02,03,04 | — | N/A | grep | `grep "LAYOUT-0[1-4]" agents/qa-tester.md` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements — this is a methodology documentation phase modifying `agents/qa-tester.md` only.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Methodology sections follow dual pattern structure | LAYOUT-01 through LAYOUT-04 | Structural/stylistic review | Read new sections and verify eval+visual pattern matches Phase 9/10 format |
| Confidence table entries are consistent | LAYOUT-01 through LAYOUT-04 | Semantic review | Verify each LAYOUT-* row has appropriate confidence level with rationale |
| Phase 11 pending stub removed | N/A | Negative test | Confirm line 747 stub text no longer present in qa-tester.md |

*Manual verifications required because this phase produces agent methodology text, not executable code.*

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 1s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
