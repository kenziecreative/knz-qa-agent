---
name: monitor
description: Run recurring smoke tests against a live URL via /loop, detect regressions, and persist results across sessions
arguments: "--project <path> --url <base-url> [--tag <tag>] [--interval <minutes>] [--runs <count>]"
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# /qa:monitor

Run recurring smoke tests against a live application. Detects regressions, persists results to `.qa/reports/` and `.qa/HISTORY.md`, and writes alert markers for the SessionStart hook to surface on next session.

## Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` specs |
| `--url <base-url>` | Yes | — | Target URL to monitor |
| `--tag <tag>` | No | `smoke` | Tag filter — only runs scenarios with this tag |
| `--interval <minutes>` | No | `5` | Minutes between runs |
| `--runs <count>` | No | `continuous` | Number of runs before stopping, or `continuous` |

## Usage

```
/qa:monitor --project ~/Projects/my-app --url https://my-app.example.com
/qa:monitor --project ~/Projects/my-app --url https://staging.my-app.com --tag critical
/qa:monitor --project ~/Projects/my-app --url https://my-app.example.com --interval 10 --runs 12
```

## Execution Flow

### Step 1: Parse Arguments

Extract from the user's input:
- **project**: Path to the project with `.qa/` specs
- **url**: Base URL to test against
- **tag**: Tag filter (default: `smoke`)
- **interval**: Minutes between runs (default: `5`)
- **runs**: Number of iterations (default: `continuous`)

### Step 2: Validate Setup

Check that the project has QA specs configured:

```bash
ls {project}/.qa/ 2>/dev/null
```

If `.qa/` does not exist, stop and tell the user to run `/qa:init` first.

Search for specs containing the requested tag:

```bash
grep -rl "tags:.*{tag}" {project}/.qa/*.md 2>/dev/null
```

If no specs have scenarios tagged with the requested tag (default: `smoke`), warn the user:

```
No scenarios tagged "{tag}" found in {project}/.qa/

To use monitoring, add tags to your scenario frontmatter:
  tags: smoke

Or run with a different tag:
  /qa:monitor --project {project} --url {url} --tag critical
```

Do not proceed if no tagged scenarios exist.

### Step 3: Initialize Monitoring State

**Create HISTORY.md if it doesn't exist:**

```bash
cat {project}/.qa/HISTORY.md 2>/dev/null || echo "# QA Monitor History" > {project}/.qa/HISTORY.md
```

**Load or create the baseline file** (`.qa/.monitor-baseline.json`):

```bash
cat {project}/.qa/.monitor-baseline.json 2>/dev/null
```

The baseline file is a JSON object mapping scenario names to their last-known-pass status:

```json
{
  "Login flow": "pass",
  "Homepage loads": "pass",
  "Checkout completes": "pass"
}
```

If the file does not exist, it will be created after the first clean run.

**Confirm monitoring is starting:**

```
[Monitor] Starting monitoring of {url}
  Tag: {tag}
  Interval: {interval} minutes
  Runs: {runs}
  Project: {project}

Press Ctrl+C or end the session to stop monitoring.
```

### Step 4: Set Up Recurring Execution via /loop

Use `/loop` to run the monitoring iteration on the specified interval. The loop body is the monitoring iteration described in Step 5.

Configure `/loop` with:
- **Interval**: `{interval}` minutes
- **Iterations**: `{runs}` (or indefinite if `continuous`)
- **Task**: The monitoring iteration (Step 5) for each cycle

If `/loop` is not available, run iterations manually in sequence, pausing the specified interval between each. Notify the user that manual sequencing is being used.

### Step 5: Monitoring Iteration (Each Loop Cycle)

Each iteration follows this sequence:

#### 5a. Run QA Tests

Invoke `/qa:run` with the tag filter:

```
/qa:run --project {project} --url {url} --tag {tag}
```

Capture the complete results, including:
- List of scenarios run and their pass/fail status
- Any failures with details
- Overall result (Pass / Fail / Pass with Concerns)
- Duration

#### 5b. Detect Regressions

Load the current baseline from `.qa/.monitor-baseline.json`:

```bash
cat {project}/.qa/.monitor-baseline.json 2>/dev/null
```

For each scenario in the current run:

1. **If scenario FAILED and baseline shows it as `"pass"`** → this is a **regression**. Collect regression details: scenario name, failure message, and the fact it previously passed.

2. **If scenario PASSED** → not a regression. Note it as passing.

3. **If scenario is not in baseline** → it's a new scenario. Not a regression. Will be added to baseline on next clean run.

4. **If scenario FAILED and baseline shows it as `"fail"` or it's not in baseline** → known failure or new failure. Not a regression. Still reported, but not flagged as regression.

Regressions are the key alert signal: something that was working is now broken.

#### 5c. Update Baseline

**After a clean run (all scenarios passed):** Update the baseline file to reflect current state.

Write to `.qa/.monitor-baseline.json`:

```json
{
  "{scenario-name-1}": "pass",
  "{scenario-name-2}": "pass"
}
```

This keeps the baseline current so new scenarios get added over time and the regression signal stays accurate.

**After a run with failures:** Do NOT update the baseline. The baseline represents the last-known-good state.

#### 5d. Write Timestamped Report

Generate an ISO timestamp:

```bash
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

Write a report to `.qa/reports/monitor-{ISO-timestamp}.md`:

```markdown
---
type: monitor
run-at: {ISO-timestamp}
url: {url}
tag: {tag}
result: {Pass | Fail | Pass with Concerns}
regressions: {count}
---

# Monitor Run: {ISO-timestamp}

**URL:** {url}
**Tag filter:** {tag}
**Result:** {Pass | Fail | Pass with Concerns}
**Regressions:** {count}

## Scenario Results

| Scenario | Result | Note |
|----------|--------|------|
| {scenario-name} | PASS / FAIL | (regression) if applicable |

## Regression Details

{If regressions exist:}
These scenarios previously passed but now fail:

{For each regression:}
### {scenario-name}
- **Status in baseline:** pass
- **Current status:** fail
- **Failure:** {failure details}

{If no regressions:}
No regressions detected.

## Failures (Non-Regression)

{Any new or pre-existing failures not flagged as regressions, or "None"}
```

#### 5e. Append to HISTORY.md

Append a one-line summary entry to `.qa/HISTORY.md`:

```markdown
## Run: {ISO-timestamp}
- **URL**: {url}
- **Tag**: {tag}
- **Results**: {passed} passed, {failed} failed, {concerns} concerns
- **Regressions**: {none | N regressions: scenario-name-1, scenario-name-2}
- **Duration**: {duration}

```

#### 5f. Write Alert Marker (If Regressions Detected)

If any regressions were detected, write `.qa/.monitor-alert`:

```bash
cat > {project}/.qa/.monitor-alert << 'EOF'
{ISO-timestamp} — {N} regression(s) detected

Regressions (previously passing, now failing):
{For each regression:}
- {scenario-name}: {brief failure summary}

Latest report: .qa/reports/monitor-{ISO-timestamp}.md
Run `/qa:report` for trend analysis or `/qa:run --tag {tag}` to investigate.
EOF
```

This file is read by the SessionStart hook on the next session. The hook will present the alert and delete the file.

**If no regressions:** Do not write or modify the alert file. If a previous alert file exists from an older run, leave it — SessionStart will handle clearing it.

#### 5g. Print Status Line

After each iteration, print a brief status to the user:

```
[Monitor] Run {N}: {passed}/{total} passed — {no regressions | N regression(s) detected}
```

If regressions: also print the scenario names on the next line, indented:

```
[Monitor] Run {N}: {passed}/{total} passed — 2 regression(s) detected
  - Login flow (was passing)
  - Checkout completes (was passing)
```

### Step 6: End Monitoring

When the loop completes (either `--runs` count reached or user stops):

Print a summary:

```
[Monitor] Monitoring complete.
  Runs completed: {N}
  Total regressions detected: {total}
  Reports saved to: {project}/.qa/reports/
  Full history: {project}/.qa/HISTORY.md
```

## Files Written

| File | Purpose |
|------|---------|
| `.qa/reports/monitor-{timestamp}.md` | Full report for each run |
| `.qa/HISTORY.md` | Append-only run log across all sessions |
| `.qa/.monitor-baseline.json` | Baseline scenario states for regression detection |
| `.qa/.monitor-alert` | Alert marker written when regressions detected; read by SessionStart |

## Regression Detection Algorithm

A regression is specifically: a scenario that **passed** in the most recent baseline but **fails** in the current run.

This means:
- **Scenario passes** → not a regression
- **Scenario fails, not in baseline** → new failure (not a regression — no prior passing state to compare)
- **Scenario fails, baseline shows "fail"** → known failure (not a regression — was already failing)
- **Scenario fails, baseline shows "pass"** → regression (was working, now broken)

The baseline only updates on clean runs. This preserves the "last known good" state so regressions remain detectable across multiple consecutive failure runs.

## Examples

**Start continuous monitoring of production:**
```
/qa:monitor --project ~/Projects/store --url https://store.example.com
```

**Monitor staging every 10 minutes, run 6 times (1 hour):**
```
/qa:monitor --project ~/Projects/store --url https://staging.store.com --interval 10 --runs 6
```

**Monitor critical paths (not just smoke):**
```
/qa:monitor --project ~/Projects/store --url https://store.example.com --tag critical
```
