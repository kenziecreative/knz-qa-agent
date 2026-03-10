---
name: session-start-qa-context
description: Load last QA results and known issues at session start
---

# Session Start Hook: Load QA Context

When this hook fires (at the start of a Claude Code session), follow these steps exactly.

## Step 1: Check for QA Setup

First, check if this project has QA configured:

```bash
ls .qa/ 2>/dev/null
```

If `.qa/` directory does not exist → **exit silently**. This project has not been initialized for QA. Do not show any output.

If `.qa/` exists, continue to Step 2.

## Step 2: Load Last QA Report

Find the most recent QA report:

```bash
ls -t .qa/reports/ 2>/dev/null | head -1
```

If no reports exist (directory empty or missing) → skip this section silently.

If a report exists, read its full content:

```bash
cat ".qa/reports/$(ls -t .qa/reports/ 2>/dev/null | head -1)"
```

### Present the Report

Parse the report's frontmatter or filename to determine the date and result. Then present the information in this format.

**If the last report was a clean pass with no concerns:**

```
Last QA Run: [date] — All clear, no issues.
```

**If the last report had findings (Fail or Pass with Concerns):**

```
Last QA Run: [date from report filename or frontmatter]
Result: [Pass / Fail / Pass with Concerns]

[Full report content]
```

Show the full report content — do not summarize or truncate it. The user needs full context to make informed decisions about the current session's work.

## Step 3: Load Known Issues from Memory

Check for persisted QA memory:

```bash
ls .qa/memory/ 2>/dev/null
```

If no memory files exist, skip this section silently.

If memory files exist, read each one (excluding the `gsd-suggested` marker file):

```bash
for f in .qa/memory/*.md 2>/dev/null; do cat "$f"; done
```

Extract and present known issues or patterns in this format:

```
Known QA Issues:
- [issue or pattern from memory file 1]
- [issue or pattern from memory file 2]
```

Only include items that represent active known issues or recurring patterns. Skip expired entries (those with `updated` dates older than 30 days).

## Step 4: Presentation Guidelines

- If both report and memory are clean → one brief line is enough:
  ```
  Last QA Run: [date] — All clear, no issues.
  ```
- If findings exist → show full report, then known issues below it
- Keep framing factual and informative, not alarming
- Present this context at the start of the session so it informs the work ahead

## Step 5: Silent Exit Conditions

Exit without any output if:
- `.qa/` directory does not exist (project not initialized for QA)
- `.qa/reports/` is empty and `.qa/memory/` is empty or missing

The goal is zero noise for projects that don't use this QA agent.
