---
phase: 12
slug: performance-responsive
status: verified
threats_open: 0
threats_total: 8
asvs_level: 1
created: 2026-04-20
---

# Phase 12 — Security

> Per-phase security contract: threat register, accepted risks, and audit trail.

---

## Trust Boundaries

| Boundary | Description | Data Crossing |
|----------|-------------|---------------|
| Agent → Page (eval) | Agent injects JS into page context via playwright-cli eval/run-code | PerformanceObserver setup, getComputedStyle reads, getBoundingClientRect probes |
| Page → Agent (results) | Page returns eval results that agent interprets and reports | Metric values, element identifiers, CSS property values, dimension data |

---

## Threat Register

| Threat ID | Category | Component | Disposition | Mitigation | Status |
|-----------|----------|-----------|-------------|------------|--------|
| T-12-01 | Information Disclosure | PERF-02 getComputedStyle eval | mitigate | Security note: skip properties beginning with `--auth`, `--token`, `--key`, `--secret` (line 1821) | closed |
| T-12-02 | Information Disclosure | PERF-01 PerformanceObserver element attribution | accept | Only exposes tagName + id + className — no textContent or user data | closed |
| T-12-03 | Denial of Service | PERF-01 observer accumulation | accept | Fixed-size objects (single LCP/CLS/INP entry); no unbounded accumulation | closed |
| T-12-04 | Tampering | Page JS clearing window.__qa* globals | accept | Non-conflicting namespace; if cleared, agent reports "not measured" — no false assertion | closed |
| T-12-05 | Information Disclosure | PERF-03 overflow detection eval | accept | Only reads scrollWidth/clientWidth dimensions; text snippets truncated to 50 chars | closed |
| T-12-06 | Information Disclosure | PERF-04 checkbox label reading | mitigate | Label text truncated to 80 chars via `.slice(0, 80)` (line 2169); only consent-context checkboxes read | closed |
| T-12-07 | Tampering | PERF-04 infinite scroll navigation | accept | Standard browser history.back() — no data mutation | closed |
| T-12-08 | Denial of Service | PERF-03 overlap detection O(n²) loop | mitigate | Loop capped at `Math.min(rects.length, 200)` (lines 2058-2059) | closed |

*Status: open · closed*

---

## Accepted Risks

| Threat ID | Risk | Rationale |
|-----------|------|-----------|
| T-12-02 | Element structural info (tagName/id/className) exposed in reports | Non-sensitive; required for actionable findings |
| T-12-03 | Observer globals persist until page unload | Fixed-size; no memory growth vector |
| T-12-04 | Page JS could clear measurement globals | Low probability; graceful degradation (reports "not measured") |
| T-12-05 | Scroll dimensions visible in reports | Dimensional data is non-sensitive layout info |
| T-12-07 | Back-navigation changes page state | Expected test behavior; no persistent mutation |

---

## Audit Trail

### Security Audit 2026-04-20

| Metric | Count |
|--------|-------|
| Threats found | 8 |
| Closed | 8 |
| Open | 0 |

**Verification method:** Manual grep/code inspection of implementation against PLAN.md threat register.

**Mitigations verified:**
- T-12-01: `--auth/--token/--key/--secret` filter present in Performance & Responsive security note
- T-12-06: `.slice(0, 80)` truncation on consent checkbox label text
- T-12-08: `Math.min(rects.length, 200)` cap on overlap detection loop
