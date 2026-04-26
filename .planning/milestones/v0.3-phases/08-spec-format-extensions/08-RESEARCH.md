# Phase 8: Spec Format Extensions - Research

**Researched:** 2026-04-19
**Domain:** Markdown spec format authoring + Claude Code agent/skill instruction editing
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Visual Focus Tiering**
- D-01: `## Visual Focus` mirrors the existing `## Accessibility Focus` pattern exactly — absent = no Tier 2 visual checks, present empty = full Tier 2, present with specific areas = selective Tier 2
- D-02: Four named areas, phase-aligned: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive` — maps 1:1 to Phase 9-12 requirement groups
- D-03: Tier 1 visual checks always run regardless of this section's presence. Visual Focus only controls Tier 2 opt-in (and which parts).

**Design Reference Format**
- D-04: `## Design Reference` uses keyed subsections per viewport (### desktop, ### mobile, etc.) with state paths as nested bullet items — mirrors `## Environments` structural pattern
- D-05: Path format: `- state-name: .qa/designs/filename.png` — relative paths from project root
- D-06: Missing viewport subsection = agent skips visual diff for that viewport and notes "no design reference for [viewport]" in report. Never blocks test run.
- D-07: Missing section entirely = no design comparison (backward compatible)

**Browser Engine Selection**
- D-08: `## Browsers` uses a simple bullet list of engine names — matches `## Viewports` pattern
- D-09: Valid values: `chromium`, `webkit`, `firefox` — maps directly to `playwright-cli --browser` flag
- D-10: Section absent = Chromium only (current behavior, full backward compatibility)
- D-11: Section present = run each listed engine; skill/agent maps each bullet to a `--browser=<engine>` flag

**Spec Documentation**
- D-12: SPEC-FORMAT.md follows the existing v2 documentation pattern: one Section Reference entry per new section, one example block each, one-line addendum to Backward Compatibility clause
- D-13: No combined "v3 features" example — sections are independently useful
- D-14: Agent (qa-tester.md) gets corresponding trigger/execution sections for each new spec section — same pattern as Accessibility Focus Trigger

### Claude's Discretion
- Example content within SPEC-FORMAT.md documentation (exact wording, example spec snippets)
- Ordering of new sections within SPEC-FORMAT.md (recommended placement among existing sections)
- Whether to add a sample spec file to `examples/` demonstrating the new sections

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| INFRA-01 | Visual tiering infrastructure — `## Visual Focus` spec section controls Tier 2 opt-in (parallels `## Accessibility Focus`) | Accessibility Focus Trigger pattern (qa-tester.md lines 719-731) is the exact template to mirror |
| INFRA-02 | `## Design Reference` spec format section for mockup image paths per viewport and state | `## Environments` keyed subsection pattern (SPEC-FORMAT.md lines 139-181) is the structural precedent |
| INFRA-03 | `## Browsers` spec format section for cross-browser engine selection | `## Viewports` bullet list pattern is the structural precedent; `playwright-cli --browser` is the execution hook |
</phase_requirements>

---

## Summary

Phase 8 is a pure documentation and instruction-editing phase. No new code is executed, no libraries are installed, no runtime behavior changes. The deliverables are edits to three existing files (`examples/SPEC-FORMAT.md`, `agents/qa-tester.md`, `skills/run/SKILL.md`) and optionally a new example spec file.

Every decision is locked by the CONTEXT.md discussion. The patterns to mirror are already in the codebase — the Accessibility Focus Trigger in `qa-tester.md` (lines 719-731) is the precise template for the Visual Focus Trigger. The `## Environments` section structure in `SPEC-FORMAT.md` (lines 139-181) is the precise template for Design Reference. The `## Viewports` bullet list is the precise template for Browsers.

The research confirms this phase has no external dependencies, no environment requirements, and no library choices to make. The planner needs to produce three focused editing tasks.

**Primary recommendation:** Plan as three sequential tasks — one per requirement (INFRA-01, INFRA-02, INFRA-03) — each updating SPEC-FORMAT.md, qa-tester.md, and skills/run/SKILL.md in turn.

---

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Spec parsing (Visual Focus, Design Reference, Browsers) | Agent (qa-tester.md) | Skill (run/SKILL.md) | The agent reads and interprets spec sections during test execution; the skill pre-processes and forwards spec content in the invocation prompt |
| Spec format documentation | Static docs (SPEC-FORMAT.md) | — | Pure reference document, no runtime component |
| Browser engine dispatch | Skill (run/SKILL.md) | Agent (qa-tester.md) | Skill maps bullet list to `--browser=<engine>` flags before spawning agent; agent is informed which engine is active |
| Design reference path resolution | Agent (qa-tester.md) | — | Agent resolves relative paths at runtime during visual comparison steps |
| Visual Focus depth toggling | Agent (qa-tester.md) | — | Agent reads section presence/content and decides Tier 2 scope, same as Accessibility Focus |

---

## Standard Stack

This phase involves no package installations. All work is Markdown editing.

**Tools in use (already present):**
| Tool | Version | Role |
|------|---------|------|
| `playwright-cli` | existing | Already handles `--browser` flag — no changes needed to playwright-cli itself |
| `agents/qa-tester.md` | current | Receives new trigger/execution sections |
| `skills/run/SKILL.md` | current | Agent invocation template receives new spec section forwarding |
| `examples/SPEC-FORMAT.md` | current | Receives three new Section Reference entries |

[VERIFIED: codebase grep] No npm installs, no external dependencies.

---

## Architecture Patterns

### System Architecture: New Section Data Flow

```
Spec file (.qa/*.md)
        |
        | contains (optionally)
        v
## Visual Focus      ## Design Reference      ## Browsers
  [area list]          ### desktop              - chromium
                         - state: path.png      - webkit
        |
        v
skills/run/SKILL.md  (parsing + forwarding)
        |
        | forwards parsed sections in invocation prompt
        v
agents/qa-tester.md  (trigger sections read section presence/content)
        |
        +-- Visual Focus Trigger --> controls Tier 2 visual scope
        +-- Design Reference handling --> resolves paths, compares
        +-- Browsers section handling --> loops per engine
```

### Pattern 1: Accessibility Focus Trigger (template to mirror verbatim)

This is the exact pattern to replicate for Visual Focus. From `agents/qa-tester.md` lines 719-731:

```markdown
### Accessibility Focus Trigger

The `## Accessibility Focus` section in a spec controls accessibility testing depth:

- **Section ABSENT:** Run only baseline Tier 1 checks ...
- **Section PRESENT with no items listed:** Run FULL Tier 2 ...
- **Section PRESENT with specific areas listed:** Run Tier 2 checks for only the listed areas. Valid area names:
  - `focus-management` — ...
  - `page-structure` — ...
  - `interactive-elements` — ...
  - `form-accessibility` — ...

**This is a depth toggle, not an on/off switch.** Baseline Tier 1 checks always run ...
```

[VERIFIED: codebase read, qa-tester.md lines 719-731]

### Pattern 2: Environments Keyed Subsection (template for Design Reference)

From `examples/SPEC-FORMAT.md` lines 139-181, the Environments section uses `### profile-name` subsections with key: value pairs under each. Design Reference mirrors this with `### desktop`, `### mobile`, etc., but with bullet items (`- state-name: path`) instead of YAML key-value pairs.

[VERIFIED: codebase read, SPEC-FORMAT.md lines 139-181]

### Pattern 3: Viewports Bullet List (template for Browsers)

The `## Viewports` section in SPEC-FORMAT.md is a plain bullet list of strings. `## Browsers` follows the identical pattern:

```markdown
## Browsers
- chromium
- webkit
- firefox
```

[VERIFIED: codebase read, SPEC-FORMAT.md lines 127-131]

### Recommended Section Ordering in SPEC-FORMAT.md

Based on logical grouping within the existing doc, the three new sections should be placed as follows (Claude's discretion area):

1. `## Visual Focus` — immediately after `## Accessibility Focus` (same tier-control pattern, complementary concern)
2. `## Design Reference` — after `## Visual Focus` (references visual assets needed for Visual Focus checks)
3. `## Browsers` — after `## Design Reference`, before `## Test Scenarios` (execution-scoping sections cluster together with Viewports and Environments)

This keeps all "depth/scope control" sections together and before the scenarios that they apply to.

### Anti-Patterns to Avoid

- **Combining v3 feature examples:** D-13 explicitly prohibits a combined block. Each section gets its own example.
- **Making any new section block test execution:** D-06 states missing viewport subsection in Design Reference never blocks the run. D-10 states absent Browsers section defaults to Chromium. None of the new sections are required.
- **Changing the Accessibility Focus Trigger:** The Visual Focus Trigger is NEW; the existing Accessibility Focus Trigger is unchanged. Do not edit it.
- **Adding `--browser` flag to the agent directly:** Browser dispatch is the skill's responsibility. The agent is informed which browser is active, but it does not invoke playwright-cli with `--browser` itself — the skill handles multi-engine looping.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Tier 2 visual methodology | Custom visual check logic in this phase | Leave as placeholder — downstream phases (9-12) implement it | Phase 8 is infrastructure only; methodology comes in Phases 9-12 |
| Browser flag parsing | Custom `--browser` flag parser | Bullet list mapped to `playwright-cli --browser=<engine>` | playwright-cli already supports `--browser` flag; the mapping is a one-liner in skill logic |
| Design image diffing | Pixel diff algorithm | Not in scope at all in this phase | VIS-01 (pixel-level diff) is explicitly deferred to Future Requirements |

**Key insight:** This phase creates the spec vocabulary (the interface). The implementations that respond to that vocabulary come in later phases. Don't implement Phase 9 methodology while building Phase 8 infrastructure.

---

## Common Pitfalls

### Pitfall 1: Conflating "Visual Focus Trigger" implementation with "Visual Focus methodology"

**What goes wrong:** The agent section describes what Tier 2 visual checks ARE (the methodology), but Phase 8 only installs the trigger — the presence/absence logic that says "should I do Tier 2 visual checks?" The actual Tier 2 checks are placeholders pointing to downstream phases.

**Why it happens:** The Accessibility Focus Trigger in qa-tester.md references the full Structured Accessibility section as the thing it unlocks. Visual Focus Trigger references methodology that doesn't exist yet.

**How to avoid:** Write the Visual Focus Trigger section in qa-tester.md as: "When `## Visual Focus` is present, perform visual verification. [Note: Tier 2 visual methodology will be added in Phases 9-12. For now, note in the report that Visual Focus was requested and flag it as pending methodology.]" This makes the infrastructure real and wires up the trigger, without hallucinating methodology.

**Warning signs:** If the task says "implement visual spacing checks" or "implement design diff logic," it has overstepped Phase 8 scope.

### Pitfall 2: Over-specifying Design Reference path handling

**What goes wrong:** Adding validation logic (does the file exist?), error handling for corrupt images, or fallback behavior beyond what D-06 specifies.

**Why it happens:** Natural tendency to make things robust.

**How to avoid:** D-06 defines the exact behavior: missing viewport subsection = skip + note in report. That's the complete spec. Implement only that.

**Warning signs:** If the agent section describes reading image files with the Read tool, checking file existence, or computing diffs — that's Phase 9 scope.

### Pitfall 3: Editing Accessibility Focus Trigger instead of adding Visual Focus Trigger

**What goes wrong:** Modifying lines 719-731 of qa-tester.md instead of adding a parallel NEW section.

**Why it happens:** The Visual Focus Trigger is a mirror of Accessibility Focus Trigger; it's tempting to consolidate.

**How to avoid:** Add a completely new `### Visual Focus Trigger` section. Do not touch the existing `### Accessibility Focus Trigger` section.

### Pitfall 4: Backward compat clause omission

**What goes wrong:** Updating SPEC-FORMAT.md without extending the Backward Compatibility clause (line 385) to mention the new sections.

**Why it happens:** The clause is at the bottom of the file, easy to miss.

**How to avoid:** D-12 explicitly calls this out. The backward compat clause must get a one-line addendum listing the three new sections as optional.

---

## Code Examples

### Visual Focus section (spec syntax)

```markdown
## Visual Focus
```
(present, empty = full Tier 2 visual)

```markdown
## Visual Focus
- design-verification
- layout-integrity
```
(present with specific areas = selective Tier 2)

### Design Reference section (spec syntax)

```markdown
## Design Reference

### desktop
- home-logged-out: .qa/designs/desktop-home-logged-out.png
- home-logged-in: .qa/designs/desktop-home-logged-in.png
- checkout-step-1: .qa/designs/desktop-checkout-1.png

### mobile
- home-logged-out: .qa/designs/mobile-home-logged-out.png
```

### Browsers section (spec syntax)

```markdown
## Browsers
- chromium
- webkit
```

### Visual Focus Trigger (qa-tester.md new section — template)

[VERIFIED: mirrors Accessibility Focus Trigger pattern, ASSUMED: exact placeholder wording for pending methodology]

```markdown
### Visual Focus Trigger

The `## Visual Focus` section in a spec controls visual verification depth:

- **Section ABSENT:** Run only baseline Tier 1 visual observations (visual anomalies noted during functional testing). This is the default.
- **Section PRESENT with no items listed:** Run FULL Tier 2 visual verification — all areas (design-verification, ux-states, layout-integrity, performance-responsive).
- **Section PRESENT with specific areas listed:** Run Tier 2 visual checks for only the listed areas. Valid area names:
  - `design-verification` — compare live page against design reference images (Phase 9)
  - `ux-states` — verify empty, loading, error, and interaction states (Phase 10)
  - `layout-integrity` — spacing, alignment, grid integrity, content clipping (Phase 11)
  - `performance-responsive` — Core Web Vitals, cross-browser rendering, viewport sweep (Phase 12)

**This is a depth toggle, not an on/off switch.** Baseline Tier 1 visual observations always run. The Visual Focus section only controls whether Tier 2 also runs (and which parts of it).

**Note:** Tier 2 visual methodology for each area will be added in Phases 9-12. If a Visual Focus section is present in a spec before those phases ship, note in the report: "Visual Focus requested for [areas] — full methodology pending Phase [N]."
```

### Design Reference handling (qa-tester.md new section — template)

```markdown
### Design Reference

When a `## Design Reference` section is present in the spec, use the provided image paths during visual verification:

- The section uses keyed subsections per viewport (`### desktop`, `### mobile`, etc.)
- Each subsection contains bullet items in the format: `- state-name: .qa/designs/filename.png`
- Paths are relative to the project root

**Behavior:**
- If a viewport subsection is present: use its images as reference during visual comparison for that viewport
- If a viewport subsection is absent: skip visual diff for that viewport, note "no design reference for [viewport]" in the report
- If the `## Design Reference` section is absent entirely: no design comparison runs (backward compatible)

The actual comparison methodology (how to compare live screenshots against reference images) is implemented in Phase 9.
```

### Browsers section handling (skill + agent)

In `skills/run/SKILL.md` — agent invocation template addition:

```markdown
{If spec has Browsers section:}
## Browser Engines
**Engines to test**: {comma-separated engine list from spec}
Run the full test once per engine. Report results labeled by engine: "[chromium] Scenario X: PASS"

{If spec has no Browsers section:}
## Browser Engines
**Engine**: chromium (default)
```

In `agents/qa-tester.md` — new spec v2 section:

```markdown
### Browsers

When a `## Browsers` section is present, the qa-run skill runs the test suite once per listed engine. The active engine is passed to you in the test assignment header.

**Your responsibility:** Include the active engine name in your report header and findings. Example: "[webkit] Login form — PASS". If you notice rendering differences between engines (reported across multiple runs), note them as cross-browser findings.

**Valid engines:** `chromium`, `webkit`, `firefox`

**Default (section absent):** Chromium only — same as current behavior.
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| v1 specs (no optional sections) | v2 specs (tags, depends_on, Data Sets, Environments, Accessibility Focus) | Phase 5 | All v1 specs still work unchanged |
| Global commands | Skills with frontmatter | Phase 1 (v0.2) | Skills are the only invocation pattern now |

**Deprecated/outdated:**
- Nothing in Phase 8 deprecates existing patterns. All changes are additive.

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | The Visual Focus Trigger placeholder wording ("methodology pending Phase N") is appropriate for the agent section | Code Examples | If wrong, the agent may silently skip or misreport when Visual Focus is present in a spec before Phases 9-12 ship |
| A2 | Browser looping (one run per engine) is handled by the skill, not by the agent natively spawning multiple browser sessions | Architecture Patterns | If wrong, the skill invocation template needs to spawn the agent N times rather than passing a single engine; planning tasks would need to address multi-invocation logic |

---

## Open Questions

1. **Should a sample spec be added to `examples/`?**
   - What we know: D-13 says no combined example block in SPEC-FORMAT.md, but says nothing about a standalone example file
   - What's unclear: Whether a separate `examples/visual-testing-spec.md` (or similar) helps users understand how the three sections work together
   - Recommendation: The planner should include this as a Wave 2 optional task (Claude's discretion). It adds low implementation cost and high user value. A sample spec showing all three new sections is more useful than three isolated documentation snippets.

2. **How does the skill loop engines in practice?**
   - What we know: D-11 says "run each listed engine" and "skill maps each bullet to a `--browser=<engine>` flag"
   - What's unclear: Whether `playwright-cli` supports `--browser` today, or if this is a speculative flag reference
   - Recommendation: The planner should include a validation step in the Browsers task — verify `playwright-cli --help` shows browser selection, or note that this remains a forward reference to be validated when Phase 12 (cross-browser testing methodology) ships. The spec format is the vocabulary; the execution is the methodology.

---

## Environment Availability

Step 2.6: SKIPPED — Phase 8 is purely documentation and text-editing changes with no external dependencies, CLI tool invocations, or runtime services required.

---

## Validation Architecture

No `workflow.nyquist_validation` key in `.planning/config.json` — validation is enabled.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected — this plugin has no automated test infrastructure |
| Config file | None |
| Quick run command | Manual review: read modified files and confirm sections are present |
| Full suite command | Manual: install plugin in a test project, run `/qa:run` with a spec containing each new section |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| INFRA-01 | `## Visual Focus` present triggers Tier 2 note in agent output | Manual smoke | Read `agents/qa-tester.md` and confirm Visual Focus Trigger section present | ❌ Wave 0 |
| INFRA-02 | `## Design Reference` present causes agent to note reference images per viewport | Manual smoke | Read `agents/qa-tester.md` and confirm Design Reference handling section present | ❌ Wave 0 |
| INFRA-03 | `## Browsers` section maps to engine list in skill invocation prompt | Manual smoke | Read `skills/run/SKILL.md` and confirm Browsers forwarding block present | ❌ Wave 0 |
| All | SPEC-FORMAT.md documents all three new sections with examples | File read | Read `examples/SPEC-FORMAT.md` and count new Section Reference entries | ❌ Wave 0 |
| All | Existing specs without new sections continue to work unchanged | Manual | Run existing `examples/sample-login-spec.md` — no errors expected | ❌ Wave 0 |

### Sampling Rate

- **Per task commit:** Read the modified file and confirm the new section is present and correctly formatted
- **Per wave merge:** Read all three modified files and verify backward compat clause is updated
- **Phase gate:** All three modified files reviewed; backward compat clause updated; sample-login-spec.md unchanged

### Wave 0 Gaps

- [ ] No automated test infrastructure exists for this plugin — all validation is manual file inspection and optional live testing
- [ ] No framework install needed — tests are document review steps, not code execution

---

## Security Domain

This phase makes no changes to authentication, session handling, input validation, cryptography, access control, or any runtime execution path. ASVS categories are not applicable.

`security_enforcement` is not explicitly set to `false` in config, but there is no security surface area in this phase — all changes are static Markdown documentation and AI instruction edits.

---

## Sources

### Primary (HIGH confidence)
- `agents/qa-tester.md` lines 719-731 — Accessibility Focus Trigger (exact pattern to mirror for Visual Focus)
- `examples/SPEC-FORMAT.md` lines 127-131 — Viewports section (pattern for Browsers)
- `examples/SPEC-FORMAT.md` lines 139-181 — Environments section (pattern for Design Reference)
- `examples/SPEC-FORMAT.md` line 385 — Backward Compatibility clause (to extend)
- `skills/run/SKILL.md` lines 99-191 — Agent Invocation template (receives new forwarding blocks)
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — All locked decisions (D-01 through D-14)

### Secondary (MEDIUM confidence)
- `LEARNINGS.md` — Three-tier model context, skill structure patterns

### Tertiary (LOW confidence)
- None

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new libraries; all patterns are verified from codebase reads
- Architecture: HIGH — all integration points identified and verified in source
- Pitfalls: HIGH — derived from explicit decision constraints and pattern analysis

**Research date:** 2026-04-19
**Valid until:** Stable indefinitely — pure documentation phase, no external dependencies that can drift
