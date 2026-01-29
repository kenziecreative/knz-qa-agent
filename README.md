# QA Agent

An intelligent QA agent for Claude Code that tests web applications like a human would - with intuition, pattern recognition, and deep inspection capabilities.

## Overview

This project provides QA testing commands that integrate with Claude Code. Unlike traditional E2E testing that follows rigid scripts, this agent:

- **Tests with judgment** - Understands intent, not just steps
- **Inspects deeply** - Console errors, network requests, timing, visual anomalies
- **Recognizes patterns** - Escalates after repeated failures instead of blindly retrying
- **Questions discrepancies** - When behavior differs from spec, asks which is correct
- **Includes accessibility** - Keyboard navigation and basic ARIA checks are part of every test

## Installation

The commands are installed as global Claude Code commands in `~/.claude/commands/qa/`.

**First-time install:**
```bash
mkdir -p ~/.claude/commands/qa
cp commands/*.md ~/.claude/commands/qa/
```

**To update:**
```bash
cp commands/*.md ~/.claude/commands/qa/
```

After installation, restart Claude Code. Commands will be available as `/qa:check`, `/qa:run`, etc.

## Commands

### `/qa:check` - Phase Verification Gate

Quick verification that a phase's deliverables actually work. Designed for GSD integration.

```bash
/qa:check --url http://localhost:3000
/qa:check --url http://localhost:3000 --phase 3
```

**Use when:** After completing a feature/phase, before moving on.

**What it does:**
1. Reads phase context (goal, plan, deliverables)
2. Runs targeted browser tests to verify functionality
3. Checks console, network, accessibility
4. Reports pass/fail with evidence
5. Writes report to `.qa/reports/`

**Escalation:** After 3 failures, recommends pausing to reconsider the implementation approach.

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

**Use when:** Full QA pass on a feature, regression testing, exploring new functionality.

### `/qa:init` - Initialize QA Structure

Set up the `.qa/` directory in a project with templates.

```bash
/qa:init ~/Projects/my-app
/qa:init .  # current directory
```

**Creates:**
- `.qa/README.md` - Quick reference
- `.qa/smoke-test.md` - Starter template

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

**Use when:** Bootstrapping specs for new features, converting PRDs to testable specs.

### `/qa:report` - Compile Results

Generate a summary report from the current session's test runs.

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

## Reports

Test results are written to `.qa/reports/` in the project being tested:

```
.qa/reports/
├── HISTORY.md                          # Running log of all QA activity
├── check-phase-3-2024-01-29-143022.md  # Individual check reports
└── run-login-2024-01-29-150145.md      # Individual run reports
```

## GSD Integration

For web projects using GSD, add to `PROJECT.md`:

```markdown
## Verification

This is a web application. After completing each phase, run `/qa:check --url http://localhost:3000` to verify the deliverables work before proceeding.
```

GSD's executor will see this and run QA checks at phase boundaries.

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

When observed behavior differs from spec:
- Notes the discrepancy
- Assesses if it seems intentional or buggy
- Asks for clarification if ambiguous

## Accessibility

Baseline accessibility is part of every test - not a separate pass:

**Always checked:**
- Keyboard navigation (Tab, Enter, Escape, arrows)
- Visible focus indicators
- Form labels and error association
- Alt text on images
- Semantic HTML (real buttons, real links)
- Basic ARIA (expanded states, live regions)

**Out of scope (needs specialized tools):**
- Screen reader testing
- Color contrast measurement
- Complex ARIA widget patterns

An advanced accessibility agent is planned as a separate project to handle these automated checks using axe-core and browser DevTools Protocol.

## Project Structure

```
knz-qa-agent/
├── README.md                           # This file
├── LEARNINGS.md                        # Decisions, patterns, lessons learned
├── commands/                           # Source commands (copy to ~/.claude/commands/qa/)
│   ├── check.md
│   ├── run.md
│   ├── init.md
│   ├── gen.md
│   └── report.md
├── agents/                             # Agent definition (reference)
│   └── qa-tester.md
├── skills/                             # Original skill files (reference)
│   └── ...
└── examples/
    ├── SPEC-FORMAT.md                  # Spec format documentation
    └── sample-login-spec.md            # Example spec
```

**Installed commands location:** `~/.claude/commands/qa/`

## Requirements

- Claude Code
- Docker MCP with Playwright browser tools enabled
- Web application running locally or accessible URL

## Development Notes

- Commands use the Docker MCP's Playwright browser tools
- Reports use Write tool to persist to `.qa/reports/`
- Phase context is read from GSD's `.planning/` structure when available
- Accessibility checks are integrated into the main test flow, not separate
- See [LEARNINGS.md](LEARNINGS.md) for architectural decisions and patterns

## Related Projects

- **Advanced Accessibility Agent** (planned) - Separate project for automated a11y testing with axe-core
- **GSD** - Project orchestration that integrates with `/qa:check`

## License

MIT

## Version History

- **0.1.0** - Initial release
  - Core commands: check, run, init, gen, report
  - GSD integration via `/qa:check`
  - Baseline accessibility built into methodology
  - Report output to `.qa/reports/` with HISTORY.md log
  - Pattern recognition with 3-failure escalation
  - Spec skepticism for discrepancy handling
