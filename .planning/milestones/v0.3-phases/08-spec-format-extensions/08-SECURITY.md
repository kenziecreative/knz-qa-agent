---
phase: 08
slug: spec-format-extensions
status: verified
threats_open: 0
asvs_level: 1
created: 2026-04-20
---

# Phase 08 — Security

> Per-phase security contract: threat register, accepted risks, and audit trail.

---

## Trust Boundaries

| Boundary | Description | Data Crossing |
|----------|-------------|---------------|
| N/A | No trust boundaries affected — pure documentation phase | No runtime data flow changes |

---

## Threat Register

| Threat ID | Category | Component | Disposition | Mitigation | Status |
|-----------|----------|-----------|-------------|------------|--------|
| T-08-01 | Information Disclosure | Design Reference paths in spec files | accept | Image paths are relative project paths, not secrets. Spec files are project-local and gitignored by convention (.qa/ directory). | closed |
| T-08-02 | Tampering | Spec sections could instruct agent to execute unintended commands | accept | Agent interprets spec sections as test instructions within existing sandboxed execution model (playwright-cli via Bash). No new tool grants or permission changes. Spec format is read-only input. | closed |

*Status: open · closed*
*Disposition: mitigate (implementation required) · accept (documented risk) · transfer (third-party)*

---

## Accepted Risks Log

| Risk ID | Threat Ref | Rationale | Accepted By | Date |
|---------|------------|-----------|-------------|------|
| AR-08-01 | T-08-01 | Design reference image paths are relative project paths, not secrets. Spec files live in .qa/ which is project-local. No sensitive data exposure risk. | GSD orchestrator | 2026-04-20 |
| AR-08-02 | T-08-02 | Spec sections are interpreted as test instructions within the agent's existing sandboxed execution model. No new tool grants, permission escalations, or command injection vectors introduced. The spec format is read-only input to the agent. | GSD orchestrator | 2026-04-20 |

*Accepted risks do not resurface in future audit runs.*

---

## Security Audit Trail

| Audit Date | Threats Total | Closed | Open | Run By |
|------------|---------------|--------|------|--------|
| 2026-04-20 | 2 | 2 | 0 | GSD orchestrator |

---

## Sign-Off

- [x] All threats have a disposition (mitigate / accept / transfer)
- [x] Accepted risks documented in Accepted Risks Log
- [x] `threats_open: 0` confirmed
- [x] `status: verified` set in frontmatter

**Approval:** verified 2026-04-20
