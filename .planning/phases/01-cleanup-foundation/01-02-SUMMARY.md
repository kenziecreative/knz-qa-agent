---
phase: 01-cleanup-foundation
plan: 02
subsystem: documentation-and-devenv
tags: [readme, learnings, symlink, docs, devenv, cleanup]

requires:
  - 01-01 (commands/ deleted, skills confirmed as sole invocation pattern)

provides:
  - README with accurate skills-only structure and .mcp.json in Project Structure
  - LEARNINGS with no stale command references, updated evolution note
  - dev symlink: ~/.claude/plugins/cache/local/qa-agent/0.1.0 -> dev repo
  - no orphaned global commands at ~/.claude/commands/qa/

affects:
  - all future dev work (symlink means edits are immediately live)
  - 02+ phases (README accurately describes what exists)

tech-stack:
  added: []
  patterns:
    - "dev symlink pattern: plugin cache points to dev repo for live editing"

key-files:
  created: []
  modified:
    - README.md
    - LEARNINGS.md
  external-changes:
    - "~/.claude/commands/qa/ (deleted)"
    - "~/.claude/plugins/cache/local/qa-agent/0.1.0 (replaced with symlink)"

key-decisions:
  - "Dev symlink over reinstall: symlink to dev repo means zero friction for iterating on plugin"
  - "Historical command reference preserved in LEARNINGS evolution note (factual, not structural)"

patterns-established:
  - "Plugin cache symlink pattern for development workflow"

duration: "~2 minutes"
completed: "2026-03-10"
---

# Phase 1 Plan 2: Cleanup Foundation - Documentation and Dev Environment Summary

**One-liner:** Updated README/LEARNINGS to remove commands/ refs and add .mcp.json to project structure; deleted orphaned global commands; replaced frozen plugin copy with dev symlink for live editing.

## Performance

- Duration: ~2 minutes
- Tasks: 2/2 completed
- Commits: 1 task commit (Task 2 was external filesystem changes only)

## Accomplishments

1. README Project Structure: removed `commands/` directory, added `.claude-plugin/.mcp.json`
2. README Installation: updated to explain Playwright MCP is bundled via `.mcp.json`
3. README Requirements: updated to reference `.claude-plugin/.mcp.json` directly
4. README version history: updated 0.2.0 and 0.1.0 entries to use skills-only language
5. LEARNINGS plugin pattern tree: removed `commands/`, added `.claude-plugin/.mcp.json`
6. LEARNINGS evolution note: updated from "migrated to plugin" to "commands fully removed"
7. LEARNINGS section title: "Skill/Command Structure" renamed to "Skill Structure"
8. Deleted `~/.claude/commands/qa/` (5 files: check.md, gen.md, init.md, report.md, run.md)
9. Created symlink: `~/.claude/plugins/cache/local/qa-agent/0.1.0` -> dev repo

## Task Commits

| Task | Name | Commit | Key Files |
|------|------|--------|-----------|
| 1 | Update README and LEARNINGS | 110fa88 | README.md, LEARNINGS.md |
| 2 | Delete old global commands and set up dev symlink | (external) | ~/.claude/commands/qa/, ~/.claude/plugins/cache/local/qa-agent/0.1.0 |

## Files Created/Modified

**Modified:**
- `README.md` — Removed commands/ from Project Structure, added .mcp.json, updated Installation/Requirements/version history
- `LEARNINGS.md` — Renamed Skill/Command Structure section, updated plugin pattern tree, updated evolution note

**External filesystem changes (not tracked in repo):**
- `~/.claude/commands/qa/` — Deleted (5 command files)
- `~/.claude/plugins/cache/local/qa-agent/0.1.0` — Replaced directory with symlink to dev repo

## Decisions Made

1. **Dev symlink pattern** — The plugin cache entry (`~/.claude/plugins/cache/local/qa-agent/0.1.0`) is now a symlink to the dev repo. This means any edit in the dev repo is immediately reflected when Claude Code loads the plugin — no reinstall or copy step needed.

2. **Historical reference preserved** — LEARNINGS.md evolution note mentions `~/.claude/commands/qa/` as where commands used to live. This is factual history, not a structural claim about what exists today. The verification grep (`grep "commands/" LEARNINGS.md`) picks this up but the context makes clear it's historical.

## Deviations from Plan

**Minor:** Task 2 produced no git commit because all changes were external filesystem operations (deleting `~/.claude/commands/qa/` and creating a symlink). These changes cannot be committed to the repo. Documented in SUMMARY instead.

## Issues

None.

## Next Phase Readiness

Ready for Phase 2 (or remaining phases in Phase 1 if any). With:
- Documentation accurately reflecting current structure
- Dev symlink live (edits immediately reflected)
- No orphaned global command files creating confusion
- Clean LEARNINGS with correct tool references

The dev workflow is now efficient: edit any file in the repo, Claude Code sees the change immediately on next load.
