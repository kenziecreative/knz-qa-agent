---
phase: 9
slug: design-verification
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-19
---

# Phase 9 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual verification (no automated test runner — QA agent is itself the testing tool) |
| **Config file** | N/A |
| **Quick run command** | Review qa-tester.md for correct structure and wording |
| **Full suite command** | `/qa:run` on a test project with `## Visual Focus: design-verification` |
| **Estimated runtime** | ~5 minutes (manual review) |

---

## Sampling Rate

- **After every task commit:** Review qa-tester.md for correct structure
- **After every plan wave:** Verify all methodology sections are present and structured correctly
- **Before `/gsd-verify-work`:** Full manual review of qa-tester.md design verification sections
- **Max feedback latency:** N/A (manual verification)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| TBD | TBD | TBD | DESIGN-01 | — | N/A | manual | grep "Mockup Comparison" agents/qa-tester.md | N/A | ⬜ pending |
| TBD | TBD | TBD | DESIGN-02 | — | N/A | manual | grep "Typography Verification" agents/qa-tester.md | N/A | ⬜ pending |
| TBD | TBD | TBD | DESIGN-03 | — | N/A | manual | grep "Image.*Media" agents/qa-tester.md | N/A | ⬜ pending |
| TBD | TBD | TBD | DESIGN-04 | — | N/A | manual | grep "Color.*Theme" agents/qa-tester.md | N/A | ⬜ pending |
| TBD | TBD | TBD | DESIGN-05 | — | N/A | manual | grep "Cross-Page Consistency" agents/qa-tester.md | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. Phase 9 is a documentation/methodology edit to `agents/qa-tester.md` — no test framework setup needed.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Mockup comparison methodology present with structured category checklist | DESIGN-01 | Agent methodology is Markdown text — verified by reading | Read qa-tester.md Design Verification section, confirm 5-category checklist (typography, spacing, color, imagery, layout) |
| Typography probes use document.fonts.check() and scrollWidth detection | DESIGN-02 | JS eval snippets embedded in methodology — verified by reading | Read qa-tester.md Typography Verification section, confirm font load and overflow detection approaches |
| Image quality probes use naturalWidth/naturalHeight/complete | DESIGN-03 | JS eval snippets embedded in methodology — verified by reading | Read qa-tester.md Image & Media Quality section, confirm distortion threshold and upscale detection |
| Color/theme checks use CSS custom properties and emulateMedia | DESIGN-04 | Playwright interaction patterns embedded in methodology | Read qa-tester.md Color & Theme section, confirm hover-then-eval and dark mode toggle patterns |
| Cross-page consistency sampling strategy documented | DESIGN-05 | Methodology text — verified by reading | Read qa-tester.md Cross-Page Consistency section, confirm element sampling strategy |

---

## Validation Sign-Off

- [ ] All tasks have verification criteria defined
- [ ] Manual verification instructions are clear and actionable
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
