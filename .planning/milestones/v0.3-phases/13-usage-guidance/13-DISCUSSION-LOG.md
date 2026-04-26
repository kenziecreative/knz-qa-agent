# Phase 13: Usage Guidance - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 13-usage-guidance
**Areas discussed:** ORCHESTRATION.md audience & structure, /qa:guide skill behavior, Relationship to README.md, Discoverability & integration

---

## ORCHESTRATION.md Audience & Structure

| Option | Description | Selected |
|--------|-------------|----------|
| Dual-audience (Recommended) | Layered sections — structured tables for AI agents, surrounding prose for developers. One doc satisfies both success criteria. | ✓ |
| AI-primary | Terse machine-parseable content. Developers use README instead. Risks failing success criterion 3. | |
| You decide | Claude picks the approach that best fits the plugin's patterns. | |

**User's choice:** Dual-audience (Recommended)
**Notes:** Research showed success criteria explicitly require both AI-parseable content and human onboarding in this document.

---

## /qa:guide Skill Behavior

| Option | Description | Selected |
|--------|-------------|----------|
| File-read only (Recommended) | Reads .qa/ state + ORCHESTRATION.md, outputs short narrative guidance. Matches report/init pattern. Fast and deterministic. | |
| Agent-spawning | Spawns qa-tester for richer reasoning. Heavyweight, non-deterministic, adds browser overhead to a pure read task. | |
| You decide | Claude picks the approach that best fits the plugin's patterns. | ✓ |

**User's choice:** You decide
**Notes:** Claude's discretion. File-read only is the clear fit given plugin patterns — skills that produce output from existing state (init, report) never spawn the agent.

---

## Relationship to README.md

| Option | Description | Selected |
|--------|-------------|----------|
| Additive + README pointer (Recommended) | ORCHESTRATION.md is self-contained. README gains one line linking to it. Zero duplication, natural discovery path. | ✓ |
| Fully separate | No cross-linking. Docs stay independent. No edits to README. | |
| You decide | Claude picks the relationship that makes sense. | |

**User's choice:** Additive + README pointer (Recommended)
**Notes:** ORCHESTRATION.md assumes the reader hasn't seen README (primary reader is an AI agent). README gets a single pointer.

---

## Discoverability & Integration

| Option | Description | Selected |
|--------|-------------|----------|
| README + CLAUDE.md (Recommended) | Passive but reliable. CLAUDE.md tells AI agents, README tells developers. No hook complexity added. | ✓ |
| Also add SessionStart nudge | Hook suggests /qa:guide when .qa/ doesn't exist. Contextual but adds noise risk and hook complexity. | |
| You decide | Claude picks the discoverability approach. | |

**User's choice:** README + CLAUDE.md (Recommended)
**Notes:** Hook already has clean semantics. Adding detection logic risks fragility for marginal benefit.

---

## Claude's Discretion

- /qa:guide implementation approach (file-read only is the expected pattern, exact logic is Claude's call)
- ORCHESTRATION.md section ordering and exact headers
- /qa:guide placement in README skill listing
- Exact wording of CLAUDE.md reference

## Deferred Ideas

None — discussion stayed within phase scope
