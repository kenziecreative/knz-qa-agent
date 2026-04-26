---
phase: 8
slug: spec-format-extensions
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-19
---

# Phase 8 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification (no automated test framework — this phase is documentation/text-only) |
| **Config file** | none |
| **Quick run command** | `grep -c "## Visual Focus" examples/SPEC-FORMAT.md` |
| **Full suite command** | `grep -l "Visual Focus\|Design Reference\|Browsers" agents/qa-tester.md examples/SPEC-FORMAT.md skills/run/SKILL.md` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run quick grep verification
- **After every plan wave:** Run full grep suite across all modified files
- **Before `/gsd-verify-work`:** Full suite must confirm all three sections present in all target files
- **Max feedback latency:** 1 second

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 08-01-01 | 01 | 1 | INFRA-01 | — | N/A | grep | `grep -c "## Visual Focus" examples/SPEC-FORMAT.md agents/qa-tester.md` | N/A | ⬜ pending |
| 08-01-02 | 01 | 1 | INFRA-02 | — | N/A | grep | `grep -c "## Design Reference" examples/SPEC-FORMAT.md agents/qa-tester.md` | N/A | ⬜ pending |
| 08-01-03 | 01 | 1 | INFRA-03 | — | N/A | grep | `grep -c "## Browsers" examples/SPEC-FORMAT.md agents/qa-tester.md` | N/A | ⬜ pending |
| 08-02-01 | 02 | 1 | INFRA-01 | — | N/A | grep | `grep -c "visual.focus\|Visual Focus" skills/run/SKILL.md` | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

Existing infrastructure covers all phase requirements. This phase modifies documentation and agent prompt text only — no test framework installation needed.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Backward compatibility — specs without new sections work unchanged | INFRA-01, INFRA-02, INFRA-03 | Requires running qa-agent against existing spec | Run `/qa:run` with an existing v1/v2 spec and verify no errors |
| SPEC-FORMAT.md readability | INFRA-01, INFRA-02, INFRA-03 | Subjective documentation quality | Read through new sections and verify examples are clear |

---

## Validation Sign-Off

- [ ] All tasks have automated verify commands
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 1s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
