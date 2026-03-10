# Phase 1: Cleanup & Foundation - Context

**Gathered:** 2026-03-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Clean up the QA agent plugin: remove stale references, consolidate to skills (delete commands), bundle Playwright MCP server, update plugin manifest, and establish dev-to-system deployment via symlink. No new capabilities — pure restructuring.

</domain>

<decisions>
## Implementation Decisions

### Commands removal
- Delete the `commands/` directory entirely — skills fully replace them
- Delete the old standalone commands at `~/.claude/commands/qa/` (5 files: init, gen, run, check, report)
- Update README to remove command invocation references, document only skill patterns (/qa:run etc.)
- Scan examples/ directory and update any command references to skill patterns
- Full codebase grep for any remaining command references

### Plugin deployment & settings
- Symlink `~/.claude/plugins/cache/local/qa-agent/0.1.0/` to this dev repo so changes are immediately live
- Claude's discretion on plugin.json settings defaults (based on what the agent actually uses)
- Claude's discretion on version bump strategy (0.2.0-alpha now vs 0.2.0 at milestone end)
- Claude's discretion on whether .local.md per-project config pattern is needed for Phase 1

### Playwright MCP bundling
- Bundle .mcp.json in the plugin's .claude-plugin/ directory — plugin is self-contained
- Use npx to run @playwright/mcp (downloads on demand, no local install required)
- Headless mode by default — more stable, no window interference issues
- All Playwright tools (console, network, screenshots, DOM snapshots, JS execution) work identically in headless mode

### Stale reference cleanup
- Clean-slate rewrite of qa-tester.md agent file with correct tool list: `mcp__playwright__*`, `Read`, `Write`, `Bash`, `Glob`, `Grep`
- Remove all `mcp__MCP_DOCKER__browser_*` references
- Remove all `Task` tool references (replace with `Agent` where applicable)
- Claude's discretion on whether to refine methodology or just fix tools/references
- Full scan of all files: skills, agent, README, LEARNINGS.md, examples
- Remove root-level SPEC files (SPEC-advanced-accessibility-agent.md, SPEC-cli-testing-support.md) — accessibility agent is a separate project, CLI testing is out of scope

### Claude's Discretion
- Plugin.json settings defaults and what to expose
- Version bump timing and strategy
- Level of agent methodology refinement during rewrite
- Whether .local.md pattern is needed now or deferred

</decisions>

<specifics>
## Specific Ideas

- Symlink approach for dev: changes in this repo are immediately available system-wide without reinstalling
- The installed plugin at `~/.claude/plugins/cache/local/qa-agent/0.1.0/` is currently a frozen copy — symlink replaces it
- Old `~/.claude/commands/qa/` commands exist independently of the plugin and must be cleaned up separately
- Headless was chosen specifically because headed mode caused stability issues when the browser window was accidentally interacted with

</specifics>

<deferred>
## Deferred Ideas

- Per-project config via .local.md pattern — may be needed in Phase 2 (Agent Architecture) when settings become more complex
- Version in installed_plugins.json will need updating when version bumps — future concern when distribution model solidifies

</deferred>

---

*Phase: 01-cleanup-foundation*
*Context gathered: 2026-03-10*
