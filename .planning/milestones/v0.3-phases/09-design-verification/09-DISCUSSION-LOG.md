# Phase 9: Design Verification - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-19
**Phase:** 09-design-verification
**Areas discussed:** Mockup comparison approach, Typography & font checks, Image & media quality, Color & theme consistency

---

## Mockup Comparison Approach

| Option | Description | Selected |
|--------|-------------|----------|
| Structured category checklist (Recommended) | Agent scans reference vs live page using fixed categories (typography, spacing, color, imagery, layout) in order — reproducible findings with confidence levels | ✓ |
| Holistic visual scan | Agent looks at both images and reports whatever it notices — faster but variable coverage between runs | |

**User's choice:** Structured category checklist
**Notes:** Research showed holistic scanning produces variable findings because what the model notices first varies. Structured checklist gives fixed scan order for reproducible coverage.

---

## Typography & Font Checks

| Option | Description | Selected |
|--------|-------------|----------|
| JS evaluate + visual combo (Recommended) | evaluate() for verifiable font assertions (loaded, overflow, family) — screenshots for cross-page consistency and readability judgment | ✓ |
| Visual judgment only | Screenshot-based comparison only — simpler but misses subtle font drift and can't confirm font load status | |

**User's choice:** JS evaluate + visual combo
**Notes:** document.fonts API provides deterministic font load verification. evaluate() catches overflow via scrollWidth > clientWidth. Visual judgment supplements for readability and cross-page consistency.

---

## Image & Media Quality

| Option | Description | Selected |
|--------|-------------|----------|
| Evaluate-first + screenshot fallback (Recommended) | JS probes catch broken images, wrong dimensions, missing lazy-load deterministically — visual judgment supplements for perceptual quality (blur, artifacts) | ✓ |
| Visual judgment only | Screenshot-based detection only — catches perceptual issues but misses DOM-level problems | |

**User's choice:** Evaluate-first + screenshot fallback
**Notes:** naturalWidth/naturalHeight vs rendered dimensions provides deterministic aspect ratio and upscale detection. Matches existing Tier 1/Tier 2 model.

---

## Color & Theme Consistency

| Option | Description | Selected |
|--------|-------------|----------|
| JS evaluate + visual combo (Recommended) | Read CSS variables and computed styles programmatically, trigger hover/focus states via playwright before reading — screenshots for holistic dark mode rendering | ✓ |
| Visual judgment only | Screenshot-based color comparison only — catches obvious issues but can't verify exact token values | |

**User's choice:** JS evaluate + visual combo
**Notes:** Hover/focus states require sequencing playwright actions before reading computed styles. Dark mode tested by toggling prefers-color-scheme or clicking theme toggle, then re-reading CSS variables.

---

## Claude's Discretion

- Exact JS evaluate snippets and DOM queries for each check
- Threshold values for aspect ratio distortion and upscale detection
- Methodology section structure in qa-tester.md
- Cross-page consistency sampling strategy

## Deferred Ideas

None — discussion stayed within phase scope
