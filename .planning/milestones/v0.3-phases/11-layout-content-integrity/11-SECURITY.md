---
phase: 11
slug: layout-content-integrity
status: verified
threats_open: 0
asvs_level: 1
created: 2026-04-20
---

# Phase 11 — Security

> Per-phase security contract: threat register, accepted risks, and audit trail.

---

## Trust Boundaries

| Boundary | Description | Data Crossing |
|----------|-------------|---------------|
| eval code execution | Agent-generated JS runs in browser context via playwright-cli eval — read-only DOM probes (LAYOUT-01/02) and DOM mutation (LAYOUT-03) | DOM structure, CSS computed styles, text content |
| eval regex scanning | LAYOUT-04 scans page content including meta tags and attributes | Visible text, alt/title/aria-label attributes, meta content, img src URLs |

---

## Threat Register

| Threat ID | Category | Component | Disposition | Mitigation | Status |
|-----------|----------|-----------|-------------|------------|--------|
| T-11-01 | Information Disclosure | eval CSS property reads | mitigate | Security note in section header: exclude `--auth`, `--token`, `--key`, `--secret` CSS custom property names from reports (line 1425) | closed |
| T-11-02 | Tampering | Plan 01 read-only probes | accept | Plan 01 probes are read-only (getBoundingClientRect, getComputedStyle, querySelectorAll). No DOM mutation in LAYOUT-01 or LAYOUT-02 | closed |
| T-11-03 | Information Disclosure | Link text sampling (LAYOUT-02) | accept | Link text extracted from nav/header/footer is navigation labels, not sensitive data. No PII exposure risk | closed |
| T-11-04 | Tampering | LAYOUT-03 DOM injection | mitigate | Environment note in section header (line 1427): "DOM content injection mutates live page state. Use only in test environments." Page reload mandate after all LAYOUT-03 checks (line 1681) | closed |
| T-11-05 | Information Disclosure | LAYOUT-04 meta content scanning | accept | Meta content is public HTML — no sensitive data exposure beyond what's already in the page source | closed |
| T-11-06 | Tampering | LAYOUT-03 route interception escalation | mitigate | Route interception cleanup mandate: `playwright-cli route-clear` after testing (line 1689). Follows established STATE-01 pattern | closed |
| T-11-07 | Information Disclosure | LAYOUT-04 attribute scanning | accept | Scans alt, title, aria-label — public accessibility attributes, no PII. Findings report only snippet (first 80 chars) | closed |

*Status: open / closed*
*Disposition: mitigate (implementation required) / accept (documented risk) / transfer (third-party)*

---

## Accepted Risks Log

| Risk ID | Threat Ref | Rationale | Accepted By | Date |
|---------|------------|-----------|-------------|------|
| AR-11-01 | T-11-02 | Read-only DOM probes cannot modify page state — low tampering risk | GSD orchestrator | 2026-04-20 |
| AR-11-02 | T-11-03 | Navigation link text is public UI content, not PII | GSD orchestrator | 2026-04-20 |
| AR-11-03 | T-11-05 | Meta content is in public page source — no additional exposure from scanning | GSD orchestrator | 2026-04-20 |
| AR-11-04 | T-11-07 | Accessibility attributes are public by design; snippet truncation limits exposure | GSD orchestrator | 2026-04-20 |

---

## Security Audit Trail

| Audit Date | Threats Total | Closed | Open | Run By |
|------------|---------------|--------|------|--------|
| 2026-04-20 | 7 | 7 | 0 | GSD orchestrator |

---

## Sign-Off

- [x] All threats have a disposition (mitigate / accept / transfer)
- [x] Accepted risks documented in Accepted Risks Log
- [x] `threats_open: 0` confirmed
- [x] `status: verified` set in frontmatter

**Approval:** verified 2026-04-20
