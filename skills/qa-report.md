---
name: report
description: Generate a cross-session QA report from persisted reports and history, including trend analysis and optional GitHub issue creation
arguments: "--project <path> [--format <format>] [--since <date-or-duration>] [--create-issues]"
allowed-tools: Read, Write, Bash, Grep, Glob
---

# /qa:report

Generate a QA report that reads from persisted files on disk — works without any prior conversation context. Answers two questions: "what just happened?" (latest run) and "is this a pattern?" (trend analysis).

## Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` directory |
| `--format <format>` | No | `markdown` | Output format: `markdown` or `json` |
| `--since <date-or-duration>` | No | last 5 runs or 7 days | Historical window for trend analysis. Examples: `--since 2026-03-01` or `--since 7d` or `--since 14d` |
| `--create-issues` | No | off | Generate GitHub issues from High confidence findings |

## Usage

```
/qa:report --project ~/Projects/my-app
/qa:report --project ~/Projects/my-app --format markdown
/qa:report --project ~/Projects/my-app --format json
/qa:report --project ~/Projects/my-app --since 14d
/qa:report --project ~/Projects/my-app --create-issues
/qa:report --project ~/Projects/my-app --since 2026-03-01 --create-issues
```

## Purpose

Reads from `.qa/reports/` and `.qa/HISTORY.md` to generate an authoritative report across sessions. This skill does not rely on conversation context — all data comes from files on disk.

Use this skill to:
- See what the latest test run found
- Identify patterns and recurring failures across multiple runs
- Create GitHub issues from critical findings
- Review regressions and improvement trends

## Execution Flow

### Step 1: Parse Arguments

Extract from user input:
- **project**: Path to project with `.qa/` directory
- **format**: `markdown` (default) or `json`
- **since**: Historical window (default: last 5 runs or 7 days, whichever is less)
- **create-issues**: Whether to create GitHub issues (default: false)

Parse the `--since` value:
- If it's a date (e.g., `2026-03-01`), use it as a cutoff date
- If it's a duration (e.g., `7d`, `14d`), calculate the cutoff date by subtracting from today
- If not provided, use last 5 runs or 7 days (whichever covers fewer runs)

### Step 2: Read Latest Report from Disk

This is the PRIMARY data source. Do NOT rely on conversation context.

```bash
ls -1 {project}/.qa/reports/ 2>/dev/null | sort | tail -20
```

Sort report filenames to find the most recent (filenames contain ISO timestamps, so lexicographic sort gives chronological order).

Read the most recent report file:

```bash
cat {project}/.qa/reports/{most-recent-file}
```

**If no reports exist:**

```
No QA reports found in {project}/.qa/reports/

Run tests first to generate reports:
  /qa:run --project {project} --url <your-url>
  /qa:monitor --project {project} --url <your-url>

Then run /qa:report to see results.
```

Stop here. Do not proceed without a report file.

**Fallback:** If `.qa/reports/` is empty AND a test was run in this session (results are in conversation context), use conversation context as a fallback. This fallback is exceptional — file-based reading is the expected path.

### Step 3: Present Latest Run Results

Parse the most recent report and present the "what just happened?" section.

Extract from the report file:
- Run timestamp (from filename or frontmatter `run-at`)
- Overall result (Pass / Fail / Pass with Concerns)
- Scenario results (passed, failed, regressions)
- All findings grouped by confidence level

Present in this order:
1. Executive summary (counts, overall result)
2. Definite Issues (High confidence) — need immediate attention
3. Likely Issues (Medium confidence) — need investigation
4. Possible Issues (Low confidence) — worth noting

This section should be immediately useful without scrolling. The user gets the answer to "what just happened?" before any trend context.

### Step 4: Trend Analysis from HISTORY.md

Read the history file:

```bash
cat {project}/.qa/HISTORY.md 2>/dev/null
```

**If HISTORY.md does not exist or has fewer than 2 run entries:**

```
Not enough history for trend analysis. Run more tests to build trend data.
(Need at least 2 runs in .qa/HISTORY.md)
```

Skip trend analysis and continue to Step 6.

**If history exists with 2+ entries:**

Parse run entries within the `--since` window. Each entry has the format:

```
## Run: {ISO-timestamp}
- **URL**: {url}
- **Tag**: {tag}
- **Results**: {passed} passed, {failed} failed, {concerns} concerns
- **Regressions**: {none | N regressions: scenario-name-1, scenario-name-2}
- **Duration**: {duration}
```

From the history entries in the window, extract:
- Total runs in window
- Per-run pass/fail/concern counts
- Regression events (which scenarios, on which dates)
- Scenarios that appear in failures across multiple runs

#### Compute trend metrics:

**Stability score:** What percentage of scenarios passed consistently across all runs in the window?

```
stability = (scenarios that passed in ALL runs in window) / (total unique scenarios seen) × 100%
```

**Recurring failures:** Scenarios that failed in 2 or more runs in the window. List with:
- Scenario name
- Failure count out of total runs in window
- Dates when failure occurred

**Critical recurring failures:** Scenarios that failed in 3 or more runs in the window. These are flagged with elevated prominence:

```
RECURRING: {scenario name} — failed {N} of last {M} runs (first seen: {date})
```

**Regression timeline:** When did previously-passing scenarios start failing? List regressions with the date they first appeared in the window.

**Improvement trends:** Scenarios that appeared in failures in older runs but passed in the most recent runs. These are positive signals.

### Step 5: GitHub Issue Creation (if --create-issues)

Only execute this step when `--create-issues` flag is present.

**For each High confidence (Definite) finding from the latest report:**

1. Check if a similar open issue already exists:

```bash
gh issue list --search "{scenario name or brief description}" --state open
```

2. If an open issue is found — skip. Report it as "skipped (duplicate found)".

3. If no open issue found — create one:

```bash
gh issue create \
  --title "QA: {brief issue description}" \
  --body "{detailed finding with:
  - Evidence from the report (console errors, network failures, screenshots referenced)
  - Trend data if the issue is recurring (e.g., 'Failed 3 of last 5 runs — first seen 2026-03-05')
  - Spec and scenario reference
  - Suggested next steps}" \
  --label "bug,qa-agent"
```

4. Record the created issue URL.

**For Medium confidence findings:** List them but do NOT auto-create issues. Output a suggestion:

```
Medium confidence findings (review manually before creating issues):
- {finding}: {confidence rationale}
```

**If `gh` CLI is not authenticated or not installed:**

```
GitHub issue creation skipped: gh CLI not authenticated.
Run `gh auth login` to authenticate, then re-run with --create-issues.
```

Continue with report output — don't fail on auth error.

**Report which issues were created and which were skipped:**

```
Issues created: {N}
  - #{issue-number}: {title} ({url})

Issues skipped (duplicates found): {N}
  - "{scenario name}" — matched existing issue #{number}
```

### Step 6: Compile and Output Report

#### Markdown format (default)

```markdown
# QA Report

**Generated:** {timestamp}
**Project:** {project path}
**Data source:** .qa/reports/{latest report filename}

---

## Latest Run

**Run:** {ISO-timestamp}
**Result:** {Pass | Fail | Pass with Concerns}
**URL tested:** {url from report}

### Executive Summary

| Metric | Count |
|--------|-------|
| Scenarios Run | {N} |
| Passed | {N} |
| Failed | {N} |
| Pass with Concerns | {N} |

**Definite Issues (High confidence):** {N}
**Likely Issues (Medium confidence):** {N}
**Possible Issues (Low confidence):** {N}

### Definite Issues (High Confidence)

{For each high-confidence issue:}

#### {Issue title}

- **Confidence:** High — {rationale}
- **Severity:** {Critical | Major | Minor}
- **Spec/Scenario:** {spec name} > {scenario name}
- **Description:** {what happened}
- **Evidence:** {console errors, network failures, screenshots}
- **Suggested next steps:** {what to fix}

---

### Likely Issues (Medium Confidence)

{For each medium-confidence issue — same structure}

---

### Possible Issues (Low Confidence)

{For each low-confidence issue — same structure}

---

### Passed Scenarios

{List of scenarios that passed}

---

## Trend Analysis (last {N} runs / {N} days)

**Stability score:** {X}% of scenarios passed consistently across all {N} runs in window

### Recurring Failures

{If any scenario failed in 3+ runs:}

> **RECURRING: {scenario name}** — failed {N} of last {M} runs (first seen: {date})
> {brief failure description}

{If any scenario failed in 2 runs:}

**{scenario name}** — failed {N} of last {M} runs ({dates})

{If no recurring failures:}
No recurring failures in this window.

### Regression Timeline

{For each regression in window:}
- **{date}:** {scenario name} started failing (was previously passing)

{If no regressions:}
No new regressions detected in this window.

### Improvements

{For each scenario that was failing but now passes:}
- **{scenario name}** — was failing in older runs, now passing

{If none:}
No improvement trends detected.

---

## Recommendations

### Immediate (from latest run)

{High confidence issues → fix now}
{Medium confidence issues → investigate}

### Recurring Issues (escalated)

{Issues that appear in 3+ runs → prioritize}

### Coverage Gaps

{What wasn't tested, inferred from spec/scenario names in history}

---

## Issues Created

{If --create-issues was used:}

**Created:** {N} issues
{List with URLs}

**Skipped (duplicates):** {N}
{List with existing issue numbers}

{If --create-issues was not used:}
Run with `--create-issues` to create GitHub issues from critical findings.
```

#### JSON format

```json
{
  "generated_at": "{ISO-timestamp}",
  "project": "{project path}",
  "data_source": ".qa/reports/{filename}",
  "latest_run": {
    "run_at": "{ISO-timestamp}",
    "url": "{url}",
    "result": "Pass | Fail | Pass with Concerns",
    "summary": {
      "total_scenarios": 0,
      "passed": 0,
      "failed": 0,
      "concerns": 0
    },
    "issues": [
      {
        "confidence": "high | medium | low",
        "confidence_rationale": "{rationale}",
        "severity": "Critical | Major | Minor",
        "description": "{description}",
        "spec": "{spec} > {scenario}",
        "evidence": "{evidence}"
      }
    ]
  },
  "trends": {
    "window": "{N} runs / {N} days",
    "stability_score": 0.0,
    "total_runs_in_window": 0,
    "recurring_failures": [
      {
        "scenario": "{name}",
        "failure_count": 0,
        "total_runs": 0,
        "first_seen": "{date}",
        "severity": "recurring | critical-recurring"
      }
    ],
    "regression_timeline": [
      {
        "scenario": "{name}",
        "first_failed": "{date}",
        "previously_passing": true
      }
    ],
    "improvements": [
      {
        "scenario": "{name}",
        "was_failing": true,
        "now_passing": true
      }
    ]
  },
  "recommendations": {
    "immediate": ["{recommendation}"],
    "recurring": ["{escalated recommendation}"],
    "coverage_gaps": ["{gap}"]
  },
  "issues_created": [
    {
      "title": "{title}",
      "number": 0,
      "url": "{url}",
      "status": "created | skipped-duplicate"
    }
  ]
}
```

## Report Structure Design

The report is structured to answer two questions in order:

1. **"What just happened?"** — Latest Run section comes first. This is always immediately useful, regardless of how much history exists.

2. **"Is this a pattern?"** — Trend Analysis section follows. This adds longitudinal context once the immediate picture is clear.

Recommendations are last because they synthesize both views.

## Cross-Session Capability

This skill reads entirely from disk. It does not need prior conversation context. You can run `/qa:report` in a brand new session and get a complete picture of what was tested and what was found — as long as `.qa/reports/` has report files.

This means:
- Reports accumulate value across sessions
- Different engineers can run `/qa:report` and see the same data
- CI/CD pipelines can call this skill to generate reports without session state

## Files Read

| File | Purpose |
|------|---------|
| `.qa/reports/{most-recent}.md` | Latest run results — primary data source |
| `.qa/HISTORY.md` | Run log across all sessions — used for trend analysis |

## Example: Recurring Failure Detection

If `.qa/HISTORY.md` shows:

```
## Run: 2026-03-09T10:00:00Z
- Regressions: 1 regression: Login flow

## Run: 2026-03-10T10:00:00Z
- Regressions: 1 regression: Login flow

## Run: 2026-03-11T10:00:00Z
- Regressions: 1 regression: Login flow
```

The report will flag:

```
> RECURRING: Login flow — failed 3 of last 3 runs (first seen: 2026-03-09)
```

And escalate this in the Recommendations section as a priority fix.
