# QA Plugin Orchestration Guide

This document is the primary reference for orchestrating AI agents and human developers using the QA plugin. The plugin provides 7 skills for testing web applications via Claude Code: `/qa:init`, `/qa:gen`, `/qa:run`, `/qa:check`, `/qa:monitor`, `/qa:report`, and `/qa:guide`. Read this document from top to bottom the first time; use section anchors for lookup thereafter. No prior knowledge of the plugin source files is assumed.

---

## Quick Reference (For Orchestrating Agents)

Machine-parseable lookup table. Read this first to select the right skill.

| Skill | Purpose | Spawns Agent? | Key Arguments |
|-------|---------|---------------|---------------|
| `/qa:init` | Scaffold `.qa/` directory and template spec | No | `path` |
| `/qa:gen` | Generate a spec from description, docs, or URL | Yes | `<spec-name>`, `--from`, `--a11y-depth` |
| `/qa:run` | Execute tests from spec files or ad-hoc instructions | Yes | `--project`, `--spec`, `--url`, `--tag`, `--env` |
| `/qa:check` | Verify phase deliverables as a GSD gate check | Yes | `--url`, `--phase` |
| `/qa:monitor` | Recurring smoke tests via /loop, persists regression alerts | Yes | `--project`, `--url`, `--tag`, `--interval`, `--runs` |
| `/qa:report` | Cross-session report with trend analysis and GitHub issue creation | No | `--project`, `--format`, `--since`, `--create-issues` |
| `/qa:guide` | Inspect project QA state and recommend next action | No | `--project` |

**Agent-spawning skills** (`gen`, `run`, `check`, `monitor`) launch the `qa-tester` agent with a browser session via `playwright-cli`.

**File-read-only skills** (`init`, `report`, `guide`) read from disk and produce output without spawning a browser agent.

---

## Interpreting Results

### Result Format

Results from `/qa:check` follow one of three structured output blocks an orchestrating agent can match against:

**Pass:**
```
✅ Phase {N} Verification: PASSED

Verified:
- {deliverable 1}: Works
- {deliverable 2}: Works

Confidence: High — all deliverables verified with evidence.

Notes:
- {any observations for later}

Ready to proceed to next phase.
```

**Fail:**
```
❌ Phase {N} Verification: FAILED

Definite Issues:
1. {Issue description}
   - Confidence: High — {rationale}
   - Evidence: {what was observed}
   - Severity: {Blocker / Major / Minor}

Likely Issues:
(if any)

Possible Issues:
(if any)

Recommendation: Address these issues before proceeding.
```

**Partial pass:**
```
⚠️ Phase {N} Verification: PASSED WITH CONCERNS

Working:
- {deliverable 1}: Works

Possible Issues:
- {issue that's not a blocker but should be noted}
  - Confidence: Low — {rationale}

Recommendation: Can proceed, but schedule follow-up for noted concerns.
```

Results from `/qa:run` follow the same confidence grouping (Definite / Likely / Possible) without the Phase gate framing.

### Confidence Levels

Every finding the agent produces is tagged with one of three confidence levels:

| Level | Meaning | Action |
|-------|---------|--------|
| High (Definite) | Confirmed real AND impactful — agent has strong evidence | Fix before proceeding |
| Medium (Likely) | Probably real, needs investigation — may be context-dependent | Investigate; may not block |
| Low (Possible) | Worth noting, may be intentional behavior | Review at leisure |

All findings are shown in reports — none are filtered. Grouping is by confidence, not severity.

### Parsing Result Data

An orchestrating agent should parse results as follows:

```
# Determine pass/fail from /qa:check output:
- Look for line starting with "✅" → PASSED
- Look for line starting with "❌" → FAILED
- Look for line starting with "⚠️" → PASSED_WITH_CONCERNS

# Extract issue severity from FAILED block:
- "Severity: Blocker" → must fix before proceeding
- "Severity: Major" → high priority, likely blocking
- "Severity: Minor" → low priority, can defer

# Check /qa:report trend output for:
- "RECURRING: {scenario name}" → escalated recurring failure (3+ runs)
- "Regressions: {count}" lines in HISTORY.md → trend direction

# 3-failure escalation rule:
- If /qa:check returns FAILED three times for the same phase:
  → Stop and escalate. The implementation approach may need reconsideration.
  → Do not continue invoking /qa:check expecting a different result.
```

---

## Workflow Integration

### Recommended Flow

A typical project lifecycle using the QA plugin:

1. **`/qa:init`** — Scaffold the `.qa/` directory and a starter `smoke-test.md` spec
2. **`/qa:gen`** — Generate specs from feature descriptions or by exploring the running app
3. **`/qa:run`** — Execute tests against local or staging URL; reports saved to `.qa/reports/`
4. **`/qa:report`** — Review latest results and trend analysis across sessions
5. **`/qa:monitor`** — Set up recurring smoke checks for deployed applications

For a new feature under development, the sequence is typically: gen → run → (fix issues) → run again → report.

### GSD Phase Integration

`/qa:check` is designed for use at GSD phase boundaries. Add this to a project's executor methodology:

```markdown
## Verification
This is a web application. After completing each phase, run `/qa:check --url http://localhost:3000`
to verify the deliverables work before proceeding.
```

**How `/qa:check` works as a gate:**
1. It reads the active phase's plan to understand what was supposed to be delivered
2. Spawns `qa-tester` to verify deliverables at the provided URL
3. Returns a structured pass/fail result the orchestrator can act on

**3-failure escalation rule:** If `/qa:check` fails three times for the same phase, stop retrying. Surface this escalation message to the human:

> "Phase {N} has failed verification three times. The repeated failures suggest the implementation approach may have a fundamental issue. Recommend pausing to review: Is the technical approach sound? Are there missing requirements or misunderstandings? Should this be broken into smaller phases?"

### Decision Points for Orchestrating Agents

Use this decision list to determine which skill to invoke next:

- **No `.qa/` directory exists?** → Run `/qa:init {project-path}`
- **No spec files in `.qa/`?** → Run `/qa:gen {spec-name} "description of what to test"`
- **Need to verify a phase before marking it complete?** → Run `/qa:check --url <app-url> --phase N`
- **Need comprehensive testing across all scenarios?** → Run `/qa:run --project <path>`
- **Need only smoke tests?** → Run `/qa:run --project <path> --tag smoke`
- **Want trend data across multiple test sessions?** → Run `/qa:report --project <path>`
- **Deploying to staging or production?** → Run `/qa:monitor --project <path> --url <deployed-url>`
- **Unsure what the project needs next?** → Run `/qa:guide --project <path>`

---

## Skill Reference

### /qa:init

Initialize QA testing structure in a project. Creates the `.qa/` directory and a starter `smoke-test.md` spec.

**Does not spawn the `qa-tester` agent.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `path` | No | `.` (current directory) | Path to the project to initialize |

#### Usage

```bash
# Initialize in a specific project
/qa:init ~/Projects/my-app

# Initialize in current directory
/qa:init .
```

#### What It Produces

Creates:
```
{project}/
└── .qa/
    ├── README.md       # Quick reference for spec format
    └── smoke-test.md   # Starter spec with example scenarios
```

#### When to Use

Use `/qa:init` exactly once per project before running any other QA skills. If `.qa/` already exists, do not re-run — it will overwrite the existing structure.

---

### /qa:gen

Generate a QA test specification from a description, existing documentation, or by exploring a running application.

**Spawns the `qa-tester` agent.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `<spec-name>` | Yes | — | Name for the generated spec file (no `.md` extension) |
| `--from <source>` | No | — | Source to generate from: file path or URL |
| `--a11y-depth <level>` | No | `baseline` | Accessibility depth: `baseline` (Tier 1 only) or `deep` (adds Tier 2 structured checks) |
| `description` | No | — | Natural language description of what to cover |

#### Usage

```bash
# Generate from a plain description
/qa:gen login "User authentication flow with email/password, including forgot password"

# Generate from existing documentation
/qa:gen checkout --from ~/Projects/store/docs/checkout-flow.md

# Generate by exploring a running app
/qa:gen user-profile --from http://localhost:3000/profile "Explore the user profile page"

# Generate with deep accessibility checks included
/qa:gen dashboard --a11y-depth deep "Main dashboard with charts and data tables"
```

#### What It Produces

Creates `{project}/.qa/{spec-name}.md` — a spec file in the v2 format that can be run immediately with `/qa:run`.

#### When to Use

Use `/qa:gen` when starting testing on a new feature or page. It saves time compared to writing specs manually and ensures the correct spec format.

---

### /qa:run

Execute QA tests from spec files or ad-hoc instructions. The most frequently used skill.

**Spawns the `qa-tester` agent.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | No* | — | Path to project containing `.qa/` specs |
| `--spec <name>` | No | all specs | Specific spec name to run (no `.md` extension) |
| `--url <base-url>` | No* | from spec | Base URL to test against (overrides spec's Base URL) |
| `--tag <tag>` | No | all scenarios | Filter to scenarios matching this tag. Comma-separated for OR logic: `--tag smoke,critical` |
| `--env <profile>` | No | first profile | Environment profile name from spec's `## Environments` section |
| `task` (quoted) | No* | — | Ad-hoc test description when not using a spec file |

*Either `--project` or (`--url` + ad-hoc `task`) must be provided.

#### Usage

```bash
# Run all specs in a project
/qa:run --project ~/Projects/my-app

# Run a specific spec
/qa:run --project ~/Projects/my-app --spec login

# Run against a different URL (overrides spec's Base URL)
/qa:run --project ~/Projects/my-app --spec login --url https://staging.example.com

# Run only smoke-tagged scenarios across all specs
/qa:run --project ~/Projects/my-app --tag smoke

# Run with a specific environment profile
/qa:run --project ~/Projects/my-app --spec checkout --env staging

# Ad-hoc test without a spec file
/qa:run --url http://localhost:3000 "Test that the homepage loads without console errors"
```

#### What It Produces

- Writes a report to `.qa/reports/{timestamp}-{spec-name}.md`
- Appends a summary line to `.qa/HISTORY.md`
- Terminal output with pass/fail per scenario, grouped by confidence

#### When to Use

Use `/qa:run` for comprehensive testing against specs. Use `/qa:check` instead for quick phase gate verification. Use `--tag smoke` for fast health checks before deeper test runs.

---

### /qa:check

Verify that a GSD phase's deliverables actually work. A targeted gate check, not a full QA pass.

**Spawns the `qa-tester` agent.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--url <base-url>` | Yes | — | URL of the running application to test |
| `--phase <number>` | No | inferred from GSD context | Phase number to verify |

#### Usage

```bash
# Verify the active phase (inferred from GSD context)
/qa:check --url http://localhost:3000

# Verify a specific phase
/qa:check --url http://localhost:3000 --phase 3
```

#### What It Produces

A structured pass/fail result (see Interpreting Results section above). If issues are found:
- Lists findings grouped by Definite / Likely / Possible
- Includes evidence (console errors, network failures, screenshots)
- Provides severity: Blocker / Major / Minor

#### When to Use

Use `/qa:check` at GSD phase boundaries before marking a phase complete. It is not a replacement for `/qa:run` with full spec coverage — it only tests the key deliverables of the just-completed phase.

---

### /qa:monitor

Run recurring smoke tests against a deployed application. Detects regressions across multiple automated runs and surfaces alerts at the start of the next session.

**Spawns the `qa-tester` agent repeatedly via `/loop`.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` specs |
| `--url <base-url>` | Yes | — | Target URL to monitor |
| `--tag <tag>` | No | `smoke` | Tag filter — only runs scenarios with this tag |
| `--interval <minutes>` | No | `5` | Minutes between runs |
| `--runs <count>` | No | `continuous` | Number of runs before stopping, or `continuous` |

#### Usage

```bash
# Monitor with defaults (smoke tag, 5 min interval, continuous)
/qa:monitor --project ~/Projects/my-app --url https://my-app.example.com

# Monitor only critical scenarios every 10 minutes
/qa:monitor --project ~/Projects/my-app --url https://staging.example.com --tag critical --interval 10

# Monitor for a fixed number of runs (e.g., during a deployment window)
/qa:monitor --project ~/Projects/my-app --url https://my-app.example.com --runs 12
```

#### What It Produces

- Report files in `.qa/reports/monitor-{timestamp}.md` after each run
- Updated `.qa/HISTORY.md` after each run
- `.qa/.monitor-alert` marker file if a regression is detected — surfaced by the SessionStart hook at the start of the next session

#### When to Use

Use `/qa:monitor` for deployed applications where you need regression detection between manual test sessions. It runs indefinitely (or for `--runs` iterations) in the background. The `smoke` tag filter keeps it fast — full spec runs are for manual or CI invocations.

---

### /qa:report

Generate a cross-session QA report with trend analysis. Reads entirely from disk — no prior conversation context needed.

**Does not spawn the `qa-tester` agent.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` directory |
| `--format <format>` | No | `markdown` | Output format: `markdown` or `json` |
| `--since <date-or-duration>` | No | last 5 runs or 7 days | Historical window for trend analysis. Examples: `--since 2026-03-01` or `--since 7d` |
| `--create-issues` | No | off | Generate GitHub issues from High confidence findings (requires `gh auth`) |

#### Usage

```bash
# Standard report
/qa:report --project ~/Projects/my-app

# Report as JSON (for CI/CD pipelines)
/qa:report --project ~/Projects/my-app --format json

# Trend analysis for the last 2 weeks
/qa:report --project ~/Projects/my-app --since 14d

# Create GitHub issues from critical findings
/qa:report --project ~/Projects/my-app --create-issues

# Historical report from a specific date
/qa:report --project ~/Projects/my-app --since 2026-03-01 --create-issues
```

#### What It Reads

| File | Purpose |
|------|---------|
| `.qa/reports/{most-recent}.md` | Latest run results — primary data source |
| `.qa/HISTORY.md` | Run log across all sessions — used for trend analysis |

#### What It Produces

- Latest run summary (pass/fail counts, findings)
- Trend analysis (recurring failures flagged with `RECURRING:` prefix)
- Optional: GitHub issues created via `gh cli` for High confidence findings

#### When to Use

Use `/qa:report` after completing a testing session, or from CI/CD pipelines to surface results without a browser session. Reports accumulate across sessions — each run adds to the history that trend analysis reads from.

---

### /qa:guide

Inspect the project's QA state and recommend which skill to use next. Context-aware guidance based on what exists in the `.qa/` directory.

**Does not spawn the `qa-tester` agent. Reads from disk only.**

#### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing (or missing) `.qa/` directory |

#### Usage

```bash
# Get recommendations for a project
/qa:guide --project ~/Projects/my-app

# Get recommendations for current directory
/qa:guide --project .
```

#### What It Reads

| File | Purpose |
|------|---------|
| `{project}/.qa/` | Existence check — determines if QA is initialized |
| `{project}/.qa/*.md` | Spec count — determines if specs exist |
| `{project}/.qa/reports/` | Report count — determines if tests have been run |
| `{project}/.qa/HISTORY.md` | Line count — determines trend data readiness |
| `{project}/.qa/.monitor-alert` | Presence check — surfaces active monitoring regressions |

#### What It Produces

A short narrative (2-4 bullet recommendations) grounded in the actual project state. Example output:

```
- No spec files found in .qa/. Run /qa:gen to create a spec from your feature descriptions or by exploring the app.
- 3 spec files found, no reports yet. Run /qa:run --project . to execute your first test pass.
- Last report (2026-04-19): 2 High confidence findings remain open. Run /qa:run --project . --tag regression to recheck.
- MONITORING ALERT active — a regression was detected. Run /qa:report --project . for trend analysis.
```

#### When to Use

Use `/qa:guide` when entering a project for the first time, when unsure which skill fits the current situation, or when you want the plugin to tell you what's needed next based on current state.

---

## Getting Started (For Developers)

### Installation

The QA plugin requires two things:

1. **Plugin installed in your project** — Clone or add the plugin to your Claude Code plugins directory. The plugin directory follows Anthropic's Claude Code plugin convention.

2. **`playwright-cli` installed globally** — All browser automation runs through `playwright-cli`. Install it before running any agent-spawning skill:
   ```bash
   npm install -g playwright-cli
   ```

### First Test Walkthrough

If you're starting fresh on a new project:

**Step 1 — Initialize:**
```
/qa:init ~/Projects/my-app
```
This creates `.qa/` with a starter `smoke-test.md` spec. Open the spec and customize it for your application's base URL and core flows.

**Step 2 — Generate a spec (optional but recommended):**
```
/qa:gen login "User authentication: email/password login, logout, forgot password flow"
```
This generates a more detailed spec than the smoke-test starter. Review the generated spec and adjust as needed.

**Step 3 — Run tests:**
```
/qa:run --project ~/Projects/my-app --spec smoke-test --url http://localhost:3000
```
The agent opens a browser, walks through the spec scenarios, and writes a report to `.qa/reports/`.

**Step 4 — Review results:**
```
/qa:report --project ~/Projects/my-app
```
The report shows what passed, what failed (grouped by confidence), and trend analysis once you have multiple runs.

### Spec Format Overview

Specs are Markdown files in `.qa/`. The format is human-readable and supports:

- **Personas** — test as multiple users (admin, guest, etc.) with credentials
- **Viewports** — mobile (375px), tablet (768px), desktop (1280px) by default
- **Tags** — label scenarios for selective execution (`smoke`, `critical`, `regression`, `a11y`)
- **Dependencies** — declare `depends_on: Scenario Name` when order matters
- **Environments** — define `local`, `staging`, `production` profiles with different URLs and credentials
- **Accessibility Focus** — add `## Accessibility Focus` section to trigger Tier 2 deep accessibility checks

See `examples/SPEC-FORMAT.md` for the complete spec format reference with examples.

### Where Reports Live

All QA output goes into the target project's `.qa/` directory:

- **Reports:** `.qa/reports/{timestamp}-{spec-name}.md` — one file per test run
- **History:** `.qa/HISTORY.md` — running log used for trend analysis
- **Memory:** `.qa/memory/` — patterns and known issues the agent learns across sessions

Reports accumulate automatically — you do not need to manage them.

---

## Data Architecture

QA data lives in the `.qa/` directory of each target project. The plugin never writes outside this directory.

```
{project}/
└── .qa/
    ├── *.md              # Spec files — test definitions
    ├── README.md         # Quick reference (created by /qa:init)
    ├── reports/          # Test reports (generated by /qa:run, /qa:check, /qa:monitor)
    │   └── HISTORY.md    # Running log of all QA activity
    ├── memory/           # QA-specific memory: patterns, known issues, baselines
    │   └── *.md          # Memory files (auto-expire after 30 days of no update)
    ├── .monitor-alert    # Regression alert marker (written by /qa:monitor, deleted by SessionStart)
    └── env.local         # Environment secrets — gitignore this file
```

**Important:** `.qa/env.local` should be in `.gitignore`. It holds real credentials for environment profiles. The spec files themselves should use `$ENV_VAR` placeholders for any secrets.

---

## Hooks (Automatic Behavior)

The plugin includes two hooks that run automatically in Claude Code sessions:

### SessionStart Hook

When a Claude Code session starts in a project with a `.qa/` directory, the hook:

1. Checks for the most recent report in `.qa/reports/` and presents a one-liner if clean, or the full report if findings exist
2. Checks for a `.qa/.monitor-alert` marker file and surfaces monitoring regressions prominently
3. Reads `.qa/memory/*.md` and surfaces known issues or recurring patterns

If `.qa/` does not exist, the hook exits silently — zero noise for projects that don't use QA.

### Stop Hook

When a Claude Code session ends after making changes to source files, the hook checks whether any changed files are test-relevant (source, styles, config, API routes). If so, it suggests running `/qa:run` or `/qa:check` to verify the changes. The suggestion is informational only — not a blocker.

---

## Cross-Session Capability

The following skills and hooks read entirely from disk and require no conversation context:

- **`/qa:report`** — reads `.qa/reports/` and `.qa/HISTORY.md` from any session
- **`/qa:guide`** — reads `.qa/` state from any session
- **SessionStart hook** — reads `.qa/reports/` and `.qa/memory/` at session open

This means reports and guidance accumulate value across sessions. Different engineers working on the same project see the same QA data. CI/CD pipelines can invoke `/qa:report` without a running browser session.

Agent-spawning skills (`run`, `check`, `gen`, `monitor`) always start fresh — they inspect the live application, not cached state. Each run adds to the persistent history that file-read skills consume.
