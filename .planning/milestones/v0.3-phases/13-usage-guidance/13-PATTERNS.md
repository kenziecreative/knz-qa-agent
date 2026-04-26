# Phase 13: Usage Guidance - Pattern Map

**Mapped:** 2026-04-20
**Files analyzed:** 4 (2 new, 2 modified)
**Analogs found:** 4 / 4

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `docs/ORCHESTRATION.md` | documentation | reference | `examples/SPEC-FORMAT.md` | role-match |
| `skills/guide/SKILL.md` | skill | file-I/O | `skills/report/SKILL.md` | exact |
| `README.md` | documentation | reference | `README.md` (self) | self-modify |
| `CLAUDE.md` | config | reference | `CLAUDE.md` (self) | self-modify |

---

## Pattern Assignments

### `docs/ORCHESTRATION.md` (documentation, reference)

**Analog:** `examples/SPEC-FORMAT.md`

`SPEC-FORMAT.md` is the project's only existing static reference doc — a dual-audience document that serves both the AI agent (mechanical spec syntax) and human developers (explanatory prose). `ORCHESTRATION.md` follows this same layered structure but with a harder AI-agent-primary orientation as stated in D-01/D-02.

**Document structure pattern** (`examples/SPEC-FORMAT.md` lines 1–10):
```markdown
# QA Spec Format

Test specifications are written in Markdown and live in the `.qa/` directory of your project.

## Basic Structure

```markdown
# [Feature Name]
...
```
```

Key structural conventions to copy:
- Top-level `#` title that names the document plainly
- Opening paragraph stating the document's scope and where it applies
- `##` section headers as navigation anchors for both AI and human readers
- Code blocks used liberally for anything a machine should parse exactly
- Tables for structured lookup (argument tables, invocation tables)
- `[TODO: ...]` / `[VERIFY: ...]` markers — NOT used in ORCHESTRATION.md (it's authoritative, not a template), but the table and fenced-block conventions carry over

**Dual-audience section labeling** (D-02 decision, no direct analog — use this pattern):

Sections for agents should open with a clear machine-parseable header and contain tables + code blocks. Sections for humans should open with narrative prose. The `README.md` Skills section (lines 28–123) demonstrates the human-facing narrative style to match for the human-oriented sections of ORCHESTRATION.md.

**Skills invocation table shape** — copy from `README.md` lines 28–53 (the Skills listing block):
```markdown
### `/qa:run` - Full Spec Execution

Run comprehensive QA tests from spec files or ad-hoc descriptions.

```bash
# Run a specific spec
/qa:run --project ~/Projects/my-app --spec login
```
```

**Result/output block pattern** — copy from `skills/check/SKILL.md` lines 96–143 (the Pass/Fail/Partial output blocks). These show how the plugin surfaces structured results that an orchestrating agent can parse:
```markdown
**If verification passes:**
```
✅ Phase {N} Verification: PASSED

Verified:
- {deliverable 1}: Works
...

Ready to proceed to next phase.
```
```

**Argument table pattern** — copy from `skills/report/SKILL.md` lines 12–19:
```markdown
| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` directory |
| `--format <format>` | No | `markdown` | Output format: `markdown` or `json` |
```

**Cross-session / file-based data note pattern** — copy the "Cross-Session Capability" section from `skills/report/SKILL.md` lines 436–444 as a model for explaining what /qa:guide reads from disk:
```markdown
## Cross-Session Capability

This skill reads entirely from disk. It does not need prior conversation context. You can run `/qa:report` in a brand new session and get a complete picture...
```

---

### `skills/guide/SKILL.md` (skill, file-I/O)

**Analog:** `skills/report/SKILL.md` (exact match — file-read-only skill, no agent spawn)

`skills/report/SKILL.md` is the canonical file-read-only skill. `/qa:guide` must follow the same structural and behavioral conventions precisely.

**Frontmatter pattern** (`skills/report/SKILL.md` lines 1–6):
```yaml
---
name: qa:guide
description: Inspect project QA state and recommend the next action
argument-hint: "--project <path>"
allowed-tools: Read, Write, Bash, Grep, Glob
---
```

Note: `allowed-tools` MUST NOT include `Agent` — this is the defining difference from agent-spawning skills (`run`, `check`, `gen`, `monitor` all include `Agent`). `init` and `report` do not, and `guide` follows that pattern.

**Opening purpose paragraph pattern** (`skills/report/SKILL.md` lines 10–13):
```markdown
Generate a QA report that reads from persisted files on disk — works without any prior conversation context. Answers two questions: "what just happened?" (latest run) and "is this a pattern?" (trend analysis).
```

The `/qa:guide` equivalent should be a single sentence stating what it produces and that it reads from disk, not session context.

**Arguments table pattern** (`skills/report/SKILL.md` lines 15–19):
```markdown
## Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` directory |
```

**State inspection pattern** (`hooks/session-start.md` lines 12–28):

This is the closest model for the file-inspection logic /qa:guide must replicate. The hook checks `.qa/` existence, report count, and alert file — identical to D-05's five checks. Copy the bash command patterns verbatim:

```bash
# Check .qa/ exists
ls .qa/ 2>/dev/null

# Find report count
ls -t .qa/reports/ 2>/dev/null | head -1

# Check for monitor alert
cat .qa/.monitor-alert 2>/dev/null
```

For spec count (D-05 item 2), use:
```bash
ls {project}/.qa/*.md 2>/dev/null | wc -l
```

For HISTORY.md line count (D-05 item 4):
```bash
wc -l < {project}/.qa/HISTORY.md 2>/dev/null
```

**No-file fallback pattern** (`skills/report/SKILL.md` lines 73–85):
```markdown
**If no reports exist:**

```
No QA reports found in {project}/.qa/reports/

Run tests first to generate reports:
  /qa:run --project {project} --url <your-url>

Then run /qa:report to see results.
```

Stop here. Do not proceed without a report file.
```

`/qa:guide` applies the same graceful-degradation logic per state check: each missing state produces a specific actionable suggestion (D-05), not an error.

**Files Read table pattern** (`skills/report/SKILL.md` lines 445–450):
```markdown
## Files Read

| File | Purpose |
|------|---------|
| `.qa/reports/{most-recent}.md` | Latest run results — primary data source |
| `.qa/HISTORY.md` | Run log across all sessions — used for trend analysis |
```

`/qa:guide` should have an equivalent table listing all five state files it reads.

**Output format pattern** (D-06 — short narrative, 2–4 bullets):

The `report` skill produces a long structured report. The `guide` skill produces short narrative output. Model the conversational framing on `hooks/session-start.md` lines 42–50 and 127–134:
```markdown
Last QA Run: [date] — All clear, no issues.
```

The guide output should be similarly terse and actionable — 2–4 bullet recommendations, not a checklist or table.

**ORCHESTRATION.md read pattern** (D-07):

The skill should read the relevant section of `docs/ORCHESTRATION.md` to inform its guidance. Model this on how `skills/report/SKILL.md` Step 2 makes the disk-read the PRIMARY source (lines 58–60):
```markdown
This is the PRIMARY data source. Do NOT rely on conversation context.
```

For `guide`, the equivalent is: read `docs/ORCHESTRATION.md` to ground recommendations in documented guidance, then combine with the state inspection results.

---

### `README.md` (documentation, reference — MODIFIED)

**Analog:** `README.md` itself (self-modification, add one pointer line + list entry)

**Skills listing pattern** (lines 28–123) — the existing skill entries follow this shape:

```markdown
### `/qa:SKILLNAME` - Short Title

One-sentence description.

```bash
# Example usage
/qa:SKILLNAME --project ~/Projects/my-app
```
```

Add `/qa:guide` as a new entry in this block following that exact shape. No new subsection needed — insert it between `/qa:monitor` and `/qa:report` or at the end of the skills list, per planner's discretion.

**Project Structure listing** (lines 362–386) — the skills directory listing follows this pattern:
```markdown
│   ├── monitor/SKILL.md         # /qa:monitor — recurring smoke tests via /loop
│   └── report/SKILL.md          # /qa:report — cross-session QA reports with trends
```

Add a line for `guide/SKILL.md` with the same inline comment style.

**Pointer line placement** — add a one-line pointer to `docs/ORCHESTRATION.md` in the Development Notes section (lines 396–399) or as a new "Orchestration" item. Keep it minimal — a single link line:
```markdown
- See [docs/ORCHESTRATION.md](docs/ORCHESTRATION.md) for orchestration agent usage and AI integration patterns
```

---

### `CLAUDE.md` (config, reference — MODIFIED)

**Analog:** `CLAUDE.md` itself (self-modification, add one reference line)

**Project Structure block** (lines 9–30) — the file structure listing follows the same pattern as README.md. The `docs/` directory is new and should appear in this listing.

**Integration Points section** (lines 62–66 in CONTEXT.md code_context) — the established convention for calling out integration points is a bullet list. Add the ORCHESTRATION.md reference as a new line in the relevant section.

Exact insertion — the Project Structure section currently ends with:
```markdown
LEARNINGS.md             # Architecture decisions and patterns for this project
README.md                # User-facing documentation
```

Add:
```markdown
docs/
└── ORCHESTRATION.md     # AI agent orchestration reference — how to invoke and integrate the plugin
```

No other changes to CLAUDE.md are required. The file is an AI-facing document; a structural entry and a brief prose note in an appropriate section (e.g., a new "Orchestration" line under "How It Works" or "Development Guidelines") are sufficient.

---

## Shared Patterns

### Skill Frontmatter Convention
**Source:** `skills/report/SKILL.md` lines 1–6 and `skills/run/SKILL.md` lines 1–6
**Apply to:** `skills/guide/SKILL.md`

All skills use YAML frontmatter with these four fields:
```yaml
---
name: qa:SKILLNAME
description: One-sentence description of what the skill produces
argument-hint: "--project <path> [other flags]"
allowed-tools: Read, Write, Bash, Grep, Glob
---
```

`allowed-tools` is the critical differentiator. File-read-only skills (`report`, `init`, and now `guide`) omit `Agent`. Agent-spawning skills (`run`, `check`, `gen`, `monitor`) include `Agent, Edit`.

### State-Inspection Bash Commands
**Source:** `hooks/session-start.md` lines 12–100
**Apply to:** `skills/guide/SKILL.md` (all five state checks from D-05)

The hook already implements four of the five D-05 checks. Copy its bash command patterns directly rather than inventing new ones. The hook uses `2>/dev/null` consistently on every check to silence errors when files don't exist — maintain this discipline in `guide`.

### Graceful Degradation Pattern
**Source:** `skills/report/SKILL.md` lines 73–85
**Apply to:** `skills/guide/SKILL.md`

Every state check in `guide` should have an explicit "not found" branch that produces a specific, actionable next step rather than erroring or producing no output. The `report` skill's "If no reports exist → tell user to run /qa:run first" is the model.

### Document Section Ordering
**Source:** `skills/report/SKILL.md` lines 426–435 ("Report Structure Design")
**Apply to:** `docs/ORCHESTRATION.md`

The report's design rationale explains answering questions in order of urgency. Apply the same principle to ORCHESTRATION.md section ordering: agent-critical information (invocation tables, result parsing) before human-oriented orientation.

### Skill Usage Example Blocks
**Source:** `skills/report/SKILL.md` lines 22–29, `skills/run/SKILL.md` lines 15–50
**Apply to:** `docs/ORCHESTRATION.md` (the per-skill invocation reference tables)

Usage examples consistently use fenced bash blocks with `#` comment lines explaining each variant. Invocation tables in ORCHESTRATION.md should follow the same convention, not invent a different format.

---

## No Analog Found

All four files have close analogs. No files require falling back to external research patterns.

---

## Metadata

**Analog search scope:** `skills/`, `hooks/`, `examples/`, `README.md`, `CLAUDE.md`
**Files scanned:** 9 (report/SKILL.md, init/SKILL.md, run/SKILL.md, gen/SKILL.md, check/SKILL.md, session-start.md, SPEC-FORMAT.md, README.md, CLAUDE.md)
**Pattern extraction date:** 2026-04-20
