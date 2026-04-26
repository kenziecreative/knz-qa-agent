# Phase 14: Documentation Gap Closure - Pattern Map

**Mapped:** 2026-04-21
**Files analyzed:** 4 (modified only — no new files)
**Analogs found:** 4 / 4 (each file is its own best analog — documentation edits, not new file creation)

---

## File Classification

| Modified File | Role | Data Flow | Closest Analog | Match Quality |
|---------------|------|-----------|----------------|---------------|
| `docs/ORCHESTRATION.md` | documentation | reference | `docs/ORCHESTRATION.md` `### /qa:guide` section (lines 436–481) for new top-level section; lines 260–268 for argument table row | exact (same document, same pattern) |
| `examples/SPEC-FORMAT.md` | documentation | reference | `examples/SPEC-FORMAT.md` `### Accessibility Focus (v2)` section (lines 195–224) for `breakpoint-sweep`; `### Scenario Dependencies (v2)` section (lines 360–397) for `placeholder_allowlist` | exact (same document, parallel sections) |
| `README.md` | documentation | reference | `README.md` `#### Accessibility Focus` section (lines 283–298) | exact (same document, parallel feature entry) |
| `CLAUDE.md` | documentation | reference | `CLAUDE.md` `### Agent Architecture` bullet list (lines 48–58) | exact (same document, append to existing list) |

---

## Pattern Assignments

### `docs/ORCHESTRATION.md` — Two changes

#### Change A: Add `--browsers` row to `/qa:run` argument table

**Location:** Lines 260–268 (the `/qa:run` `#### Arguments` table)

**Existing table pattern** (lines 260–268 of `docs/ORCHESTRATION.md`):
```markdown
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | No* | — | Path to project containing `.qa/` specs |
| `--spec <name>` | No | all specs | Specific spec name to run (no `.md` extension) |
| `--url <base-url>` | No* | from spec | Base URL to test against (overrides spec's Base URL) |
| `--tag <tag>` | No | all scenarios | Filter to scenarios matching this tag. Comma-separated for OR logic: `--tag smoke,critical` |
| `--env <profile>` | No | first profile | Environment profile name from spec's `## Environments` section |
| `task` (quoted) | No* | — | Ad-hoc test description when not using a spec file |
```

**Exact row to insert** (verified from `skills/run/SKILL.md` Mode 8 and execution flow step 10):
```markdown
| `--browsers` | No | from spec | Use browser engines from spec's `## Browsers` section (chromium, webkit, firefox). Defaults to chromium if `## Browsers` absent. |
```

**Insert after** the `--env` row, before the `task` row. The footnote line `*Either --project or...` stays at the bottom unchanged.

---

#### Change B: Add `## Visual Testing` top-level section

**Location:** Between line 481 (end of `### /qa:guide` section, the `---` separator) and line 484 (`## Getting Started (For Developers)`)

**Structural pattern to copy** — each skill section in ORCHESTRATION.md follows this template (example: `### /qa:guide`, lines 436–481):
```markdown
### /qa:guide

[one-line description]. [spawns agent / does not spawn agent statement.]

#### Arguments

| Argument | Required | Default | Description |
...

#### Usage
...

#### What It [Reads/Produces]
...

#### When to Use
...

---
```

**For the new Visual Testing section**, use a `##` level (not `###`) because it is a cross-cutting capability section, not a per-skill reference. The existing cross-cutting sections at this level are: `## Interpreting Results` (line 27), `## Workflow Integration` (line 123), `## Getting Started` (line 484), `## Data Architecture` (line 550), `## Hooks` (line 571), `## Cross-Session Capability` (line 591).

**Subsection heading pattern** — use `###` for named areas within the `##` section, consistent with how `## Interpreting Results` uses `### Result Format`, `### Confidence Levels`, `### Parsing Result Data`.

**Parallel to draw from** — `## Interpreting Results` section (lines 27–119) for document flow: overview → mechanics → decision table. The Visual Testing section should follow: overview → activation → four areas → design reference → browsers.

**Exact section outline** (content sourced from `agents/qa-tester.md` and `examples/SPEC-FORMAT.md`):
```markdown
## Visual Testing

Visual testing is Tier 2 opt-in functionality. Tier 1 visual observations (noting anomalies during functional testing) always run. Tier 2 structured visual checks activate only when a spec includes a `## Visual Focus` section — parallel to how `## Accessibility Focus` activates Tier 2 accessibility checks.

### Activating Visual Testing

Add a `## Visual Focus` section to the spec:

```markdown
## Visual Focus
- design-verification
- layout-integrity
```

Available areas:

| Area | What It Checks |
|------|---------------|
| `design-verification` | Compare live page against design reference images |
| `ux-states` | Verify empty, loading, error, and interaction states |
| `layout-integrity` | Spacing, alignment, grid integrity, content clipping |
| `performance-responsive` | Core Web Vitals, cross-browser rendering, viewport sweep |

- Present with no items listed: All four areas run (full Tier 2)
- Present with specific areas listed: Only those areas run
- Absent: Tier 1 only (visual anomalies noted during functional testing)

### Design Reference

Provide mockup images for visual comparison during `design-verification`. Add a `## Design Reference` section to the spec:

```markdown
## Design Reference

### desktop
- home-logged-out: .qa/designs/desktop-home.png

### mobile
- home-logged-out: .qa/designs/mobile-home.png
```

If a viewport subsection is absent, the agent skips visual diff for that viewport and notes "no design reference for [viewport]" in the report.

### Cross-Browser Testing

Specify which browser engines to run by adding a `## Browsers` section to the spec:

```markdown
## Browsers
- chromium
- webkit
- firefox
```

Valid engines: `chromium` (Chrome/Edge), `webkit` (Safari), `firefox` (Gecko). When the section is absent, only Chromium runs (backward compatible).

Invoke via: `/qa:run --project <path> --browsers`

The agent reports findings per engine: `[webkit] Scenario X: FAIL — rendering divergence on nav menu`

### Tier 2 Coverage Summary

| Area | Runs When |
|------|-----------|
| Design verification | `design-verification` in `## Visual Focus` AND `## Design Reference` section present |
| UX state verification | `ux-states` in `## Visual Focus` |
| Layout & content integrity | `layout-integrity` in `## Visual Focus` |
| Performance & responsive | `performance-responsive` in `## Visual Focus` |
| Breakpoint sweep (continuous) | `performance-responsive` active AND `breakpoint-sweep: continuous` in `## Visual Focus`, OR any Phase 1 breakpoint fails |
```

---

### `examples/SPEC-FORMAT.md` — Two changes

#### Change A: Add `breakpoint-sweep: continuous` to `### Visual Focus (v2)`

**Location:** After line 255 (end of the specific-areas example block), still inside `### Visual Focus (v2)` section (lines 226–255), before `### Design Reference (v2)` (line 257).

**Pattern to copy** — the `### Accessibility Focus (v2)` section (lines 195–224) documents opt-in behavior using this structure:
1. **Behavior** block (bullet list: absent / present-empty / present-with-items)
2. **Available focus areas** block (bullet list)
3. **Example (full Tier 2)** fenced code block
4. **Example (specific areas)** fenced code block

The `breakpoint-sweep: continuous` addition follows the same structure but is an additional opt-in modifier appended to the existing examples, not a separate section. Use a `**Optional modifier:**` note pattern.

**Exact content to add** (syntax verified from `agents/qa-tester.md` line 2024):
```markdown
**Optional modifier — continuous breakpoint sweep:**

When `performance-responsive` is listed, the default Phase 1 sweep tests at 6 discrete widths (320, 375, 768, 1024, 1280, 1440px). To also run a continuous sweep in ~50px increments (Phase 2), add `breakpoint-sweep: continuous` inside the `## Visual Focus` section:

```markdown
## Visual Focus
- performance-responsive
breakpoint-sweep: continuous
```

When `breakpoint-sweep: continuous` is present, continuous sweep activates from the start rather than only when a Phase 1 width fails.
```

---

#### Change B: Add `placeholder_allowlist` top-level field documentation

**Location:** After `### Browsers (v2)` section (lines 287–308) and before `### Test Scenarios` (line 310).

**Pattern to copy** — `### Scenario Dependencies (v2)` section (lines 360–397) documents a top-level scenario field using this structure:
1. Section heading (`### Field Name (v2)`)
2. Field syntax in a fenced code block
3. **Behavior** block (bullet list per case)
4. Narrative + **Example** fenced code block

The `placeholder_allowlist` field is a top-level spec field (sits at spec root, parallel to `depends_on:` at the scenario level). Document it as its own `###` subsection of the Section Reference.

**Exact content to add** (syntax and behavior verified from `agents/qa-tester.md` line 1783):
```markdown
### placeholder_allowlist (v2)

A top-level spec field that suppresses false-positive placeholder detections from the `layout-integrity` visual check. When the agent's placeholder regex matches text that is intentionally present (e.g., a legitimate "Learn more" CTA), listing it here prevents it from being reported as a finding.

**Syntax:**

```markdown
placeholder_allowlist: [TODO, Learn more]
```

**Behavior:**

- Field absent: All placeholder regex matches are reported normally
- Field present: Any placeholder finding whose matched text contains a listed string is suppressed
- Suppressed findings are noted in the report: "Suppressed N placeholder finding(s) matching spec allowlist"
- The field is only active when `layout-integrity` is included in `## Visual Focus`

**Example:**

```markdown
placeholder_allowlist: [TODO, Learn more, Click here]

## Visual Focus
- layout-integrity
```

Place `placeholder_allowlist` at the top level of the spec, outside any section header. Strings are matched as substrings — if "Learn more" is in the list, any placeholder finding containing "Learn more" anywhere in its matched text is suppressed.
```

---

### `README.md` — One change: Add three entries to `### v2 Features`

**Location:** After line 298 (end of `#### Accessibility Focus` entry), before line 300 (`See [examples/SPEC-FORMAT.md]...`).

**Pattern to copy** — the existing `#### Accessibility Focus` entry (lines 283–298 of `README.md`) is the direct model. It uses this structure:
1. `####` heading
2. Fenced code block showing the spec section syntax
3. Bullet list of available area names
4. Prose sentence: "Without this section, Tier 1 [X] checks always run... With this section, the agent runs the full Tier 2 structured methodology."
5. Optional: one-liner for activation via skill flag

**Three entries to add** (area names verified from `examples/SPEC-FORMAT.md` lines 238–241; syntax verified from `agents/qa-tester.md`):

```markdown
#### Visual Focus

Add a `## Visual Focus` section to activate Tier 2 structured visual verification for specific areas:

```markdown
## Visual Focus
- design-verification
- layout-integrity
```

Available areas: `design-verification`, `ux-states`, `layout-integrity`, `performance-responsive`

Without this section, Tier 1 baseline visual observations run during functional testing. With this section, the agent runs the full Tier 2 structured methodology for each listed area.

#### Design Reference

Add a `## Design Reference` section to provide mockup images for visual comparison during `design-verification`:

```markdown
## Design Reference

### desktop
- home-default: .qa/designs/desktop-home.png

### mobile
- home-default: .qa/designs/mobile-home.png
```

Each viewport gets a subsection. Each entry maps a state name to an image path (relative to project root). When absent, no design comparison runs (backward compatible).

#### Browsers

Add a `## Browsers` section to run the full test suite across multiple browser engines:

```markdown
## Browsers
- chromium
- webkit
- firefox
```

Valid engines: `chromium` (Chrome/Edge), `webkit` (Safari), `firefox` (Gecko). When absent, only Chromium runs. Activate browser selection at run time with `/qa:run --project <path> --browsers`.
```

---

### `CLAUDE.md` — One change: Append two bullets to `### Agent Architecture` list

**Location:** Line 58 (end of the current bullet list in `### Agent Architecture`), inside the `## How It Works` section.

**Pattern to copy** — the existing bullet list (lines 49–58 of `CLAUDE.md`) uses this format exactly:
```
- [Capability name] ([parenthetical scope/detail])
```

Examples from the existing list:
- `- Structured accessibility testing (Tier 1 baseline + Tier 2 deep checks)`
- `- Spec v2 features (dependencies, data-driven scenarios, tags, environments)`

**Two bullets to append** (content verified from `RESEARCH.md` MISSING-06 fix and cross-checked with `agents/qa-tester.md`):
```markdown
- Visual & UX design verification (Tier 2 — design reference comparison, UX states, layout integrity, performance/responsive, triggered by `## Visual Focus` in specs)
- Cross-browser engine testing (Chromium, WebKit, Firefox via `## Browsers` in specs)
```

---

## Shared Patterns

### Documentation consistency rule: Tier 1 / Tier 2 framing

**Source:** `examples/SPEC-FORMAT.md` `### Accessibility Focus (v2)` (lines 195–204) and `README.md` `#### Accessibility Focus` (lines 283–298)

**Apply to:** All new Visual Focus documentation in ORCHESTRATION.md, SPEC-FORMAT.md, and README.md

The established framing is:
- "Absent: Tier 1 only ([baseline behavior description])"
- "Present with no items listed: Full Tier 2 (all areas)"
- "Present with specific areas listed: Tier 2 for those areas only"

Every new section describing `## Visual Focus` behavior must use this exact three-case structure. Do not invent new language.

---

### Documentation consistency rule: v2 label in section headings

**Source:** `examples/SPEC-FORMAT.md` `### Accessibility Focus (v2)` (line 195), `### Visual Focus (v2)` (line 226), `### Design Reference (v2)` (line 257), `### Browsers (v2)` (line 287)

**Apply to:** `placeholder_allowlist` section heading in SPEC-FORMAT.md

New section heading must be `### placeholder_allowlist (v2)` — the `(v2)` suffix is the established convention for all spec features added in v2.

---

### Documentation consistency rule: Backward-compatibility note placement

**Source:** `examples/SPEC-FORMAT.md` lines 479–481 (`## Backward Compatibility` section), `docs/ORCHESTRATION.md` `/qa:run` argument table footnote (line 269)

**Apply to:** Any new field documentation in SPEC-FORMAT.md

The established pattern is: document the absent-section default behavior inline within the field description ("Section absent: [backward-compatible behavior]"), not in a separate backward-compat section. Do not add a new backward-compat paragraph — include it in the **Behavior** bullet list.

---

## No Analog Found

None. All four files are self-analogous — this phase edits existing well-structured documents by inserting content that follows patterns already present in those same documents.

---

## Metadata

**Analog search scope:** `docs/`, `examples/`, `README.md`, `CLAUDE.md`, `agents/qa-tester.md`, `skills/run/SKILL.md`
**Files read:** 7 (ORCHESTRATION.md, SPEC-FORMAT.md, README.md, CLAUDE.md, agents/qa-tester.md lines 1770-2070, skills/run/SKILL.md, RESEARCH.md)
**Pattern extraction date:** 2026-04-21

**Ground-truth source for all syntax:** `agents/qa-tester.md`
- `placeholder_allowlist` behavior: line 1783
- `breakpoint-sweep: continuous` behavior: line 2024
- Browser engine values: execution flow step 10 in `skills/run/SKILL.md`
