# Phase 13: Usage Guidance - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase delivers two things: (1) `docs/ORCHESTRATION.md` — a static reference document teaching both orchestrating AI agents and human developers how to use the QA plugin, and (2) `/qa:guide` — a context-aware skill that reads project state and returns guidance on which QA commands to use next. Minor updates to README.md and CLAUDE.md for discoverability.

</domain>

<decisions>
## Implementation Decisions

### ORCHESTRATION.md Audience & Structure
- **D-01:** Dual-audience document with layered sections — structured invocation tables and result-parsing blocks for AI agents, surrounding prose and orientation for human developers
- **D-02:** Sections should be clearly labeled so the two audiences don't bleed together (e.g., `## For Orchestrating Agents` with tables, `## Getting Started` with narrative)
- **D-03:** Must satisfy all three success criteria: (1) teach agents to invoke/interpret/integrate, (2) context-aware guidance via /qa:guide, (3) developer can read it and know how to use every skill without reading source files

### /qa:guide Skill Behavior
- **D-04:** No-agent, file-read only — does NOT spawn qa-tester agent. Matches the `report`/`init` pattern (skills that produce output from existing state read files directly)
- **D-05:** State inspection depth: (1) `.qa/` existence → suggest `/qa:init`, (2) spec file count in `.qa/*.md` → suggest `/qa:gen`, (3) `.qa/reports/` count → suggest `/qa:run`, (4) `.qa/HISTORY.md` line count → note trend data readiness, (5) `.qa/.monitor-alert` presence → surface monitoring alerts
- **D-06:** Output format is a short narrative (2-4 bullet decisions), not a checklist or decision tree — matches conversational tone of `report` and `check`
- **D-07:** Skill reads relevant sections of ORCHESTRATION.md to inform guidance (not just hard-coded logic)

### Relationship to README.md
- **D-08:** ORCHESTRATION.md is fully additive — no shared content with README, no duplication of skill usage examples
- **D-09:** ORCHESTRATION.md is self-contained (assumes reader has NOT seen README — primary reader is an AI agent)
- **D-10:** README.md gains a single pointer line linking to `docs/ORCHESTRATION.md` (new "Orchestration" section or line under existing structure)

### Discoverability & Integration
- **D-11:** CLAUDE.md updated to reference `docs/ORCHESTRATION.md` so AI agents working on the codebase know it exists
- **D-12:** README.md mentions `/qa:guide` alongside the other skills
- **D-13:** No SessionStart hook changes — discovery is passive (README + CLAUDE.md), not contextual. Hook semantics stay clean.

### Claude's Discretion
- Implementation approach for /qa:guide (D-04 through D-07 are the user's intent; exact file-reading logic and output formatting are Claude's call)
- ORCHESTRATION.md section ordering and exact section headers
- Whether to add `/qa:guide` to the existing skill listing pattern in README or create a new subsection
- Exact wording of the CLAUDE.md reference to ORCHESTRATION.md

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Requirements
- `.planning/REQUIREMENTS.md` §INFRA-04 — docs/ORCHESTRATION.md static reference doc
- `.planning/REQUIREMENTS.md` §INFRA-05 — /qa:guide context-aware usage guidance skill

### Existing Skills (pattern reference)
- `skills/run/SKILL.md` — Agent-spawning skill pattern (for contrast — /qa:guide does NOT follow this)
- `skills/report/SKILL.md` — File-read-only skill pattern (the pattern /qa:guide SHOULD follow)
- `skills/init/SKILL.md` — Another file-read-only skill pattern

### Existing Documentation
- `README.md` — Current user-facing documentation; gets a pointer line added
- `CLAUDE.md` — Project instructions for AI; gets ORCHESTRATION.md reference added

### Agent & Hooks
- `agents/qa-tester.md` — The agent that /qa:guide does NOT spawn (but ORCHESTRATION.md documents how other skills use it)
- `hooks/session-start.md` — State inspection pattern that /qa:guide's state-reading logic should mirror
- `hooks/hooks.json` — Hook definitions (NOT modified in this phase)

### Prior Phase Context
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — Spec format patterns and documentation conventions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Patterns
- **File-read skill pattern** (`skills/report/SKILL.md`, `skills/init/SKILL.md`): `allowed-tools: Read, Write, Bash, Grep, Glob` — no Agent tool. /qa:guide follows this pattern.
- **Session-start state inspection** (`hooks/session-start.md`): Already checks `.qa/` existence, report count, monitor alerts — same checks /qa:guide needs
- **Skill frontmatter convention**: name, description, argument-hint, allowed-tools fields

### Integration Points
- `docs/ORCHESTRATION.md` — new file in new `docs/` directory
- `skills/guide/SKILL.md` — new skill following established skill directory pattern
- `README.md` — single pointer line added
- `CLAUDE.md` — reference to ORCHESTRATION.md added

### Established Conventions
- Skills use colon syntax: `/qa:guide` (not `/qa-guide`)
- Plugin architecture: skill file at `skills/{name}/SKILL.md`
- No MCP server dependencies — everything through Bash tool

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 13-usage-guidance*
*Context gathered: 2026-04-20*
