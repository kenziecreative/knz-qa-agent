# Phase 14: Documentation Gap Closure - Research

**Researched:** 2026-04-21
**Domain:** Documentation authoring — Markdown documentation, plugin reference docs, spec format documentation
**Confidence:** HIGH

## Summary

Phase 14 is a pure documentation phase. All v0.3 visual capabilities shipped in Phases 8-12, but four documentation surfaces were not updated to reflect them. The milestone audit identified six specific gaps (MISSING-01 through MISSING-06) across ORCHESTRATION.md, SPEC-FORMAT.md, README.md, and CLAUDE.md.

The work is mechanical: read the agent (`agents/qa-tester.md`) to understand what shipped, then write clear documentation for it in the appropriate surface. No new capabilities are being added. No code changes. No agent behavior changes.

The highest-risk gap is ORCHESTRATION.md's missing Visual Testing section: orchestrating agents reading that document have no way to discover that visual testing capabilities exist or how to use them. This blocks the value of all Phase 9-12 work from being discoverable by AI agents.

**Primary recommendation:** Execute as three tasks — (1) update ORCHESTRATION.md with visual testing section and `--browsers` flag, (2) update SPEC-FORMAT.md with `breakpoint-sweep: continuous` and `placeholder_allowlist`, (3) update README.md v2 Features and CLAUDE.md Agent Architecture.

## Architectural Responsibility Map

This is a documentation-only phase. No code runs. No tiers own runtime behavior.

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| ORCHESTRATION.md visual section | docs/ layer | — | Static reference consumed by orchestrating agents and developers |
| SPEC-FORMAT.md opt-in fields | examples/ layer | — | Consumed by spec authors writing `.qa/*.md` files |
| README.md v2 Features | README | — | User-facing entry point; links to deeper reference docs |
| CLAUDE.md Agent Architecture | CLAUDE.md | — | Developer-facing project context for contributors |

## Gap Inventory (MISSING-01 through MISSING-06)

These six gaps were identified by the v0.3 milestone audit. Each is a documentation surface that exists but was not updated to cover capabilities shipped in Phases 8-12.

[VERIFIED: codebase grep + file reads]

### MISSING-01: ORCHESTRATION.md has no Visual Testing section

**File:** `docs/ORCHESTRATION.md`
**Gap:** The file covers all 7 skills comprehensively but has zero content about visual testing. A search for "Visual Focus", "Design Reference", "Browsers", "visual testing", or "Tier 2" returns no results.
**What shipped:** Phases 8-12 delivered full Tier 2 visual methodology: design verification, UX state verification, layout and content integrity, and performance/responsive checks — all controlled by `## Visual Focus` in specs.
**Impact:** Orchestrating agents that read ORCHESTRATION.md cannot discover visual testing capabilities. This is the most critical gap because it blocks discoverability for the primary audience (AI orchestrators).
**Fix:** Add a `## Visual Testing` section to ORCHESTRATION.md covering: the Visual Focus / Accessibility Focus parallel, the four Tier 2 areas, the `## Design Reference` spec section, and the `## Browsers` spec section with available engine values.

### MISSING-02: ORCHESTRATION.md /qa:run argument table missing `--browsers`

**File:** `docs/ORCHESTRATION.md`, line ~267 (the `/qa:run` arguments table)
**Gap:** The argument table lists `--project`, `--spec`, `--url`, `--tag`, `--env`, and `task` but NOT `--browsers`.
**What shipped:** Phase 8 delivered the `## Browsers` spec section. Phase 12 delivered cross-browser test execution. The `skills/run/SKILL.md` documents Mode 8: Run with specific browser engines and the `--browsers` flag.
**Impact:** Orchestrating agents cannot discover or use browser selection.
**Fix:** Add `--browsers` row to the `/qa:run` argument table in ORCHESTRATION.md.

**Exact row to add:** [VERIFIED: skills/run/SKILL.md Mode 8]
```
| `--browsers` | No | from spec | Use browser engines from spec's `## Browsers` section (chromium, webkit, firefox). Defaults to chromium if `## Browsers` absent. |
```

### MISSING-03: SPEC-FORMAT.md missing `breakpoint-sweep: continuous` opt-in field

**File:** `examples/SPEC-FORMAT.md`
**Gap:** The `### Performance & Responsive` / `breakpoint-sweep` behavior is fully documented in the agent (`agents/qa-tester.md` line 2024), but SPEC-FORMAT.md has no entry for the `breakpoint-sweep: continuous` opt-in field.
**What shipped:** Phase 12 delivered the two-phase breakpoint sweep. Phase 1 (discrete breakpoints at 320, 375, 768, 1024, 1280, 1440px) runs by default. Phase 2 (continuous sweep in ~50px increments) activates either when a Phase 1 width fails, OR when the spec opts in via `breakpoint-sweep: continuous` in the Visual Focus section.
**Impact:** Spec authors cannot discover or use continuous sweep opt-in because it is undocumented in the spec format reference.
**Fix:** Add a `breakpoint-sweep: continuous` entry within the `### Visual Focus (v2)` section of SPEC-FORMAT.md, or as a separate note under `### Performance & Responsive`. The field belongs as an additional opt-in within the Visual Focus section.

**Exact placement:** Add a sub-note to `### Visual Focus (v2)` in SPEC-FORMAT.md that documents the optional `breakpoint-sweep: continuous` modifier and shows a usage example.

### MISSING-04: SPEC-FORMAT.md missing `placeholder_allowlist` field

**File:** `examples/SPEC-FORMAT.md`
**Gap:** The `placeholder_allowlist` field is fully implemented in the agent (`agents/qa-tester.md` line 1783: `"If the spec declares placeholder_allowlist: [TODO, Learn more], suppress any finding where the matched snippet contains an allowlisted string"`), but SPEC-FORMAT.md has no documentation for this field.
**What shipped:** Phase 11 (Layout & Content Integrity) delivered the placeholder detection methodology with an allowlist mechanism to suppress known-false-positives.
**Impact:** Spec authors cannot suppress false-positive placeholder detections (e.g., a legitimate "Learn more" CTA) because the allowlist field is undocumented.
**Fix:** Add a `placeholder_allowlist` field entry to SPEC-FORMAT.md's `### Layout & Content Integrity` or `### Visual Focus (v2)` section. The field is a top-level spec field (not inside a section header), similar to `depends_on`.

**Exact usage (from agent source):**
```
placeholder_allowlist: [TODO, Learn more]
```
When present, any placeholder finding whose matched text contains a listed string is suppressed and noted as "Suppressed N placeholder finding(s) matching spec allowlist."

### MISSING-05: README.md v2 Features section missing Visual Focus, Design Reference, and Browsers

**File:** `README.md`
**Gap:** The `### v2 Features` section lists Tags, Dependencies, Data-Driven Scenarios, Environment Profiles, and Accessibility Focus — but does NOT list Visual Focus, Design Reference, or Browsers. These three features shipped in Phase 8.
**What shipped:** All three spec sections are documented in SPEC-FORMAT.md and implemented in the agent. README references SPEC-FORMAT.md for full documentation but should list the features at the overview level.
**Impact:** Users reading README.md v2 Features section get an incomplete picture of what v2 supports. They cannot discover visual testing spec features from the README entry point.
**Fix:** Add three new sub-entries under `### v2 Features` (parallel to "Accessibility Focus"): Visual Focus, Design Reference, and Browsers — each with a brief description and example.

### MISSING-06: CLAUDE.md Agent Architecture section not updated for v0.3 methodology

**File:** `CLAUDE.md`
**Gap:** The `### Agent Architecture` section bullet list reads:
- Testing methodology (screenshot/console/network discipline)
- Pattern recognition and 3-failure escalation rules
- Multi-viewport testing (mobile/tablet/desktop)
- Multi-persona testing (role-based auth flows)
- Form intelligence (validation, submission states, autofill)
- SPA awareness (routing, history, hydration)
- Structured accessibility testing (Tier 1 baseline + Tier 2 deep checks)
- Network intelligence (timing, error correlation, mixed content)
- Spec v2 features (dependencies, data-driven scenarios, tags, environments)

The list is missing v0.3 additions: Visual & UX design verification (Tier 2 visual checks), cross-browser engine testing, and the Visual Focus spec trigger.

**Fix:** Append to the bullet list:
- Visual & UX design verification (Tier 2 — design reference comparison, UX states, layout integrity, performance/responsive, triggered by `## Visual Focus` in specs)
- Cross-browser engine testing (Chromium, WebKit, Firefox via `## Browsers` in specs)

## Standard Stack

This is a documentation phase. There is no runtime technology stack. All work is Markdown file editing.

| Document | Format | Audience | Edit Type |
|----------|--------|----------|-----------|
| `docs/ORCHESTRATION.md` | Markdown | AI orchestrators + developers | Add section + extend table row |
| `examples/SPEC-FORMAT.md` | Markdown | Spec authors | Add field documentation within existing sections |
| `README.md` | Markdown | Users | Add entries to v2 Features section |
| `CLAUDE.md` | Markdown | Contributors | Append to bullet list |

## Architecture Patterns

### How ORCHESTRATION.md is structured

[VERIFIED: file read]

The document follows this order: Quick Reference table → Interpreting Results → Workflow Integration → Skill Reference (per-skill) → Getting Started → Data Architecture → Hooks → Cross-Session Capability.

The Visual Testing section should be a new top-level section, inserted AFTER `## Skill Reference` and BEFORE `## Getting Started`. This placement puts it where orchestrating agents will encounter it after learning what skills exist and before the developer walkthrough.

### How SPEC-FORMAT.md is structured

[VERIFIED: file read]

The document follows this order: Basic Structure example → Minimal Spec Example → Section Reference (one subsection per spec section). New field documentation belongs inside the relevant existing `### Visual Focus (v2)` subsection for `breakpoint-sweep: continuous`, and as a new top-level field description for `placeholder_allowlist`.

The `placeholder_allowlist` field is a top-level spec field (sits at spec root, not inside any section). It should be documented in the Section Reference as its own entry, parallel to how `depends_on:` is documented for scenarios. Placement: after the `### Browsers (v2)` section and before `### Test Scenarios`.

### How README.md v2 Features is structured

[VERIFIED: file read]

The `### v2 Features` section uses the pattern: one `####` subsection per feature, each with a description paragraph, a code example, and a one-liner of behavior. The three new entries (Visual Focus, Design Reference, Browsers) should each follow this pattern.

### Pattern: Parallel between Visual Focus and Accessibility Focus

[VERIFIED: codebase comparison]

The existing Accessibility Focus documentation in README.md is the exact model to follow for Visual Focus:
- "Add an `## Accessibility Focus` section to activate Tier 2 structured accessibility testing"
- Four named areas: `focus-management`, `page-structure`, `interactive-elements`, `form-accessibility`
- "Without this section, Tier 1 baseline checks always run... With this section, the agent runs the full Tier 2 structured methodology"

Visual Focus follows identical pattern but with four different areas: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive`.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Version references | Don't invent spec section names | Read agents/qa-tester.md as ground truth | The agent is the implementation; docs must match it |
| New visual area names | Don't create new area names | Use the four names from Phase 8: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive` | Names are hard-coded in the agent trigger logic |
| New argument syntax | Don't invent argument behavior | Read skills/run/SKILL.md for authoritative argument behavior | Planner task descriptions must be accurate |

## Common Pitfalls

### Pitfall 1: Adding content to SPEC-FORMAT.md that contradicts the agent

**What goes wrong:** Documenting spec field behavior that doesn't match how `agents/qa-tester.md` actually handles it.
**Why it happens:** The agent was updated in Phases 9-12 with specific behavior decisions (e.g., `breakpoint-sweep: continuous` exact syntax, allowlist exact format). Documenting from memory rather than reading the agent produces drift.
**How to avoid:** Before writing any SPEC-FORMAT.md entry, grep the agent for the exact behavior string and use it verbatim.
**Warning signs:** If documentation says something the agent doesn't corroborate.

### Pitfall 2: Creating a new top-level section in ORCHESTRATION.md with wrong placement

**What goes wrong:** Inserting the Visual Testing section in the wrong position (e.g., at the very end, or inside Skill Reference).
**Why it happens:** The document is long (600+ lines). Easy to insert at the wrong level or location.
**How to avoid:** The Visual Testing section goes AFTER the last skill reference (`### /qa:guide`) and BEFORE `## Getting Started`. It is a top-level `##` section.

### Pitfall 3: README.md listing Visual Focus but not documenting `breakpoint-sweep: continuous` or `placeholder_allowlist`

**What goes wrong:** README mentions Visual Focus without covering all its opt-in sub-features. Users still can't discover `breakpoint-sweep: continuous` or `placeholder_allowlist`.
**How to avoid:** Per success criteria, README.md only needs to list Visual Focus, Design Reference, and Browsers. The two opt-in fields are SPEC-FORMAT.md gaps only — keep them there. README points to SPEC-FORMAT.md for complete format documentation.

### Pitfall 4: Updating CLAUDE.md Agent Architecture without reading the current file first

**What goes wrong:** Rewriting the section instead of appending to it, or adding bullets in the wrong style.
**How to avoid:** Read CLAUDE.md in full before editing. The existing bullet list uses a specific format — match it exactly. Only append; do not change existing bullets.

## Code Examples

### Exact field syntax verified from agents/qa-tester.md

[VERIFIED: grep of agents/qa-tester.md lines 1783, 2024]

**`placeholder_allowlist` field (top-level spec field):**
```markdown
placeholder_allowlist: [TODO, Learn more]
```
When present, placeholder findings whose matched text contains a listed string are suppressed. The report notes: "Suppressed N placeholder finding(s) matching spec allowlist."

**`breakpoint-sweep: continuous` opt-in (within `## Visual Focus` section):**
```markdown
## Visual Focus
- performance-responsive
breakpoint-sweep: continuous
```
When `breakpoint-sweep: continuous` is present, the agent runs a continuous sweep in ~50px increments across the full width range (not just the 6 default discrete breakpoints). This supplements (does not replace) Phase 1 discrete testing.

### Visual Focus section reference (from examples/SPEC-FORMAT.md)

[VERIFIED: file read]

The four valid area names:
- `design-verification` — compare live page against design reference images (Phase 9)
- `ux-states` — verify empty, loading, error, and interaction states (Phase 10)
- `layout-integrity` — spacing, alignment, grid integrity, content clipping (Phase 11)
- `performance-responsive` — Core Web Vitals, cross-browser rendering, viewport sweep (Phase 12)

### `--browsers` argument row for ORCHESTRATION.md /qa:run table

[VERIFIED: skills/run/SKILL.md Mode 8 and execution flow step 10]

```
| `--browsers` | No | from spec | Use browser engines from spec's `## Browsers` section (chromium, webkit, firefox). Defaults to chromium if `## Browsers` absent. |
```

### ORCHESTRATION.md Visual Testing section outline

[VERIFIED: cross-referenced docs/ORCHESTRATION.md + agents/qa-tester.md sections]

The section should cover:
1. **Overview** — What visual testing is; parallel to accessibility (Tier 1 always runs, Tier 2 opt-in)
2. **Activating Visual Testing** — Add `## Visual Focus` to spec; list of four areas
3. **Design Reference** — How to provide mockup paths per viewport; the `## Design Reference` format
4. **Cross-Browser Testing** — How `## Browsers` section works; valid engine values; what the agent reports per engine
5. **Tier 2 Coverage** — Brief one-liner per area describing what each checks

## State of the Art

This is entirely internal to the QA plugin — no external ecosystem changes apply.

| Document State Before Phase 14 | State After Phase 14 |
|--------------------------------|---------------------|
| ORCHESTRATION.md: no visual testing content | ORCHESTRATION.md: visual testing section + `--browsers` in argument table |
| SPEC-FORMAT.md: missing two field docs | SPEC-FORMAT.md: `breakpoint-sweep: continuous` and `placeholder_allowlist` documented |
| README.md: 5 v2 features listed (missing 3) | README.md: 8 v2 features listed (Visual Focus, Design Reference, Browsers added) |
| CLAUDE.md: v0.2-era Agent Architecture bullets | CLAUDE.md: v0.3 bullets added (visual verification, cross-browser) |

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `--browsers` is the correct flag name (vs `--browser`) | Gap Inventory MISSING-02 | Low — skills/run/SKILL.md Mode 8 shows `--browsers` (plural) explicitly |

**All other claims are verified against source files via grep or file reads.**

## Open Questions

None. All gaps are precisely identified with exact file locations and verified content from source. Phase is fully scoped.

## Environment Availability

Step 2.6: SKIPPED (no external dependencies — documentation-only phase, no tools, CLIs, or services required beyond file editing).

## Validation Architecture

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Verification Approach |
|--------|----------|-----------|----------------------|
| INFRA-01 | Visual Focus spec section documented in SPEC-FORMAT.md | Manual/grep | `grep "### Visual Focus (v2)" examples/SPEC-FORMAT.md` already passes (shipped Phase 8); `breakpoint-sweep: continuous` addition can be verified with grep |
| INFRA-02 | Design Reference spec section documented in SPEC-FORMAT.md | Manual/grep | Already passes (shipped Phase 8); only `placeholder_allowlist` addition is new |
| INFRA-03 | Browsers spec section documented in SPEC-FORMAT.md | Manual/grep | Already passes (shipped Phase 8); only ORCHESTRATION.md argument table row is new |
| INFRA-04 | ORCHESTRATION.md covers visual testing capabilities | Manual/grep | `grep "Visual Focus" docs/ORCHESTRATION.md` — currently FAILS, must pass after phase |
| INFRA-05 | /qa:guide skill exists (Phase 13 complete) | N/A | Already done; Phase 14 updates docs around it |

**Note:** INFRA-01, INFRA-02, INFRA-03 were partially completed in Phase 8 (the spec format sections themselves). Phase 14 completes the remaining documentation gaps for those requirements (SPEC-FORMAT.md field additions, ORCHESTRATION.md coverage, README.md listing).

### Wave 0 Gaps

None — this phase is documentation editing only. No test framework or test file setup required.

## Security Domain

Phase 14 makes no changes to runtime code, agent behavior, skill execution, or browser automation. Zero attack surface changes. Security analysis: N/A.

## Sources

### Primary (HIGH confidence)
- `agents/qa-tester.md` — Ground truth for all agent behavior; verified exact syntax for `breakpoint-sweep: continuous` (line 2024) and `placeholder_allowlist` (line 1783)
- `docs/ORCHESTRATION.md` — Verified current state; confirmed no visual testing content
- `examples/SPEC-FORMAT.md` — Verified current state; confirmed `breakpoint-sweep` and `placeholder_allowlist` absent
- `README.md` — Verified current v2 Features section lists 5 features, missing 3
- `CLAUDE.md` — Verified current Agent Architecture section; confirmed v0.3 capabilities absent
- `skills/run/SKILL.md` — Verified `--browsers` flag syntax (Mode 8, execution flow step 10)
- `.planning/phases/08-spec-format-extensions/08-01-PLAN.md` — Verified Phase 8 scope (Visual Focus, Design Reference, Browsers spec sections)
- `.planning/phases/13-usage-guidance/13-01-SUMMARY.md`, `13-02-SUMMARY.md` — Verified Phase 13 delivered ORCHESTRATION.md + /qa:guide

### Secondary (MEDIUM confidence)
- `.planning/phases/13-usage-guidance/13-REVIEW.md` — Identified WR-01/WR-02 warnings in /qa:guide skill; these are pre-existing issues, not Phase 14 scope unless explicitly required by success criteria

## Metadata

**Confidence breakdown:**
- Gap inventory: HIGH — all gaps verified by direct file reads and grep
- Fix approach: HIGH — exact content to add is traceable to source files
- Scope boundaries: HIGH — success criteria is specific and checkable

**Research date:** 2026-04-21
**Valid until:** Stable indefinitely (documentation of existing shipped capabilities)
