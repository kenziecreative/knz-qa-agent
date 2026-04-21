# QA Agent Plugin

A Claude Code plugin that tests web applications like a human would — with intuition, pattern recognition, and the ability to escalate when something isn't working.

## What This Is

This is the source code for the `qa-agent` Claude Code plugin. It provides skills, an agent, and hooks that enable AI-powered QA testing of web applications. When installed as a plugin in a user's project, it adds `/qa:run`, `/qa:check`, `/qa:gen`, `/qa:init`, `/qa:monitor`, `/qa:report`, and `/qa:guide` commands.

## Project Structure

```
.claude-plugin/
├── plugin.json          # Plugin metadata, version, default settings
└── .mcp.json            # MCP server config (currently empty — no MCP dependencies)
agents/
└── qa-tester.md         # The core QA agent — system prompt + methodology
skills/
├── run/SKILL.md         # /qa:run — execute tests from specs or ad-hoc
├── check/SKILL.md       # /qa:check — verify phase deliverables (GSD integration)
├── gen/SKILL.md         # /qa:gen — generate spec from description or docs
├── init/SKILL.md        # /qa:init — scaffold .qa directory in a project
├── monitor/SKILL.md     # /qa:monitor — recurring smoke tests via /loop
├── report/SKILL.md      # /qa:report — cross-session QA reports with trends
└── guide/SKILL.md       # /qa:guide — context-aware usage guidance
hooks/
├── hooks.json           # Hook definitions (SessionStart, Stop)
├── session-start.md     # Loads QA memory on session start
└── stop.md              # Persists QA observations on session end
docs/
└── ORCHESTRATION.md     # AI agent orchestration reference — how to invoke and integrate the plugin
examples/                # Example spec files
LEARNINGS.md             # Architecture decisions and patterns for this project
README.md                # User-facing documentation
```

## How It Works

### Browser Automation

The agent uses `playwright-cli` (a standalone CLI tool) for all browser interaction, invoked via the Bash tool. There are no MCP server dependencies.

Key points:
- The agent must `playwright-cli open [url]` to start a session and `playwright-cli close` when done
- Element refs come from `playwright-cli snapshot` output
- The full command reference is documented in `agents/qa-tester.md` under "Browser Control via playwright-cli"

### Agent Architecture

The `qa-tester` agent (`agents/qa-tester.md`) is the workhorse. It contains:
- Testing methodology (screenshot/console/network discipline)
- Pattern recognition and 3-failure escalation rules
- Multi-viewport testing (mobile/tablet/desktop)
- Multi-persona testing (role-based auth flows)
- Form intelligence (validation, submission states, autofill)
- SPA awareness (routing, history, hydration)
- Structured accessibility testing (Tier 1 baseline + Tier 2 deep checks)
- Network intelligence (timing, error correlation, mixed content)
- Spec v2 features (dependencies, data-driven scenarios, tags, environments)
- Visual & UX design verification (Tier 2 -- design reference comparison, UX states, layout integrity, performance/responsive, triggered by `## Visual Focus` in specs)
- Cross-browser engine testing (Chromium, WebKit, Firefox via `## Browsers` in specs)

Skills orchestrate the agent — they parse user arguments, resolve specs/environments/tags, and spawn the agent with a structured prompt.

### Data Flow

When used in a target project:
```
User project/.qa/
├── *.md              # Spec files (test definitions)
├── reports/          # Test reports (generated)
│   └── HISTORY.md    # Running log of all QA activity
└── memory/           # QA-specific memory (patterns, known issues, baselines)
```

## Development Guidelines

### Editing the Agent

`agents/qa-tester.md` is the most important file. When modifying:
- The frontmatter `tools:` list controls what tools the agent can access
- Methodology sections teach the agent *how to think*, not just what to do
- Confidence levels (High/Medium/Low) are used throughout — maintain consistency
- The accessibility section follows a specific tier model (Tier 1 always runs, Tier 2 is opt-in via spec)

### Editing Skills

Each skill in `skills/*/SKILL.md` has:
- YAML frontmatter with `allowed-tools` — this restricts what the skill can use
- Argument parsing logic — how user input maps to behavior
- Agent invocation template — the prompt sent to `qa-tester` when spawning it

Skills that spawn the agent: `run`, `check`, `gen`, `monitor`. Skills that don't: `init`, `report`, `guide`.

### Adding New Capabilities

If adding a new playwright-cli capability to the agent:
1. Add the command reference to the "Browser Control via playwright-cli" section in `agents/qa-tester.md`
2. If the skill's invocation template needs to reference it, update `skills/run/SKILL.md`
3. No MCP or tool grant changes needed — everything goes through Bash

### Things to Preserve

- **3-failure escalation rule** — the agent stops retrying after 3 failures and escalates. This is intentional and important.
- **Spec skepticism** — the agent questions specs rather than blindly following them
- **Confidence framework** — every finding gets High/Medium/Low confidence with rationale
- **Namespace isolation** — QA memory stays in `.qa/memory/`, never in `~/.claude/memory/`
- **Backward compatibility** — spec v2 features are additive; v1 specs still work unchanged
