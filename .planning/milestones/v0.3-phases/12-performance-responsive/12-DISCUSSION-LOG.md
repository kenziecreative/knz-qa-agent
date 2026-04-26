# Phase 12: Performance & Responsive - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 12-performance-responsive
**Areas discussed:** Core Web Vitals measurement, Breakpoint sweep approach, UX anti-pattern detection, Cross-browser rendering differences

---

## Core Web Vitals Measurement

| Option | Description | Selected |
|--------|-------------|----------|
| Raw PerformanceObserver (Recommended) | Use playwright-cli eval to read CLS/LCP/INP directly via browser APIs. No dependencies. Agent documents honest limitations (lab values, interaction requirements). | ✓ |
| Inject web-vitals library | Load the web-vitals npm package via CDN during test runs. Better lifecycle handling but adds network dependency and fails on CSP-restricted apps. | |

**User's choice:** Raw PerformanceObserver via eval
**Notes:** No external dependencies. Agent must document honest limitations: LCP needs body click, CLS needs visibilitychange trigger, INP only meaningful after real interactions. All values labeled as lab-environment.

---

## Breakpoint Sweep Approach

| Option | Description | Selected |
|--------|-------------|----------|
| Two-phase (Recommended) | Discrete breakpoints by default (fast). Continuous sweep activates when a discrete test fails OR when the spec opts in via breakpoint_sweep flag. | ✓ |
| Discrete breakpoints only | Test at 6 known widths (320-1440). Fast and predictable but misses mid-range overflow bugs. | |
| Continuous sweep always | Step through every 50px from 320-1440. Thorough but 20+ resize calls per page — slow and noisy. | |

**User's choice:** Two-phase approach
**Notes:** Discrete breakpoints (320, 375, 768, 1024, 1280, 1440) as default. Continuous sweep escalates on failure or spec opt-in. Overflow detection runs at every width.

---

## UX Anti-Pattern Detection

| Option | Description | Selected |
|--------|-------------|----------|
| Dual pattern, fixed list (Recommended) | Eval probes for structural checks + visual judgment for perceptual checks. Four required anti-patterns with fixed detection logic per pattern. Matches existing agent methodology. | ✓ |
| Dual pattern, spec-extensible | Same detection approach but specs can declare additional anti-patterns to check. Adds complexity without clear payoff at this stage. | |

**User's choice:** Dual pattern with fixed list
**Notes:** Four anti-patterns: modal without close button, pre-checked opt-ins, cookie banner blocking content, infinite scroll without position recovery. Each gets the detection method that fits (eval for structural, behavioral for interaction-dependent).

---

## Cross-Browser Rendering Differences

| Option | Description | Selected |
|--------|-------------|----------|
| Eval comparison (Recommended) | Compare computed styles across engines via eval. Chromium = baseline. Targets ~15-20 CSS properties. High confidence findings with clear attribution. Intentional divergences exempted. | ✓ |
| Visual judgment only | Screenshot comparison between engines. Catches user-visible differences but Medium confidence only — can't attribute to specific CSS properties. | |

**User's choice:** Eval comparison with Chromium baseline
**Notes:** ~15-20 property targets (fontFamily, lineHeight, gap, borderRadius, etc.). Intentional divergences exempted (form controls, scrollbars, system fonts). Screenshot supplement for perceptual differences.

---

## Claude's Discretion

- Exact JS evaluate snippets for PerformanceObserver, computed style comparison, anti-pattern detection
- Web Vitals score thresholds (Google's published good/needs-improvement/poor)
- Exact CSS property list for cross-browser comparison
- Methodology section structure and ordering in qa-tester.md
- Breakpoint sweep increment size for continuous mode
- PERF-* confidence table entries

## Deferred Ideas

None — discussion stayed within phase scope
