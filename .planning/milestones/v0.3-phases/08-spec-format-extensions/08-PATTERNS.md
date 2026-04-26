# Phase 8: Spec Format Extensions - Pattern Map

**Mapped:** 2026-04-19
**Files analyzed:** 4 (3 modified, 1 optional new)
**Analogs found:** 4 / 4

---

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `examples/SPEC-FORMAT.md` | config/docs | transform | self (existing sections) | exact — extend same doc |
| `agents/qa-tester.md` | agent instruction | event-driven | self (lines 719-731) | exact — mirror existing Accessibility Focus Trigger block |
| `skills/run/SKILL.md` | skill/orchestrator | request-response | self (lines 79-85, 119-129) | exact — extend existing invocation template blocks |
| `examples/visual-testing-spec.md` (optional) | config/docs | transform | `examples/sample-login-spec.md` | exact — same spec file format |

---

## Pattern Assignments

### `examples/SPEC-FORMAT.md` (documentation, additive edits)

**Analog:** Self — existing sections in the same file.

**Pattern 1 — Accessibility Focus section reference block** (lines 183-212):
This is the template to mirror for the new `### Visual Focus (v2)` entry. The block has:
- One-line summary of what the section controls
- Behavior bullet list (absent / present empty / present with items)
- Available focus areas table or list
- Two fenced example blocks (one for each usage mode)

```markdown
### Accessibility Focus (v2)

Controls the depth of accessibility testing. When present, the agent runs Tier 2 structured accessibility checks (the full methodology). When absent, only baseline Tier 1 checks run.

**Behavior:**

- Absent: Tier 1 only (keyboard navigation, focus indicators, alt text, color contrast, form labels)
- Present with no items listed: Full Tier 2 (all areas)
- Present with specific areas listed: Tier 2 for those areas only

**Available focus areas:**

- `focus-management` — modal focus trapping, skip links, aria-live regions
- `page-structure` — heading hierarchy, landmarks, page title, lang attribute
- `interactive-elements` — touch targets, color sole indicator, reduced motion, zoom
- `form-accessibility` — error summary, error links, aria-invalid, focus on first error

**Example (full Tier 2):**

```markdown
## Accessibility Focus
```

**Example (specific areas):**

```markdown
## Accessibility Focus
- focus-management
- form-accessibility
```
```

**Pattern 2 — Environments section reference block** (lines 139-181):
Structural template for the new `### Design Reference (v2)` entry. The block uses `### named-subsections` within an example fenced block, with key: value pairs. Design Reference mirrors this but uses bullet items (`- state-name: path`) instead of YAML key-value pairs.

```markdown
### Environments (v2)

Named environment profiles for running the same spec against different targets ...

**Profile fields:**
- `base_url` — overrides the spec's `## Base URL`
...

**Example:**

```markdown
## Environments

### local
base_url: http://localhost:3000
...
```

Select a profile at runtime: `/qa:run --env staging`
```

**Pattern 3 — Viewports section reference block** (lines 126-131):
Structural template for the new `### Browsers (v2)` entry. A compact reference with a plain list of accepted values.

```markdown
### Viewports
Screen sizes to test. The agent will run through scenarios at each viewport size. Common presets:
- desktop: 1280x720
- tablet: 768x1024
- mobile: 375x667
```

**Pattern 4 — Backward Compatibility clause** (line 385):
The one-line addendum target. Current text:

```markdown
All v1 specs continue to work without modification. The v2 fields (`tags:`, `depends_on:`, `## Data Sets`, `## Environments`, `## Accessibility Focus`) are all optional. A spec with none of these fields behaves exactly as before.
```

Extend by appending the three new sections to the parenthetical list.

**Recommended placement of new sections within SPEC-FORMAT.md:**
- `### Visual Focus (v2)` — immediately after `### Accessibility Focus (v2)` (lines 183-212)
- `### Design Reference (v2)` — immediately after `### Visual Focus (v2)`
- `### Browsers (v2)` — immediately after `### Design Reference (v2)`, before `### Test Scenarios`

---

### `agents/qa-tester.md` (agent instruction, additive edits)

**Analog:** Self — lines 719-731 (`### Accessibility Focus Trigger`).

**Core pattern to mirror verbatim** (lines 719-731):

```markdown
### Accessibility Focus Trigger

The `## Accessibility Focus` section in a spec controls accessibility testing depth:

- **Section ABSENT:** Run only baseline Tier 1 checks (the "Accessibility Checks" section — keyboard navigation, focus indicators, alt text, contrast, form labels). This is the default.
- **Section PRESENT with no items listed:** Run FULL Tier 2 structured accessibility checks — all areas in the "Structured Accessibility" section (Focus Management, Page Structure, Interactive Elements, Form Accessibility).
- **Section PRESENT with specific areas listed:** Run Tier 2 checks for only the listed areas. Valid area names:
  - `focus-management` — Modal/Dialog Focus, Dynamic Content Announcements, Skip Links
  - `page-structure` — Heading Hierarchy, Landmark Regions, Page Title on SPA Navigation, Language Attribute
  - `interactive-elements` — Touch Target Sizes, Color as Sole Indicator, Reduced Motion, Zoom to 200%
  - `form-accessibility` — Error Summary, Error Summary Links, aria-invalid on Errored Fields, Focus on First Error

**This is a depth toggle, not an on/off switch.** Baseline Tier 1 checks always run regardless of this section's presence. The Accessibility Focus section only controls whether Tier 2 also runs (and which parts of it).
```

**Three new sections to add — placement:** Insert immediately after line 731 (end of `### Accessibility Focus Trigger` block), before line 733 (`## Memory & Persistence`).

**New section 1 — Visual Focus Trigger** (mirrors lines 719-731 exactly):
Replace area names and tier descriptions with:
- `design-verification` — compare live page against design reference images (Phase 9)
- `ux-states` — verify empty, loading, error, and interaction states (Phase 10)
- `layout-integrity` — spacing, alignment, grid integrity, content clipping (Phase 11)
- `performance-responsive` — Core Web Vitals, cross-browser rendering, viewport sweep (Phase 12)

Closing note must include the placeholder: "If a Visual Focus section is present in a spec before those phases ship, note in the report: 'Visual Focus requested for [areas] — full methodology pending Phase [N].'"

**New section 2 — Design Reference** (new pattern, no direct analog):
Describes the keyed subsection structure and the three-case behavior:
- Viewport subsection present → use its image paths for reference
- Viewport subsection absent → skip visual diff, note "no design reference for [viewport]" in report
- Section entirely absent → no design comparison (backward compatible)
- Actual comparison methodology is Phase 9 scope — do not describe it here.

**New section 3 — Browsers** (new pattern):
Short section informing the agent that the active engine is passed in the test assignment header by the skill. Agent's only responsibility: label all findings and the report header with the active engine name (e.g., `[webkit] Login form — PASS`).

---

### `skills/run/SKILL.md` (skill instruction, additive edits)

**Analog:** Self — the Agent Invocation template block (lines 99-191).

**Pattern to extend — Execution Flow parsing block** (lines 69-85):
Current environment resolution at line 79 and tag filtering at lines 80-85 show the parse-then-forward pattern. The new Browsers section follows the same pattern: parse bullet list from spec, then forward to agent.

```markdown
3. **Environment resolution**: If `--env` flag provided, find the matching profile in `## Environments`. Extract base_url, credentials, timeouts. If profile not found, warn user and fall back to spec defaults. If no `--env` flag and spec has `## Environments`, use the first profile. If spec has no `## Environments`, use `## Base URL` (backward compatible).
4. **Tag filtering**: If `--tag` flag provided, filter scenario list to only include scenarios whose `tags:` field contains at least one of the requested tags. ...
```

Add step 10 (after step 9): **Browser engine resolution** — if spec has `## Browsers`, extract the bullet list. If section absent, default to `chromium`. Run the full test once per listed engine.

**Pattern to extend — Agent Invocation template** (lines 119-138):
Current conditional blocks (`{If environment profile active:}`, `{If tag filtering active:}`, `{If spec has dependencies:}`) show the forwarding block pattern. Add three new conditional blocks immediately after the existing Execution Notes blocks and before line 139 (`{If ad-hoc:}`):

```markdown
{If spec has Accessibility Focus section:}
## Accessibility Focus
**Areas**: {area list, or "all (full Tier 2)" if present but empty}
Run Tier 2 accessibility checks per your Accessibility Focus Trigger.
```

This existing accessibility block (added in a prior phase, reference only) is the pattern. The new blocks to add for this phase:

```markdown
{If spec has Visual Focus section:}
## Visual Focus
**Areas**: {area list, or "all (full Tier 2)" if present but empty}
Run Tier 2 visual checks per your Visual Focus Trigger.

{If spec has Design Reference section:}
## Design Reference
{full content of the ## Design Reference section from the spec}

{If spec has Browsers section:}
## Browser Engines
**Engines to test**: {comma-separated engine list from spec}
**Active engine for this run**: {current engine}
Run the full test once per engine. Report results labeled by engine: "[chromium] Scenario X: PASS"

{If spec has no Browsers section:}
## Browser Engines
**Engine**: chromium (default)
```

**Note on multi-engine looping:** The skill must loop one agent invocation per engine when `## Browsers` is present — not pass all engines to a single agent run. Each agent invocation receives one `Active engine for this run` value. This is the skill's orchestration responsibility; the agent is stateless per invocation.

---

### `examples/visual-testing-spec.md` (optional new file)

**Analog:** `examples/sample-login-spec.md` — exact format match.

**Pattern to copy** (sample-login-spec.md full structure):
- Frontmatter: none (plain markdown)
- Top-level heading: `# [Feature] Spec`
- Sections in order: Overview, Base URL, Personas, Viewports, Environments, Accessibility Focus
- New sections to add after Accessibility Focus: `## Visual Focus`, `## Design Reference`, `## Browsers`
- Then: Test Scenarios, Edge Cases to Verify, Things to Watch For, Known Issues

The sample should show all three new sections together (this is the standalone example file — not the SPEC-FORMAT.md inline examples, which are kept separate per D-13).

---

## Shared Patterns

### Conditional block syntax (Agent Invocation template)
**Source:** `skills/run/SKILL.md` lines 108-138
**Apply to:** All new forwarding blocks in the invocation template
```
{If spec has [Section]:}
## [Section Name]
...
```
Blocks are prose-conditional (evaluated by the skill author/Claude reading the skill), not code. Use this exact curly-brace notation for consistency with existing blocks.

### Tier toggle pattern (Agent instruction sections)
**Source:** `agents/qa-tester.md` lines 719-731
**Apply to:** Visual Focus Trigger section
The pattern structure is:
1. One-sentence intro naming the spec section and what it controls
2. Three bullet cases: ABSENT / PRESENT empty / PRESENT with items
3. Valid area names as sub-bullets under the third case
4. Bold closing sentence: "This is a depth toggle, not an on/off switch. Baseline [Tier 1] checks always run..."

### Section Reference entry pattern (SPEC-FORMAT.md)
**Source:** `examples/SPEC-FORMAT.md` lines 183-212 (Accessibility Focus entry)
**Apply to:** All three new Section Reference entries
Structure: `### Section Name (v2)` heading → one-line summary → `**Behavior:**` list → `**Available focus areas:**` or equivalent field list → fenced `**Example:**` block(s).

---

## No Analog Found

All files have close analogs in the codebase. No files require falling back to RESEARCH.md patterns exclusively.

| File | Note |
|------|------|
| `agents/qa-tester.md` — Design Reference section | No existing "image path reference" handling pattern in the agent, but the behavior spec is minimal (3 cases, no comparison logic). The agent section is new prose, not mirroring an existing block. |
| `agents/qa-tester.md` — Browsers section | No existing "multi-engine" section in the agent. The section is short — agent's only job is to label output with the engine name passed by the skill. |

---

## Metadata

**Analog search scope:** `examples/`, `agents/`, `skills/run/`
**Files scanned:** 4
**Pattern extraction date:** 2026-04-19

**Key constraint reminders for planner:**
- Do NOT edit lines 719-731 of `qa-tester.md` — add Visual Focus Trigger as a NEW section after line 731
- The multi-engine loop is the SKILL's job, not the agent's — agent receives one engine per invocation
- Backward compat clause (SPEC-FORMAT.md line 385) must be updated — it is easy to miss
- Phase 8 installs vocabulary and triggers only; Tier 2 visual methodology is a placeholder pointing to Phases 9-12
