---
phase: 13
slug: usage-guidance
status: verified
threats_open: 0
asvs_level: 1
created: 2026-04-21
---

# Phase 13 — Security

> Per-phase security contract: threat register, accepted risks, and audit trail.

---

## Trust Boundaries

| Boundary | Description | Data Crossing |
|----------|-------------|---------------|
| Documentation content | Static markdown (docs/ORCHESTRATION.md, README.md, CLAUDE.md) — no executable code | None — read-only reference |
| Skill file-read scope | /qa:guide reads .qa/ directory and docs/ORCHESTRATION.md | Local filesystem reads only — no secrets (.qa/env.local excluded) |

---

## Threat Register

| Threat ID | Category | Component | Disposition | Mitigation | Status |
|-----------|----------|-----------|-------------|------------|--------|
| T-13-01 | I (Information Disclosure) | docs/ORCHESTRATION.md | accept | No secrets/credentials/internal paths. Example credentials use placeholder values. | closed |
| T-13-02 | T (Tampering) | docs/ORCHESTRATION.md | accept | Version-controlled in git. Tampering visible in diffs. No runtime execution. | closed |
| T-13-03 | I (Information Disclosure) | skills/guide/SKILL.md | accept | Reads only .qa/ contents and ORCHESTRATION.md. .qa/env.local not in files-read list. | closed |
| T-13-04 | T (Tampering) | README.md, CLAUDE.md | accept | Version-controlled. Changes visible in git diff. Static documentation only. | closed |
| T-13-05 | S (Spoofing) | skills/guide/SKILL.md | accept | No authentication/authorization. Reads local files, produces text. Same trust as any file-read. | closed |

*Status: open · closed*
*Disposition: mitigate (implementation required) · accept (documented risk) · transfer (third-party)*

---

## Accepted Risks Log

| Risk ID | Threat Ref | Rationale | Accepted By | Date |
|---------|------------|-----------|-------------|------|
| AR-13-01 | T-13-01 | Public documentation, no secrets | Phase planning | 2026-04-20 |
| AR-13-02 | T-13-02 | Git version control provides tamper visibility | Phase planning | 2026-04-20 |
| AR-13-03 | T-13-03 | Skill reads non-sensitive files only (.qa/env.local excluded from scope) | Phase planning | 2026-04-20 |
| AR-13-04 | T-13-04 | Static docs in version control — no runtime risk | Phase planning | 2026-04-20 |
| AR-13-05 | T-13-05 | File-read-only operation with no auth surface | Phase planning | 2026-04-20 |

*Accepted risks do not resurface in future audit runs.*

---

## Security Audit Trail

| Audit Date | Threats Total | Closed | Open | Run By |
|------------|---------------|--------|------|--------|
| 2026-04-21 | 5 | 5 | 0 | gsd-secure-phase |

---

## Sign-Off

- [x] All threats have a disposition (mitigate / accept / transfer)
- [x] Accepted risks documented in Accepted Risks Log
- [x] `threats_open: 0` confirmed
- [x] `status: verified` set in frontmatter

**Approval:** verified 2026-04-21
