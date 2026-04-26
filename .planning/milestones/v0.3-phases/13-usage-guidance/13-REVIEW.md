---
phase: 13-usage-guidance
reviewed: 2026-04-20T12:00:00Z
depth: standard
files_reviewed: 4
files_reviewed_list:
  - docs/ORCHESTRATION.md
  - skills/guide/SKILL.md
  - README.md
  - CLAUDE.md
findings:
  critical: 0
  warning: 2
  info: 2
  total: 4
status: issues_found
---

# Phase 13: Code Review Report

**Reviewed:** 2026-04-20T12:00:00Z
**Depth:** standard
**Files Reviewed:** 4
**Status:** issues_found

## Summary

Reviewed the four files added/modified in Phase 13 (Usage Guidance): the orchestration reference document, the `/qa:guide` skill, and updates to README.md and CLAUDE.md. Overall quality is high — the documentation is well-structured, internally consistent across files, and provides clear machine-parseable formats for orchestrating agents. Two warnings found related to the guide skill's design (unreachable path resolution and overly permissive tool grants). Two info items on minor inconsistencies.

## Warnings

### WR-01: Unreachable Shell Path Resolution in Guide Skill

**File:** `skills/guide/SKILL.md:40`
**Issue:** The skill instructs the agent to locate ORCHESTRATION.md using shell self-referencing:
```bash
cat "$(dirname "$0")/../../docs/ORCHESTRATION.md" 2>/dev/null
```
In a Claude Code skill context, `$0` does not resolve to the skill file path. Skills are Markdown documents interpreted by the AI agent, not executed shell scripts. When the agent runs this command via the Bash tool, `$0` will resolve to the shell binary (e.g., `/bin/zsh`), making this path incorrect. The surrounding prose (line 42-43) correctly describes the intent ("Locate `docs/ORCHESTRATION.md` relative to the skill's own directory"), but the shell command itself will fail silently (due to `2>/dev/null`) and the agent will skip reading the orchestration reference.
**Fix:** Replace with an instruction that the agent can actually execute. Since the plugin root is known relative to the skill invocation context, use a direct path or instruct the agent to locate the file:
```markdown
Read the orchestration reference from the plugin's own directory:

```bash
# The plugin root is the parent of the skills/ directory
# Find it by locating this plugin's docs/ORCHESTRATION.md
find / -path "*/knz-qa-agent/docs/ORCHESTRATION.md" -print -quit 2>/dev/null
```

Or more practically, instruct the agent to use the Read tool directly on the known relative path from the plugin installation:
```markdown
Use the Read tool to load `docs/ORCHESTRATION.md` from the QA plugin's installation directory.
```

### WR-02: Write Tool Granted to Read-Only Skill

**File:** `skills/guide/SKILL.md:5`
**Issue:** The frontmatter grants `Write` in `allowed-tools: Read, Write, Bash, Grep, Glob`, but the skill explicitly states at line 130: "Do not run any tests, generate any files, or spawn any agents. This skill only reads and recommends." Granting `Write` to a skill documented as read-only violates the principle of least privilege. If the AI agent misinterprets guidance or hallucinates a step, the Write tool would be available to create or overwrite files.
**Fix:** Remove `Write` from the allowed-tools list:
```yaml
allowed-tools: Read, Bash, Grep, Glob
```

## Info

### IN-01: Misleading "File-read-only" Label in ORCHESTRATION.md

**File:** `docs/ORCHESTRATION.md:24`
**Issue:** The label "File-read-only skills" groups `init`, `report`, and `guide` together, but `init` creates files (it scaffolds `.qa/`) and `report --create-issues` invokes `gh` to create GitHub issues. The label only means "does not spawn a browser agent," but this could confuse an orchestrating agent into thinking these skills have no side effects.
**Fix:** Consider rewording to "Non-browser skills" or adding a clarifying note:
```markdown
**Non-browser skills** (`init`, `report`, `guide`) do not spawn the `qa-tester` browser agent. Note: `init` writes files and `report --create-issues` creates GitHub issues.
```

### IN-02: Shell Commands in Guide Skill Lack Path Quoting

**File:** `skills/guide/SKILL.md:53-78`
**Issue:** All shell commands interpolate `{project}` without quoting:
```bash
ls {project}/.qa/ 2>/dev/null
ls {project}/.qa/*.md 2>/dev/null | wc -l
```
If the project path contains spaces (e.g., `~/Projects/My App`), these commands would break. Since this is instructional text for an AI agent (which should know to quote paths), the risk is low but worth noting for consistency with defensive coding practices.
**Fix:** Add quotes around path interpolations:
```bash
ls "{project}/.qa/" 2>/dev/null
ls "{project}/.qa/"*.md 2>/dev/null | wc -l
```

---

_Reviewed: 2026-04-20T12:00:00Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
