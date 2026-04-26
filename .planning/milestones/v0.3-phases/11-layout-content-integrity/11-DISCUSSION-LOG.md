# Phase 11: Layout & Content Integrity - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 11-layout-content-integrity
**Areas discussed:** Spacing & alignment checks, Cross-page structural consistency, Realistic-data overflow testing, Placeholder & draft detection

---

## Spacing & Alignment Checks

| Option | Description | Selected |
|--------|-------------|----------|
| Dual: eval + screenshot | evaluate() compares sibling spacing via getBoundingClientRect, flags outliers. Screenshot supplements for clipping and visual misalignment. Matches the established pattern from Phases 9-10. | ✓ |
| Eval-only | Pure deterministic: sibling comparison, grid/flex property checks, overflow detection. No screenshot visual judgment for this area. | |

**User's choice:** Dual: eval + screenshot (Recommended)
**Notes:** Follows established dual pattern. Eval for sibling gap comparison and overflow detection (High confidence), screenshot for complex layout visual assessment (Medium confidence).

---

## Cross-Page Structural Consistency

| Option | Description | Selected |
|--------|-------------|----------|
| Fingerprint + content sampling | Eval checks landmark presence/counts AND samples nav/footer link text per page. Catches both missing components AND content drift. High confidence for structural absence, Medium for link parity. | ✓ |
| DOM fingerprint only | Eval checks landmark roles, element counts, child counts per region. Catches missing sections but not content drift within them. | |

**User's choice:** Fingerprint + content sampling (Recommended)
**Notes:** Distinct from DESIGN-05 which covers CSS value consistency. LAYOUT-02 covers structural/DOM consistency — same header structure, same footer presence, same nav links across pages.

---

## Realistic-Data Overflow Testing

| Option | Description | Selected |
|--------|-------------|----------|
| eval() injection primary | Inject test content via evaluate(textContent) per section. Tiered test strings: long Latin, German compounds, large numbers, long emails. Detect overflow via scrollWidth > clientWidth. Route interception available as escalation for locale-specific specs. | ✓ |
| Route interception primary | Mock API responses with long data. Tests full render pipeline but harder to isolate failures and requires per-endpoint setup + cleanup. | |

**User's choice:** eval() injection primary (Recommended)
**Notes:** Inject one logical section at a time for clear attribution. Route interception remains available as spec-driven escalation path.

---

## Placeholder & Draft Content Detection

| Option | Description | Selected |
|--------|-------------|----------|
| Dual: regex + LLM judgment | eval() regex scan for Lorem ipsum, TODO/FIXME, placeholder image URLs (High confidence). LLM screenshot judgment for vague CTAs and casing inconsistency (Medium confidence). Spec-level allowlist for false positives. | ✓ |
| Regex-only | Pure keyword/regex scan. Fast and deterministic but can't distinguish legitimate uses from draft content. No CTA or casing judgment. | |

**User's choice:** Dual: regex + LLM judgment (Recommended)
**Notes:** Scan scope includes alt, title, aria-label, and meta tags beyond visible text. Spec-level allowlist for known false positives.

---

## Claude's Discretion

- Exact JS evaluate snippets for sibling comparison and structural fingerprinting
- Threshold values for spacing deviation
- Exact regex patterns for placeholder detection
- Section naming and ordering within qa-tester.md
- Sibling element set prioritization
- Link text normalization for cross-page comparison

## Deferred Ideas

None — discussion stayed within phase scope
