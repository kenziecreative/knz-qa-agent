# QA Agent

An intelligent QA agent for Claude Code that tests web applications like a human would - with intuition, pattern recognition, and deep inspection capabilities.

## Overview

This is a Claude Code plugin that provides QA testing for web applications. Unlike traditional E2E testing that follows rigid scripts, this agent:

- **Tests with judgment** - Understands intent, not just steps
- **Inspects deeply** - Console errors, network requests, timing, visual anomalies
- **Recognizes patterns** - Escalates after repeated failures instead of blindly retrying
- **Questions discrepancies** - When behavior differs from spec, asks which is correct
- **Checks accessibility** - Keyboard navigation and ARIA checks are part of every test

## Installation

Clone or install this plugin into your Claude Code plugins directory:

```bash
# As a Claude Code plugin
git clone <repo-url> ~/.claude/plugins/qa-agent
```

Requires `playwright-cli` installed globally (e.g., via Volta or npm). The agent uses it via the Bash tool — no MCP server dependency.

## Skills

### `/qa:run` - Full Spec Execution

Run comprehensive QA tests from spec files or ad-hoc descriptions.

```bash
# Run a specific spec
/qa:run --project ~/Projects/my-app --spec login

# Run all specs
/qa:run --project ~/Projects/my-app

# Ad-hoc testing
/qa:run --url http://localhost:3000 "Test the checkout flow end-to-end"

# Override URL (test against staging)
/qa:run --project ~/Projects/my-app --spec login --url https://staging.example.com

# Run only smoke-tagged scenarios
/qa:run --project ~/Projects/my-app --tag smoke

# Run against staging environment
/qa:run --project ~/Projects/my-app --spec checkout --env staging

# Combine: critical tests on staging
/qa:run --project ~/Projects/my-app --tag critical --env staging
```

### `/qa:check` - Phase Verification Gate

Quick verification that a phase's deliverables actually work. Designed for GSD integration.

```bash
/qa:check --url http://localhost:3000
/qa:check --url http://localhost:3000 --phase 3
```

**Escalation:** After 3 failures, recommends pausing to reconsider the implementation approach.

### `/qa:init` - Initialize QA Structure

Set up the `.qa/` directory in a project with templates.

```bash
/qa:init ~/Projects/my-app
/qa:init .
```

### `/qa:gen` - Generate Specs

Generate QA specs from descriptions, documentation, or live exploration.

```bash
# From description
/qa:gen login "User authentication with email/password"

# From existing docs
/qa:gen checkout --from ./docs/checkout-flow.md

# By exploring the app
/qa:gen profile --from http://localhost:3000/profile

# Generate with deep accessibility testing
/qa:gen dashboard --a11y-depth deep "Admin dashboard with tables and filters"
```

### `/qa:monitor` - Continuous Monitoring

Run recurring smoke tests against a live URL, detect regressions, and persist results across sessions.

```bash
# Monitor with default settings (smoke tag, 5-min interval)
/qa:monitor --project ~/Projects/my-app --url https://my-app.example.com

# Monitor critical paths every 10 minutes, run 12 times (2 hours)
/qa:monitor --project ~/Projects/my-app --url https://staging.my-app.com --tag critical --interval 10 --runs 12
```

**Regressions:** Compares each run against a baseline of last-known-good results. When a previously passing scenario fails, it's flagged as a regression and an alert marker is written for the next session.

### `/qa:report` - Compile Results

Generate a cross-session QA report from persisted test results, including trend analysis and optional GitHub issue creation.

```bash
# Report on a project
/qa:report --project ~/Projects/my-app

# Report with trend analysis from last 14 days
/qa:report --project ~/Projects/my-app --since 14d

# Create GitHub issues from critical findings
/qa:report --project ~/Projects/my-app --create-issues

# JSON output
/qa:report --project ~/Projects/my-app --format json
```

## Spec Format

Specs live in `.qa/` directory as Markdown files.

### Basic Spec

The minimal v1 spec format — fully supported and backward compatible:

```markdown
# Login Flow

## Overview
User authentication via email/password.

## Base URL
http://localhost:3000

## Personas (optional)
- unauthenticated user
- valid user (test@example.com / password123)

## Viewports (optional)
- desktop (1280x720)
- mobile (375x667)

## Test Scenarios

### 1. Successful Login
**Steps:**
1. Navigate to /login
2. Enter valid credentials
3. Click Sign In

**Expected:**
- Redirects to /dashboard
- Shows welcome message
- No console errors

### 2. Invalid Credentials
**Steps:**
1. Navigate to /login
2. Enter invalid credentials
3. Click Sign In

**Expected:**
- Error message displayed
- Stays on login page
- No 500 errors

## Edge Cases to Verify
- Empty form submission
- SQL injection attempt
- Rapid submit clicks

## Things to Watch For
- Login API should respond < 500ms
- Password field should be type="password"
- Form should be keyboard navigable
```

### v2 Features

V2 adds five optional features that layer on top of the basic format. All are backward compatible — existing specs work unchanged.

#### Tags

Label scenarios for selective execution:

```markdown
### 1. Successful Login
tags: [smoke, critical]
**Steps:**
...
```

Standard tags and their meanings:

- `smoke` — Quick health check; agent moves fast and tests core paths
- `critical` — High-stakes scenario; agent applies extra thoroughness
- `regression` — Previously broken behavior; agent watches for the specific bug
- `a11y` — Accessibility emphasis; agent goes deeper on keyboard and ARIA

Run only tagged scenarios: `/qa:run --project . --tag smoke` or `/qa:run --project . --tag smoke,critical`

#### Dependencies

Mark scenarios that require a prior scenario to have passed:

```markdown
### 3. Session Persistence
depends_on: Successful Login
**Steps:**
...
```

If `Successful Login` fails, `Session Persistence` is skipped with a note. Dependencies reference scenarios by their exact heading name. Single-level chaining — the spec author controls execution order.

#### Data-Driven Scenarios

Run one scenario with multiple data variants:

```markdown
### 2. Failed Login
tags: [smoke, regression]
**Steps:**
1. Navigate to /login
2. Enter {email} and {password}
3. Click Sign In

**Expected:**
- Error message: {expected_error}

## Data Sets
| email | password | expected_error |
|-------|----------|----------------|
| wrong@example.com | password123 | Invalid email or password |
| test@example.com | wrongpass | Invalid email or password |
| locked@example.com | password123 | Account is locked |
```

Each row runs as a separate variant. All variants run even if one fails — results show per-variant.

#### Environment Profiles

Define named environments to switch between with `--env`:

```markdown
## Environments

### local
- **base_url**: http://localhost:3000
- **email**: test@example.com
- **password**: password123

### staging
- **base_url**: https://staging.example.com
- **email**: staging-test@example.com
- **password**: $STAGING_PASSWORD
```

Select at run time: `/qa:run --project . --env staging`

When no `--env` flag is passed and no `## Environments` section exists, the `## Base URL` value is used (v1 behavior unchanged).

Secrets use `$ENV_VAR` placeholders — real values come from environment variables or `.qa/env.local` (gitignored).

#### Accessibility Focus

Add an `## Accessibility Focus` section to activate Tier 2 structured accessibility testing for specific areas:

```markdown
## Accessibility Focus
- form-accessibility
- focus-management
```

Available areas: `focus-management`, `page-structure`, `interactive-elements`, `form-accessibility`

Without this section, Tier 1 baseline accessibility checks always run (keyboard nav, ARIA states, alt text, semantic HTML). With this section, the agent runs the full Tier 2 structured methodology for each listed area.

Can also be activated at generation time: `/qa:gen dashboard --a11y-depth deep`

See [examples/SPEC-FORMAT.md](examples/SPEC-FORMAT.md) for complete format documentation and a full example spec.

## Reports

Test results are written to `.qa/reports/` in the project being tested:

```text
.qa/reports/
├── HISTORY.md                          # Running log of all QA activity
├── check-phase-3-2024-01-29-143022.md  # Individual check reports
└── run-login-2024-01-29-150145.md      # Individual run reports
```

## QA Methodology

### Testing Like a Human

The agent doesn't just check if things exist - it verifies they work correctly:

1. **Screenshot** before and after interactions
2. **Console** - errors, warnings, even info logs
3. **Network** - did expected requests happen? Response times?
4. **Timing** - did it feel responsive?
5. **Visual** - layout issues, missing images, broken styles
6. **Keyboard** - can you Tab to it? Does Enter work?

### Pattern Recognition

If the same test fails 3+ times:

> "This has failed repeatedly. The implementation approach may need reconsideration, not just another patch."

### Spec Skepticism

When observed behavior differs from spec, the agent notes the discrepancy, assesses whether it seems intentional or buggy, and asks for clarification if ambiguous.

## Accessibility

Baseline accessibility is part of every test - not a separate pass:

**Always checked:**

- Keyboard navigation (Tab, Enter, Escape, arrows)
- Visible focus indicators
- Form labels and error association
- Alt text on images
- Semantic HTML (real buttons, real links)
- Basic ARIA (expanded states, live regions)

**Tier 2 (activated via spec):**

Structured deep accessibility testing, activated by adding `## Accessibility Focus` to a spec or using `--a11y-depth deep` with `/qa:gen`:

- **focus-management** — modal focus trapping, skip links, aria-live regions
- **page-structure** — heading hierarchy, landmarks, page title, lang attribute
- **interactive-elements** — touch targets, color sole indicator, reduced motion, zoom
- **form-accessibility** — error summary, error links, aria-invalid, focus on first error

**Out of scope (Tier 3 - separate agent):**

- Screen reader testing
- Color contrast measurement
- Complex ARIA widget patterns

## GSD Integration

For web projects using GSD, add to `PROJECT.md`:

```markdown
## Verification

This is a web application. After completing each phase, run `/qa:check --url http://localhost:3000` to verify the deliverables work before proceeding.
```

## Project Structure

```text
knz-qa-agent/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── .mcp.json            # MCP server config (currently empty)
├── agents/
│   └── qa-tester.md         # QA tester agent definition
├── skills/
│   ├── run/SKILL.md         # Full test execution
│   ├── check/SKILL.md       # Phase verification gate
│   ├── init/SKILL.md        # Initialize QA structure
│   ├── gen/SKILL.md         # Generate specs
│   ├── monitor/SKILL.md     # Continuous monitoring
│   └── report/SKILL.md      # Compile results
├── hooks/
│   ├── hooks.json           # Hook registration config
│   ├── stop.md              # Stop hook — triggers QA on significant changes
│   └── session-start.md     # SessionStart hook — surfaces alerts and last report
├── examples/
│   ├── SPEC-FORMAT.md       # Spec format documentation
│   └── sample-login-spec.md # Example spec
├── LEARNINGS.md             # Decisions, patterns, lessons learned
└── README.md                # This file
```

## Requirements

- Claude Code
- `playwright-cli` installed globally (CLI tool for browser automation)
- Web application running locally or accessible URL

## Development Notes

- See [LEARNINGS.md](LEARNINGS.md) for architectural decisions and patterns
- Planning tracked in `.planning/` directory (GSD workflow)
- Current milestone: v0.2 - Robust Web/UI Testing (7 phases)

## License

MIT

## Version History

- **0.2.0** (in progress) - Robust web/UI testing
  - Plugin foundation cleanup (skills-only, browser automation via `playwright-cli`)
  - Agent architecture (memory, hooks, background execution)
  - Deep web testing (network, viewports, personas, forms, SPA, error recovery)
  - Structured accessibility (Tier 2 — focus management, landmarks, zoom, touch targets)
  - Spec format v2 (dependencies, data-driven, tags/filtering, environment profiles, accessibility depth)
  - Continuous monitoring and cross-session reporting
- **0.1.0** - Initial release
  - Five skills: `/qa:run`, `/qa:check`, `/qa:init`, `/qa:gen`, `/qa:report` (six in v0.2 with `/qa:monitor`)
  - GSD integration via `/qa:check`
  - Baseline accessibility built into methodology
  - Report output to `.qa/reports/` with HISTORY.md log
  - Pattern recognition with 3-failure escalation
  - Spec skepticism for discrepancy handling
