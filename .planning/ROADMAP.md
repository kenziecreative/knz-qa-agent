# Roadmap: QA Agent v0.2 (Robust Web/UI Testing)

## Overview

This milestone transforms the QA agent from a functional prototype into a robust, full-featured web/UI testing agent. The journey starts with cleaning up the plugin foundation, builds proper agent architecture with hooks and memory, deepens web testing methodology across six concern areas, adds structured accessibility testing, evolves the spec format, and finishes with continuous monitoring and cross-session reporting.

## Phases

- [x] **Phase 1: Cleanup & Foundation** - Fix stale references, consolidate to skills, bundle Playwright MCP, update plugin manifest
- [x] **Phase 2: Agent Architecture** - Agent memory, background execution, hook system, confidence levels
- [x] **Phase 3: Deep Web Testing** - Network intelligence, multi-viewport/persona orchestration, form testing, SPA awareness, error recovery
- [x] **Phase 4: Structured Accessibility** - Focus management, trapping, landmarks, zoom, touch targets, reduced motion, heading audits, form error patterns
- [x] **Phase 5: Spec Format v2** - Scenario dependencies, data-driven testing, tags/priorities, environment profiles, a11y depth
- [ ] **Phase 6: Continuous & Reporting** - Loop-based monitoring, cross-session report reading, trend analysis, GitHub issue creation

## Phase Details

### Phase 1: Cleanup & Foundation

**Goal**: Plugin is clean, modern, and properly structured with correct tool references and bundled dependencies

**Depends on**: Nothing (first phase)

**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05, FOUND-06

**Plans:** 2 plans

**Success Criteria** (what must be TRUE):

1. Skills are the primary invocation pattern; commands directory removed or marked legacy
2. qa-tester agent lists `mcp__playwright__*`, `Read`, `Write`, `Bash`, `Glob`, `Grep` in tools
3. Plugin includes `.mcp.json` bundling Playwright MCP server configuration
4. `plugin.json` has current version and settings defaults
5. No references to `Task` tool or `mcp__MCP_DOCKER__browser_*` remain in any file

Plans:

- [x] 01-01-PLAN.md — Delete stale files, rewrite agent, update skills/MCP/manifest
- [x] 01-02-PLAN.md — Update documentation, delete old global commands, set up dev symlink

### Phase 2: Agent Architecture

**Goal**: Agent has persistent memory, can run in background, and hook system enables automation

**Depends on**: Phase 1 (clean foundation to build on)

**Requirements**: ARCH-01, ARCH-02, ARCH-03, ARCH-04, ARCH-05, ARCH-06

**Plans:** 3 plans

**Success Criteria** (what must be TRUE):

1. qa-tester agent has `memory: project` and persists patterns across sessions
2. Agent can run tests in background while user continues working
3. `hooks/hooks.json` exists with at least `Stop` and `SessionStart` hooks
4. Stop hook detects significant code changes and suggests QA verification
5. SessionStart hook loads last QA results and known issues for current project
6. Every finding in a report includes a confidence level (high/medium/low)

Plans:

- [x] 02-01-PLAN.md — Add persistent memory and background execution to agent
- [x] 02-02-PLAN.md — Add structured confidence levels to reports and skills
- [x] 02-03-PLAN.md — Create hook system (hooks.json, Stop hook, SessionStart hook)

### Phase 3: Deep Web Testing

**Goal**: Agent has sophisticated methodology for network inspection, responsive testing, form validation, SPA behavior, and error recovery

**Depends on**: Phase 2 (agent architecture supports the deeper testing patterns)

**Requirements**: NET-01 through NET-05, VIEW-01 through VIEW-04, PERS-01 through PERS-03, FORM-01 through FORM-05, SPA-01 through SPA-04, ERR-01 through ERR-03

**Plans:** 4 plans

**Success Criteria** (what must be TRUE):

1. Agent inspects network requests during interactions, flags failures and slow calls
2. Agent systematically runs each scenario at each viewport defined in spec
3. Agent runs scenarios as each persona, handling auth flows between them
4. Agent has dedicated form testing methodology (validation states, errors, submission states, multi-step)
5. Agent detects SPA-specific issues (route changes, back/forward, state persistence, hydration)
6. Agent tests error recovery paths (API failures, user retry, page recovery)

Plans:

- [x] 03-01-PLAN.md — Network intelligence (status codes, slow calls, failed request/UI correlation, over-fetching, mixed content)
- [x] 03-02-PLAN.md — Multi-viewport and multi-persona orchestration (responsive testing, touch targets, auth flows, role-based visibility)
- [x] 03-03-PLAN.md — Form intelligence (validation boundaries, error messages, submission states, autofill, multi-step)
- [x] 03-04-PLAN.md — SPA awareness and error recovery (route sync, history, state persistence, hydration, API failure simulation, recovery paths)

### Phase 4: Structured Accessibility

**Goal**: Agent performs Tier 2 accessibility testing — deeper than baseline, using only Playwright (no external tools)

**Depends on**: Phase 3 (deep web testing methodology established)

**Requirements**: A11Y-01 through A11Y-17

**Plans:** 2 plans

**Success Criteria** (what must be TRUE):

1. Agent verifies focus management in modals (moves in, traps, returns on close)
2. Agent verifies skip links exist and work
3. Agent audits heading hierarchy and landmark regions across pages
4. Agent measures touch target sizes and flags elements below 44x44px
5. Agent checks `prefers-reduced-motion` respect and zoom to 200% behavior
6. Agent verifies form error patterns (summary, field links, aria-invalid, focus management)
7. Agent checks `lang` attribute and page title updates on navigation

Plans:

- [x] 04-01-PLAN.md — Focus management (modal open/close/trap, aria-live, skip links) and page structure (heading hierarchy, landmarks, page title, lang attribute)
- [x] 04-02-PLAN.md — Interactive elements (touch targets, color sole indicator, reduced motion, zoom) and form accessibility (error summary, field links, aria-invalid, focus on first error)

### Phase 5: Spec Format v2

**Goal**: Spec format supports dependencies, data-driven testing, selective execution, and environment profiles

**Depends on**: Phase 4 (all testing capabilities exist; now evolve the spec to leverage them)

**Requirements**: SPEC-01 through SPEC-08

**Plans:** 3 plans

**Success Criteria** (what must be TRUE):

1. Specs can express scenario dependencies (`depends_on`)
2. Specs support data-driven scenarios (multiple data sets, one scenario definition)
3. Specs support tags (`smoke`, `critical`, `regression`) and `/qa:run --tag` filters by them
4. Specs support environment profiles and `/qa:run --env` selects the active profile
5. Specs support `## Accessibility Focus` section to trigger Tier 2 depth
6. `/qa:gen --a11y-depth deep` generates specs with accessibility sections

Plans:

- [x] 05-01-PLAN.md — Define v2 spec format (SPEC-FORMAT.md, sample spec) and update agent to interpret v2 sections
- [x] 05-02-PLAN.md — Update qa-run (--tag, --env, dependency/data-driven flow) and qa-gen (--a11y-depth, v2 generation)
- [x] 05-03-PLAN.md — Update README and qa-init templates with v2 documentation

### Phase 6: Continuous & Reporting

**Goal**: Agent can monitor applications continuously and reports work across sessions with trend analysis

**Depends on**: Phase 5 (spec format supports what monitoring needs)

**Requirements**: MON-01 through MON-03, RPT-01 through RPT-05

**Plans:** 2 plans

**Success Criteria** (what must be TRUE):

1. `/qa:monitor` sets up recurring smoke tests via `/loop` against a target URL
2. Monitor detects regressions (previously passing test now fails)
3. `/qa:report` reads from `.qa/reports/` directory, not just conversation context
4. Reports include trend analysis from HISTORY.md (e.g., "this endpoint slow for last 3 runs")
5. Reports can generate GitHub issues from critical findings via `gh` CLI

Plans:

- [ ] 06-01-PLAN.md — Create /qa:monitor skill with /loop integration, regression detection, and result persistence
- [ ] 06-02-PLAN.md — Upgrade /qa:report with file-based reading, trend analysis, and GitHub issue creation

## Progress

Phases execute in order: 1 -> 2 -> 3 -> 4 -> 5 -> 6

| Phase | Plans Complete | Status | Completed |
| ----- | ------------- | ------ | --------- |
| 1. Cleanup & Foundation | 2/2 | Complete ✓ | 2026-03-10 |
| 2. Agent Architecture | 3/3 | Complete ✓ | 2026-03-10 |
| 3. Deep Web Testing | 4/4 | Complete ✓ | 2026-03-10 |
| 4. Structured Accessibility | 2/2 | Complete ✓ | 2026-03-10 |
| 5. Spec Format v2 | 3/3 | Complete ✓ | 2026-03-11 |
| 6. Continuous & Reporting | 0/2 | Not started | - |
