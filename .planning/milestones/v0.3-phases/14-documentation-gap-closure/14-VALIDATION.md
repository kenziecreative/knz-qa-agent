---
phase: 14
slug: documentation-gap-closure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-21
---

# Phase 14 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual grep verification (no test framework — documentation-only phase) |
| **Config file** | none |
| **Quick run command** | `grep -c "Visual Focus" docs/ORCHESTRATION.md` |
| **Full suite command** | See Per-Task Verification Map below |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run grep verification for that task's acceptance criteria
- **After every plan wave:** Run all grep checks for all completed tasks
- **Before `/gsd-verify-work`:** All grep checks must pass
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 14-01-01 | 01 | 1 | INFRA-04 | — | N/A | grep | `grep -c "Visual Focus" docs/ORCHESTRATION.md` | ✅ | ⬜ pending |
| 14-01-02 | 01 | 1 | INFRA-04 | — | N/A | grep | `grep -c "\-\-browsers" docs/ORCHESTRATION.md` | ✅ | ⬜ pending |
| 14-02-01 | 02 | 1 | INFRA-01, INFRA-02, INFRA-03 | — | N/A | grep | `grep -c "breakpoint-sweep" docs/SPEC-FORMAT.md` | ✅ | ⬜ pending |
| 14-02-02 | 02 | 1 | INFRA-01 | — | N/A | grep | `grep -c "placeholder_allowlist" docs/SPEC-FORMAT.md` | ✅ | ⬜ pending |
| 14-03-01 | 03 | 1 | INFRA-05 | — | N/A | grep | `grep -c "Visual Focus" README.md` | ✅ | ⬜ pending |
| 14-03-02 | 03 | 1 | INFRA-05 | — | N/A | grep | `grep -c "v0.3" CLAUDE.md` | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. No test framework needed — all verification is grep-based against documentation files.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Visual testing section is comprehensive and coherent | INFRA-04 | Content quality requires human review | Read ORCHESTRATION.md visual testing section end-to-end |
| Examples are accurate and runnable | INFRA-01, INFRA-02, INFRA-03 | Spec examples need semantic review | Review SPEC-FORMAT.md examples match agent behavior |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
