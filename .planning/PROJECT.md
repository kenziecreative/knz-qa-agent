# QA Agent

## What This Is

An intelligent QA agent for Claude Code that tests web applications and user interfaces like a human would — with intuition, pattern recognition, and deep inspection. Built as a Claude Code plugin with specialized agents, skills, and hooks.

## Core Value

Tests should verify that features **actually work** for users, not just that code exists. If everything else fails, the agent must be able to run a spec and produce an accurate pass/fail result.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- Web application testing via Playwright browser tools — v0.1
- Spec-driven testing with markdown format — v0.1
- GSD integration via `/qa:check` at phase boundaries — v0.1
- Baseline accessibility checking (keyboard nav, focus, labels, ARIA) — v0.1
- Report generation to `.qa/reports/` with history tracking — v0.1
- Pattern recognition with 3-failure escalation — v0.1
- Spec skepticism for discrepancy handling — v0.1
- Skills: `/qa:check`, `/qa:run`, `/qa:init`, `/qa:gen`, `/qa:report` — v0.1
- Plugin foundation (skills-only, Anthropic-convention structure) — v0.2
- Agent architecture (memory, hooks, background execution) — v0.2
- Deep web testing (network intelligence, multi-viewport/persona, forms, SPA, error recovery) — v0.2
- Structured accessibility Tier 2 (focus management, landmarks, zoom, touch targets, reduced motion) — v0.2
- Spec format v2 (dependencies, data-driven, tags, environments, a11y depth) — v0.2
- Continuous monitoring `/qa:monitor` & cross-session reporting — v0.2
- Browser automation via `playwright-cli` (replaces Playwright MCP) — v0.2

### Active

<!-- Current scope. Building toward these. -->

- [ ] Visual & UX design verification (design reference comparison, typography, states, layout, feedback, images, animations, color, consistency, overflow, interaction states, web vitals, cross-browser, placeholders, breakpoints, UX smells)
- [ ] Usage guidance (docs/ORCHESTRATION.md + /qa:guide skill for orchestrating agents)

### Future (v0.3 Candidates)

<!-- Captured thinking for future milestones. See linked documents for full detail. -->

- UX Evaluation Mode — Cognitive walkthroughs, heuristic evaluation, affordance awareness, content/copy quality, cognitive load assessment, multi-lens personas, research recommendations, journey-level analysis. See [FUTURE-ux-evaluation-mode.md](.planning/FUTURE-ux-evaluation-mode.md) for full capture.

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- CLI/terminal testing — Different domain, different tooling; should be its own focused agent
- Screen reader testing — Requires specialized tooling (VoiceOver, NVDA, JAWS)
- Color contrast measurement — Requires computed color analysis and WCAG math
- Complex ARIA widget patterns — Requires deep a11y expertise, planned for accessibility agent
- GUI testing (non-web) — Different tooling required (Appium, etc.)
- Parallel scenario execution — Added complexity, sequential is sufficient for now

## Current Milestone: v0.3.0 Visual & UX Design Verification

**Goal:** Bridge the gap between "does it work?" and "does it look and feel right?" — adding visual/UX design verification capabilities and orchestration guidance.

**Target features:**
- Design reference comparison (mockup vs live page with root cause diagnosis)
- Typography verification (font loading, consistency, overflow, readability)
- Empty, loading, and error state verification
- Layout & spacing precision checks
- Interactive feedback quality (hover, cursor, toast, scroll)
- Image & media quality (broken images, aspect ratio, lazy loading)
- Animation & transition quality
- Color & theme consistency
- Cross-page consistency
- Content overflow with realistic data
- Interaction state matrix (systematic state sweep)
- Core Web Vitals (CLS, LCP, INP)
- Cross-browser rendering
- Placeholder & draft content detection
- Breakpoint transition sweep
- UX smell pattern detection
- Usage guidance (docs/ORCHESTRATION.md + /qa:guide skill)

## Context

The QA agent is a Claude Code plugin that provides `/qa:*` commands and skills. It uses `playwright-cli` (standalone CLI tool) for browser automation via the Bash tool. The agent tests with human-like judgment — not just checking if elements exist, but verifying they work correctly, feel responsive, and are accessible.

The focus is exclusively on web/UI testing. CLI testing was considered but deferred to keep the agent focused on its core strength: browser-based testing with visual inspection and interaction.

## Constraints

- **Runtime**: Claude Code session — all execution happens via tool calls
- **Browser**: `playwright-cli` — browser automation via standalone CLI, invoked through Bash
- **Dependencies**: `playwright-cli` installed globally
- **Architecture**: Claude Code plugin (agents, skills, hooks)

## Key Decisions

| Decision | Rationale | Outcome |
| -------- | --------- | ------- |
| Markdown specs over YAML/JSON | Human-readable, easy to write, leverages LLM interpretation | Good |
| Plugin architecture over global commands | Portable, self-contained, supports agents/hooks/MCP bundling | Good |
| Skills over commands | Auto-invocable, support context:fork, modern Claude Code pattern | Good |
| Web-only focus | Deep domain coverage > breadth; CLI can be separate agent | Current |
| Tier 2 accessibility as own phase | Enough depth to warrant focused attention, not an afterthought | Current |
| Baseline a11y integrated into functional testing | Same browser session, same context, one report | Good |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-20 — Phase 9 complete: agent can verify design fidelity (mockup comparison, typography, images, color/theme, cross-page consistency)*
