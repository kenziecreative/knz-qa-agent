---
phase: 10
slug: ux-state-verification
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-19
---

# Phase 10 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification via qa-tester agent methodology review |
| **Config file** | none — methodology phase, no test framework needed |
| **Quick run command** | `grep -c "STATE-0" agents/qa-tester.md` |
| **Full suite command** | `grep -E "(STATE-01|STATE-02|STATE-03|STATE-04)" agents/qa-tester.md` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run `grep -c "STATE-0" agents/qa-tester.md`
- **After every plan wave:** Run `grep -E "(STATE-01|STATE-02|STATE-03|STATE-04)" agents/qa-tester.md`
- **Before `/gsd-verify-work`:** Full suite must be green
- **Max feedback latency:** 1 second

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| TBD | TBD | TBD | STATE-01 | — | N/A | grep | `grep "empty state\|loading state\|error state\|first-run" agents/qa-tester.md` | ⬜ | ⬜ pending |
| TBD | TBD | TBD | STATE-02 | — | N/A | grep | `grep "state matrix\|default.*hover.*focus" agents/qa-tester.md` | ⬜ | ⬜ pending |
| TBD | TBD | TBD | STATE-03 | — | N/A | grep | `grep "cursor\|toast\|sticky header" agents/qa-tester.md` | ⬜ | ⬜ pending |
| TBD | TBD | TBD | STATE-04 | — | N/A | grep | `grep "transition\|animation\|loading animation" agents/qa-tester.md` | ⬜ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

Existing infrastructure covers all phase requirements.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Methodology coherence | ALL | Agent prompt methodology requires human review for logical flow | Read the UX State Verification section end-to-end, verify instructions are actionable |
| Confidence level accuracy | ALL | Confidence assignments require domain judgment | Review each confidence level against the rationale provided |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 1s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
