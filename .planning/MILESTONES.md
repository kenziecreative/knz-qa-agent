# QA Agent Milestones

## v0.3 Visual & UX Design Verification (Shipped: 2026-04-22)

**Phases completed:** 7 phases, 13 plans, 18 tasks

**Key accomplishments:**

- One-liner:
- One-liner:
- One-liner:
- Layout & Content Integrity methodology section added to qa-tester.md with LAYOUT-01 (spacing/alignment/clipping via sibling gap comparison and all-element overflow detection) and LAYOUT-02 (cross-page structural consistency via landmark fingerprinting and link text sampling)
- Completed the Layout & Content Integrity methodology with LAYOUT-03 (realistic-data overflow testing via tiered DOM injection), LAYOUT-04 (placeholder/draft content detection via regex scan + LLM screenshot judgment), a 16-row reporting table, and updated the Visual Focus stub note to remove layout-integrity from pending.
- One-liner:
- Reporting Performance & Responsive Findings table:
- 601-line self-contained orchestration reference covering all 7 QA skills with argument tables, structured result parsing blocks, GSD phase gate integration, and developer Getting Started walkthrough
- Visual Testing section added to ORCHESTRATION.md with four subsections covering design-verification, ux-states, layout-integrity, and performance-responsive Tier 2 areas, plus --browsers flag in the /qa:run argument table
- Four missing v0.3 documentation gaps closed: breakpoint-sweep: continuous, placeholder_allowlist, three README v2 feature entries, and two CLAUDE.md Agent Architecture bullets

---

## v0.1.0 — Initial Release (Shipped)

**Shipped:** 2026-01-29

**Goal:** Web application QA testing with human-like judgment

**Delivered:**

- Core commands: `/qa:check`, `/qa:run`, `/qa:init`, `/qa:gen`, `/qa:report`
- Spec-driven testing with markdown format
- GSD integration at phase boundaries
- Baseline accessibility checking
- Report generation with history tracking
- Pattern recognition (3-failure escalation)
- Spec skepticism for discrepancy handling

**Phases:** 1-5 (pre-GSD, not tracked)

---

## v0.2.0 — Robust Web/UI Testing (In Progress)

**Started:** 2026-03-10

**Goal:** Transform the QA agent into a robust, full-featured web/UI testing agent with deep inspection capabilities, structured accessibility testing, evolved spec format, and continuous monitoring.

**Target features:**

- Clean plugin foundation (skills over commands, bundled Playwright MCP)
- Agent architecture (memory, background execution, hooks)
- Deep web testing (network, viewports, personas, forms, SPA, error recovery)
- Structured accessibility (Tier 2 — focus management, landmarks, zoom, touch targets)
- Spec format v2 (dependencies, data-driven, tags, environments)
- Continuous monitoring and cross-session reporting

**Phases:** 6 phases, starting at 1

**Direction change:** Originally scoped as CLI testing support. Reoriented on 2026-03-10 to focus exclusively on deepening web/UI testing capabilities. CLI testing deferred to a separate agent project.
