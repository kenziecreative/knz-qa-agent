# Phase 9: Design Verification - Context

**Gathered:** 2026-04-19
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase implements the actual design verification methodology in `qa-tester.md` — the Tier 2 visual checks triggered by `## Visual Focus: design-verification` in a spec. It covers five requirement areas: design reference comparison (DESIGN-01), typography (DESIGN-02), images/media (DESIGN-03), color/theme (DESIGN-04), and cross-page consistency (DESIGN-05). Phase 8 already created the spec format infrastructure; this phase fills in the methodology the agent follows when those sections are present.

</domain>

<decisions>
## Implementation Decisions

### Mockup Comparison (DESIGN-01)
- **D-01:** Structured category checklist — the agent scans reference image vs live screenshot using fixed categories in order: typography, spacing, color, imagery, layout. Not a holistic "look and report whatever you notice" approach.
- **D-02:** Each category produces findings with confidence level (High/Medium/Low) and "live page shows X vs reference shows Y" format — consistent with the existing confidence framework in qa-tester.md.
- **D-03:** Root cause diagnosis is narrative-driven (same as existing agent findings) — the agent describes what's different and suggests why (layout shift, wrong color token, missing element, etc.).

### Typography Verification (DESIGN-02)
- **D-04:** Dual approach — `evaluate()` JS probes for verifiable assertions (font loaded via `document.fonts.check()`, text overflow via `scrollWidth > clientWidth`, computed font family) PLUS visual judgment from screenshots for cross-page consistency and readability.
- **D-05:** When a spec or design tokens file declares expected font families/sizes, the agent uses those as the baseline for comparison. When no expected values exist, the agent infers consistency by comparing computed styles across pages.
- **D-06:** Font load detection uses `document.fonts.check('16px FontName')` and `document.fonts.ready` — catches FOUT/FOIT and fallback font rendering.

### Image & Media Quality (DESIGN-03)
- **D-07:** Evaluate-first, screenshot fallback — JS probes (`naturalWidth`/`naturalHeight` vs rendered dimensions, `complete` property, `currentSrc`, lazy-load `src`/`data-src` attributes) always run as Tier 1. Screenshot-based visual judgment supplements when anomalies found or Tier 2 opted in.
- **D-08:** Aspect ratio distortion detected by comparing `naturalWidth/naturalHeight` ratio against rendered `width/height` ratio — threshold-based (e.g., >5% deviation = flag).
- **D-09:** Upscale detection: `naturalWidth < renderedWidth` or `naturalHeight < renderedHeight` = image displayed larger than its source resolution.

### Color & Theme Consistency (DESIGN-04)
- **D-10:** Dual approach — `evaluate()` reads CSS custom properties and computed styles for programmatic token consistency. Visual judgment from screenshots covers holistic rendering (especially after dark mode toggle).
- **D-11:** Hover/focus color states verified by sequencing playwright actions (hover, focus) THEN reading computed styles via evaluate — computed styles only reflect the current interaction state.
- **D-12:** Dark mode testing: agent toggles color scheme (via `evaluate` to set `prefers-color-scheme` media override or clicking a theme toggle if present) then re-reads CSS variables and takes comparison screenshots.

### Cross-Page Consistency (DESIGN-05)
- **D-13:** When no formal design system/tokens exist, the agent infers consistency by sampling computed styles from key elements across multiple pages (headers, buttons, links, body text) and flagging deviations.
- **D-14:** When design tokens (CSS custom properties) exist, the agent reads them and verifies consistent usage across pages.

### Overarching Pattern
- **D-15:** All five areas follow the same dual pattern: `evaluate()` for deterministic/measurable checks + visual judgment for holistic/perceptual quality. This mirrors the existing Tier 1 (baseline) / Tier 2 (deep) model established in qa-tester.md.

### Claude's Discretion
- Exact JS evaluate snippets and DOM queries for each check
- Threshold values for aspect ratio distortion and upscale detection
- How to structure the methodology sections in qa-tester.md (section naming, ordering within existing structure)
- Whether to add helper evaluate scripts or inline them in methodology descriptions
- Cross-page consistency sampling strategy (which elements, how many pages)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Spec Format Infrastructure (Phase 8 output)
- `agents/qa-tester.md` — Lines 733-773: Visual Focus Trigger, Design Reference, and Browsers sections (Phase 8 stubs to be filled with methodology)
- `agents/qa-tester.md` — Lines 719-731: Accessibility Focus Trigger (pattern to mirror for methodology sections)
- `examples/SPEC-FORMAT.md` — Visual Focus, Design Reference, and Browsers section references
- `skills/run/SKILL.md` — Lines 142-149: Visual Focus and Design Reference forwarding blocks in agent invocation template

### Existing Patterns
- `agents/qa-tester.md` — Lines 95-110: Testing Methodology (screenshot/console/network discipline — the five-step protocol)
- `agents/qa-tester.md` — Lines 240-310: Structured Accessibility Tier 2 checks (pattern to mirror — numbered procedure, explicit confidence levels, common patterns to flag)
- `agents/qa-tester.md` — Lines 355-375: Confidence Summary Table (existing confidence framework to extend)

### Requirements
- `.planning/REQUIREMENTS.md` §DESIGN-01 — Design reference comparison with root cause diagnosis
- `.planning/REQUIREMENTS.md` §DESIGN-02 — Typography: font loading, consistency, overflow, readability
- `.planning/REQUIREMENTS.md` §DESIGN-03 — Images: broken, distortion, lazy-load, upscaled
- `.planning/REQUIREMENTS.md` §DESIGN-04 — Color: token usage, hover/focus states, dark mode
- `.planning/REQUIREMENTS.md` §DESIGN-05 — Cross-page consistency (inferred or design system)

### Phase 8 Context
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — Visual Focus tiering decisions (D-01 through D-03), Design Reference format (D-04 through D-07)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **playwright-cli screenshot** — already available for capturing visual state
- **playwright-cli evaluate** — run arbitrary JS in page context for computed styles, DOM queries, font checks
- **playwright-cli hover** — trigger hover state before reading computed styles
- **playwright-cli snapshot** — accessibility tree for element identification
- **Confidence framework** — High/Medium/Low with rationale, already established throughout qa-tester.md

### Established Patterns
- **Accessibility Focus Trigger** (qa-tester.md lines 719-731) — depth toggle pattern: absent/present/present-with-areas controlling methodology depth
- **Structured Accessibility Tier 2** (qa-tester.md lines 240-310) — methodology template: numbered procedure + confidence level + common patterns to flag
- **Visual Focus Trigger** (qa-tester.md lines 733-747) — already exists as stub, references Phase 9 for methodology
- **Design Reference** (qa-tester.md lines 749-763) — already exists as stub, "actual comparison methodology implemented in Phase 9"

### Integration Points
- `agents/qa-tester.md` — New methodology sections go here (design verification procedures, confidence levels, common patterns)
- `agents/qa-tester.md` line 747 — Remove "Tier 2 visual methodology pending Phase N" note after methodology is implemented
- `agents/qa-tester.md` line 763 — Remove "actual comparison methodology is implemented in Phase 9" note after methodology is implemented
- `agents/qa-tester.md` Confidence Summary Table — Extend with new design verification confidence entries

</code_context>

<specifics>
## Specific Ideas

### User Session Transcript Context (from Phase 8)
The user shared a session transcript where the QA agent failed to catch visual spacing issues — agent checked for content clipping but missed that a button was flush against the bottom edge of its container (no bottom padding). This is exactly the kind of gap the structured category checklist approach (D-01) should catch — "spacing" is an explicit scan category that checks balanced spacing on all sides, not just "is content cut off?"

### Evaluate + Visual Dual Pattern
All four discussed areas converged on the same approach: JS evaluate for deterministic/measurable checks, visual judgment for holistic/perceptual quality. This maps cleanly to the existing Tier 1/Tier 2 infrastructure — evaluate probes can serve as Tier 1 baseline checks that always run, while the visual comparison methodology activates as Tier 2 when design-verification is opted into via Visual Focus.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 09-design-verification*
*Context gathered: 2026-04-19*
