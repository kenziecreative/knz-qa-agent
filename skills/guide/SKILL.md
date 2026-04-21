---
name: qa:guide
description: Inspect project QA state and recommend the next action
argument-hint: "--project <path>"
allowed-tools: Read, Write, Bash, Grep, Glob
---

# /qa:guide

Read the current project's QA state from disk and return context-aware guidance on which QA command to use next — works without any prior conversation context.

## Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--project <path>` | Yes | — | Path to project containing `.qa/` directory (or where one should be created) |

## Usage

```
/qa:guide --project ~/Projects/my-app
/qa:guide --project .
```

## Purpose

Reads from `.qa/` state files and `docs/ORCHESTRATION.md` to produce a short, actionable recommendation. Useful when an orchestrating agent or developer is unsure which QA command to run next.

## Execution Flow

### Step 1: Parse Arguments

Extract `--project` path from user input. Default to current directory if not specified.

### Step 2: Read ORCHESTRATION.md

Read the orchestration reference to ground recommendations in documented guidance (per D-07):

```bash
cat "$(dirname "$0")/../../docs/ORCHESTRATION.md" 2>/dev/null
```

Locate `docs/ORCHESTRATION.md` relative to the skill's own directory (the plugin root, not the `--project` path). Use the "Decision Points for Orchestrating Agents" and "Recommended Flow" sections to inform which skill to recommend based on the project state discovered in Step 3.

This is the reference document — read it before inspecting project state so the recommendations align with documented guidance.

### Step 3: Inspect Project State

Run all five state checks. Use `2>/dev/null` on every command to silence errors when files don't exist.

**Check 1: .qa/ directory existence**
```bash
ls {project}/.qa/ 2>/dev/null
```
If not found → recommend `/qa:init {project}` and stop. No further checks needed.

**Check 2: Spec file count**
```bash
ls {project}/.qa/*.md 2>/dev/null | wc -l
```
If 0 specs (but `.qa/` exists) → recommend `/qa:gen`.

**Check 3: Report count**
```bash
ls {project}/.qa/reports/*.md 2>/dev/null | wc -l
```
If specs exist but 0 reports → recommend `/qa:run`.

**Check 4: HISTORY.md line count**
```bash
wc -l < {project}/.qa/HISTORY.md 2>/dev/null
```
If HISTORY.md has content → note trend data is available, mention `/qa:report` for analysis.

**Check 5: Monitor alert presence**
```bash
cat {project}/.qa/.monitor-alert 2>/dev/null
```
If alert file exists → surface the alert prominently; recommend `/qa:run` to investigate or `/qa:report` for full trend context.

### Step 4: Compose Guidance

Output format is a short narrative with 2-4 bullet recommendations. NOT a checklist or decision tree.

**Priority ordering (first applicable wins as primary recommendation):**
1. Monitor alert present → lead with the alert content, recommend investigation
2. No `.qa/` directory → recommend `/qa:init`
3. No spec files → recommend `/qa:gen`
4. Specs exist but no reports → recommend `/qa:run`
5. Reports exist → summarize state, suggest `/qa:report` for trends or `/qa:monitor` for ongoing checks

**Output style — terse and actionable:**

Example when `.qa/` doesn't exist:
```
QA not initialized for this project.

- Run `/qa:init {project}` to scaffold the .qa/ directory
- Then `/qa:gen` to create test specs from your feature descriptions
```

Example when specs exist but no reports:
```
QA initialized with {N} spec(s), but no test runs yet.

- Run `/qa:run --project {project} --url <your-url>` to execute your specs
- After running, `/qa:report --project {project}` will show results and trends
```

Example when reports exist:
```
QA active: {N} specs, {M} reports, trend data available.

- Run `/qa:report --project {project}` to review trends and recurring issues
- Run `/qa:run --project {project}` to re-test with latest code changes
- Consider `/qa:monitor --project {project} --url <url>` for continuous regression detection
```

Example with monitor alert:
```
MONITORING ALERT: {alert content}

- Run `/qa:run --project {project}` to investigate the regression
- Run `/qa:report --project {project}` for full trend context
- The alert was written by /qa:monitor — a previously passing scenario is now failing
```

### Step 5: Stop

Output the guidance and stop. Do not run any tests, generate any files, or spawn any agents. This skill only reads and recommends.

## Cross-Session Capability

This skill reads entirely from disk. It does not need prior conversation context. You can run `/qa:guide` in a brand new session and get an accurate recommendation based on the current state of `.qa/`.

This means:
- An orchestrating agent entering a project for the first time can call `/qa:guide` to understand what QA work has been done
- Different engineers get consistent guidance based on the same on-disk state
- No conversation history is needed

## Files Read

| File | Purpose |
|------|---------|
| `docs/ORCHESTRATION.md` | Reference document for grounding recommendations in documented guidance |
| `{project}/.qa/` | Directory existence check — QA initialized? |
| `{project}/.qa/*.md` | Spec file count — tests defined? |
| `{project}/.qa/reports/*.md` | Report count — tests have been run? |
| `{project}/.qa/HISTORY.md` | History line count — trend data available? |
| `{project}/.qa/.monitor-alert` | Alert file — regression detected by monitor? |
