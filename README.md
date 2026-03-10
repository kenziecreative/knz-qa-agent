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

Playwright MCP is bundled — no separate setup required. The `.claude-plugin/.mcp.json` configuration launches the Playwright MCP server automatically when the plugin loads.

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
```

### `/qa:report` - Compile Results

Generate a summary report from QA test runs.

```bash
/qa:report
/qa:report --format json
/qa:report --save ./qa-report.md
```

## Spec Format

Specs live in `.qa/` directory as Markdown files:

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

See [examples/SPEC-FORMAT.md](examples/SPEC-FORMAT.md) for full format documentation.

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

**Planned (Tier 2 - structured a11y testing):**

- Focus management in modals (move in, trap, return)
- Skip links, landmarks, heading hierarchy
- Touch target sizing, zoom behavior, reduced motion
- Form error patterns (summary, field links, aria-invalid)

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
│   └── .mcp.json            # Playwright MCP server config (bundled)
├── agents/
│   └── qa-tester.md         # QA tester agent definition
├── skills/
│   ├── qa-run.md            # Full test execution
│   ├── qa-check.md          # Phase verification gate
│   ├── qa-init.md           # Initialize QA structure
│   ├── qa-gen.md            # Generate specs
│   └── qa-report.md         # Compile results
├── examples/
│   ├── SPEC-FORMAT.md       # Spec format documentation
│   └── sample-login-spec.md # Example spec
├── LEARNINGS.md             # Decisions, patterns, lessons learned
└── README.md                # This file
```

## Requirements

- Claude Code
- Playwright MCP (bundled via `.claude-plugin/.mcp.json` — no separate install)
- Web application running locally or accessible URL

## Development Notes

- See [LEARNINGS.md](LEARNINGS.md) for architectural decisions and patterns
- Planning tracked in `.planning/` directory (GSD workflow)
- Current milestone: v0.2 - Robust Web/UI Testing (6 phases)

## License

MIT

## Version History

- **0.2.0** (in progress) - Robust web/UI testing
  - Plugin foundation cleanup (skills-only, Playwright MCP bundled via `.mcp.json`)
  - Agent architecture (memory, hooks, background execution)
  - Deep web testing (network, viewports, personas, forms, SPA, error recovery)
  - Structured accessibility (Tier 2 — focus management, landmarks, zoom, touch targets)
  - Spec format v2 (dependencies, data-driven, tags, environments)
  - Continuous monitoring and cross-session reporting
- **0.1.0** - Initial release
  - Five skills: `/qa:run`, `/qa:check`, `/qa:init`, `/qa:gen`, `/qa:report`
  - GSD integration via `/qa:check`
  - Baseline accessibility built into methodology
  - Report output to `.qa/reports/` with HISTORY.md log
  - Pattern recognition with 3-failure escalation
  - Spec skepticism for discrepancy handling
