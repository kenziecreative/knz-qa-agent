# Phase 8: Spec Format Extensions - Context

**Gathered:** 2026-04-19
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase extends the QA spec markdown format with three new optional sections: `## Visual Focus`, `## Design Reference`, and `## Browsers`. These sections provide infrastructure for all downstream v0.3 visual/UX methodology (Phases 9-13) to be spec-driven. No new testing methodology is implemented here — only the spec format and its documentation.

</domain>

<decisions>
## Implementation Decisions

### Visual Focus Tiering
- **D-01:** `## Visual Focus` mirrors the existing `## Accessibility Focus` pattern exactly — absent = no Tier 2 visual checks, present empty = full Tier 2, present with specific areas = selective Tier 2
- **D-02:** Four named areas, phase-aligned: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive` — maps 1:1 to Phase 9-12 requirement groups
- **D-03:** Tier 1 visual checks (whatever baseline the agent does by default) always run regardless of this section's presence. The Visual Focus section only controls whether Tier 2 also runs (and which parts of it).

### Design Reference Format
- **D-04:** `## Design Reference` uses keyed subsections per viewport (### desktop, ### mobile, etc.) with state paths as nested bullet items — mirrors `## Environments` structural pattern
- **D-05:** Path format: `- state-name: .qa/designs/filename.png` — relative paths from project root
- **D-06:** Missing viewport subsection = agent skips visual diff for that viewport and notes "no design reference for [viewport]" in the report. Never blocks the test run.
- **D-07:** Missing section entirely = no design comparison at all (backward compatible)

### Browser Engine Selection
- **D-08:** `## Browsers` uses a simple bullet list of engine names — matches `## Viewports` pattern
- **D-09:** Valid values: `chromium`, `webkit`, `firefox` — maps directly to `playwright-cli --browser` flag
- **D-10:** Section absent = Chromium only (current behavior, full backward compatibility)
- **D-11:** Section present = run each listed engine. The skill/agent maps each bullet to a `--browser=<engine>` flag

### Spec Documentation
- **D-12:** SPEC-FORMAT.md follows the existing v2 documentation pattern: one Section Reference entry per new section, one example block each, one-line addendum to Backward Compatibility clause
- **D-13:** No combined "v3 features" example — sections are independently useful and a combined block creates maintenance drift risk
- **D-14:** Agent (qa-tester.md) gets corresponding trigger/execution sections for each new spec section — same pattern as Accessibility Focus Trigger

### Claude's Discretion
- Example content within SPEC-FORMAT.md documentation (exact wording, example spec snippets)
- Ordering of new sections within SPEC-FORMAT.md (recommended placement among existing sections)
- Whether to add a sample spec file to examples/ demonstrating the new sections

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Existing Spec Format
- `examples/SPEC-FORMAT.md` — Current spec format reference doc; all three new sections are added here
- `examples/sample-login-spec.md` — Example spec showing v2 features in practice

### Agent & Skill Files
- `agents/qa-tester.md` — Agent instructions; contains "Accessibility Focus Trigger" section (pattern to mirror for Visual Focus) and all existing spec v2 feature handling
- `skills/run/SKILL.md` — Agent invocation template; contains accessibility instructions in the prompt that need parallel visual focus instructions

### Requirements
- `.planning/REQUIREMENTS.md` §INFRA-01 — Visual tiering infrastructure
- `.planning/REQUIREMENTS.md` §INFRA-02 — Design Reference spec format section
- `.planning/REQUIREMENTS.md` §INFRA-03 — Browsers spec format section

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Patterns
- **Accessibility Focus Trigger** (qa-tester.md lines 719-731): Exact tiering pattern to mirror — section absent/present/present-with-areas controlling depth
- **Environments section** (SPEC-FORMAT.md lines 139-181): Keyed subsection pattern with named profiles — structural precedent for Design Reference viewport subsections
- **Viewports section** (SPEC-FORMAT.md lines 127-131): Simple bullet list pattern — structural precedent for Browsers section
- **Spec v2 backward compat** (SPEC-FORMAT.md lines 383-386): Existing backward compatibility clause to extend

### Integration Points
- `skills/run/SKILL.md` — Agent invocation template needs new sections in the prompt for Visual Focus, Design Reference, and Browsers
- `agents/qa-tester.md` — Needs new "Visual Focus Trigger" section and "Design Reference" section and "Browsers" section in Spec Format v2 Features
- `examples/SPEC-FORMAT.md` — Three new Section Reference entries + examples + backward compat update

</code_context>

<specifics>
## Specific Ideas

### User Transcript Context (Captured During Discussion)
The user shared a transcript from a live project session where the QA agent failed to catch visual spacing issues:
- Agent checked for content clipping but missed that a button was flush against the bottom edge of its container (no bottom padding)
- Took 3 rounds of user feedback before the agent noticed the spacing imbalance
- Root cause: agent evaluates "is content cut off?" but not "is spacing balanced on all sides?"
- This is exactly the kind of gap that `layout-integrity` and `ux-states` visual checks (Phases 10-11) should catch when opted in via `## Visual Focus`
- The spec format infrastructure in this phase enables downstream methodology to address this gap spec-driven

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 08-spec-format-extensions*
*Context gathered: 2026-04-19*
