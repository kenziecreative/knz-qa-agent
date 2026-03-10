---
phase: 01-cleanup-foundation
plan: 01
subsystem: plugin-structure
tags: [playwright, mcp, agent, skills, plugin-manifest, cleanup]

requires: []
provides:
  - clean qa-tester agent with correct Playwright MCP tools
  - .mcp.json bundling Playwright MCP in headless mode
  - updated plugin.json with version and settings defaults
  - skills with correct allowed-tools references
affects:
  - 01-02 (mode detection will depend on correct tools being in agent)
  - all future phases using qa-tester agent

tech-stack:
  added:
    - "@playwright/mcp (via npx, no local install)"
  patterns:
    - "npx on-demand MCP server pattern (no local install required)"
    - "allowed-tools frontmatter in skill files"

key-files:
  created:
    - .claude-plugin/.mcp.json
  modified:
    - agents/qa-tester.md
    - skills/qa-run.md
    - .claude-plugin/plugin.json
  deleted:
    - commands/check.md
    - commands/gen.md
    - commands/init.md
    - commands/report.md
    - commands/run.md
    - SPEC-advanced-accessibility-agent.md (was untracked)
    - SPEC-cli-testing-support.md (was untracked)

key-decisions:
  - "npx for Playwright MCP: downloads on demand, no local install, always latest"
  - "headless: true default in .mcp.json for stability"
  - "version 0.2.0-alpha: signals start of v0.2 work, not yet stable"
  - "settings.defaults exposes: headless, viewport, reportFormat, slowThreshold, screenshotsEnabled"

patterns-established:
  - "Skills use allowed-tools frontmatter to declare tool access"
  - "Agent uses mcp__playwright__* (wildcard) to cover all Playwright MCP tools"

duration: "2 minutes"
completed: "2026-03-10"
---

# Phase 1 Plan 1: Cleanup Foundation - Stale Reference Removal Summary

**One-liner:** Replaced mcp__MCP_DOCKER tools with mcp__playwright__* across agent/skills, bundled Playwright MCP via npx headless .mcp.json, and bumped plugin to 0.2.0-alpha with settings defaults.

## Performance

- Duration: ~2 minutes
- Tasks: 2/2 completed
- Commits: 2 task commits

## Accomplishments

1. Deleted the obsolete `commands/` directory (5 files) — fully replaced by `skills/`
2. Deleted out-of-scope SPEC files (accessibility agent, CLI testing)
3. Rewrote `agents/qa-tester.md` with exact tool list: `mcp__playwright__*`, `Read`, `Write`, `Bash`, `Glob`, `Grep`
4. Updated `skills/qa-run.md` allowed-tools: `Task` -> `Agent`, `mcp__MCP_DOCKER__browser_*` -> `mcp__playwright__*`
5. Created `.claude-plugin/.mcp.json` with Playwright MCP server (npx, headless)
6. Updated `.claude-plugin/plugin.json` to version `0.2.0-alpha` with settings defaults

## Task Commits

| Task | Name | Commit | Key Files |
|------|------|--------|-----------|
| 1 | Delete stale files and rewrite agent | c78e307 | agents/qa-tester.md, commands/ (deleted) |
| 2 | Update skills, create .mcp.json, update plugin.json | 0f2d5f5 | skills/qa-run.md, .claude-plugin/.mcp.json, .claude-plugin/plugin.json |

## Files Created/Modified

**Created:**
- `.claude-plugin/.mcp.json` — Playwright MCP server config (npx, headless)

**Modified:**
- `agents/qa-tester.md` — Full rewrite with correct tool list, removed stale refs
- `skills/qa-run.md` — Added allowed-tools frontmatter with correct tools
- `.claude-plugin/plugin.json` — Version bump to 0.2.0-alpha, added settings.defaults

**Deleted:**
- `commands/` directory (check.md, gen.md, init.md, report.md, run.md)
- `SPEC-advanced-accessibility-agent.md` (untracked, removed from disk)
- `SPEC-cli-testing-support.md` (untracked, removed from disk)

## Decisions Made

1. **npx for Playwright MCP** — Uses `npx @playwright/mcp@latest` so no local install is required. Downloads on demand, always uses latest version. Tradeoff: slight startup delay on first run.

2. **Headless by default** — Set `"--headless"` in .mcp.json args. More stable in automated testing contexts; can be toggled via plugin settings if needed.

3. **Version 0.2.0-alpha** — Signals start of v0.2 work. Not yet stable/production-ready. Will graduate to 0.2.0 when all 6 phases complete.

4. **Settings defaults exposed** — `headless`, `viewport` (1280x720 desktop baseline), `reportFormat` (markdown), `slowThreshold` (1000ms), `screenshotsEnabled` (true). These reflect what the agent actually uses.

5. **Other 4 skills have no allowed-tools** — `qa-check.md`, `qa-gen.md`, `qa-init.md`, `qa-report.md` had no stale tool references in frontmatter. Left without `allowed-tools` (they inherit Claude Code defaults).

## Deviations from Plan

None — plan executed exactly as written. The 4 non-run skills had no stale frontmatter tool references, consistent with plan task 2 ("If they don't have `allowed-tools` frontmatter... leave them alone").

## Issues

None.

## Next Phase Readiness

Ready for Plan 01-02 (mode detection). The agent now has:
- Correct Playwright MCP wildcard (`mcp__playwright__*`)
- Write/Bash/Grep tools for report writing and file operations
- No stale Docker-based tool references

The .mcp.json provides the Playwright MCP server that all subsequent phases depend on.
