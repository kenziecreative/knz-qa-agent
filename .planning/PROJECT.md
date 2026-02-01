# QA Agent

## What This Is

An intelligent QA agent for Claude Code that tests applications like a human would — with intuition, pattern recognition, and deep inspection. Currently supports web testing via Playwright; CLI testing is being added.

## Core Value

Tests should verify that features **actually work** for users, not just that code exists. If everything else fails, the agent must be able to run a spec and produce an accurate pass/fail result.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- Web application testing via Playwright browser tools
- Spec-driven testing with markdown format
- GSD integration via `/qa:check` at phase boundaries
- Baseline accessibility checking (keyboard nav, focus, labels, ARIA)
- Report generation to `.qa/reports/` with history tracking
- Pattern recognition with 3-failure escalation
- Spec skepticism for discrepancy handling
- Commands: `/qa:check`, `/qa:run`, `/qa:init`, `/qa:gen`, `/qa:report`

### Active

<!-- Current scope. Building toward these. -->

- [ ] CLI/terminal application testing
- [ ] Seamless mode detection (web vs CLI) from spec type
- [ ] CLI-specific assertions (exit code, stdout, stderr, file system)
- [ ] Docker execution mode for isolated testing
- [ ] Environment variable handling from .env files
- [ ] Interactive command testing with stdin injection
- [ ] Setup/teardown per scenario

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Screen reader testing — Requires specialized tooling, planned for separate accessibility agent
- Color contrast measurement — Requires visual analysis, planned for separate accessibility agent
- Complex ARIA widget patterns — Requires accessibility expertise, planned for separate accessibility agent
- Parallel scenario execution — Added complexity, sequential is sufficient for v0.2

## Context

The QA agent is a Claude Code plugin that provides `/qa:*` commands. It uses Docker MCP's Playwright browser tools for web testing. CLI testing will use the Bash tool for local execution and `docker run` for isolated execution.

The alice-os project requested CLI testing to replace ad-hoc bash test scripts with structured specs and consistent reporting.

## Constraints

- **Runtime**: Claude Code session — all execution happens via tool calls
- **Browser**: Docker MCP Playwright — web tests run in containerized browser
- **CLI**: Bash tool — CLI tests run via command execution
- **Dependencies**: Docker required for isolated CLI testing

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Markdown specs over YAML/JSON | Human-readable, easy to write, matches existing web spec format | ✓ Good |
| Mode detection via `## Type` header | Seamless UX, backwards compatible (web is default) | — Pending |
| `~` expands in execution context | Docker isolation requires container-relative paths | — Pending |
| Teardown always runs | Prevents test artifact pollution | — Pending |

---
*Last updated: 2026-01-31 after milestone v0.2 started*
