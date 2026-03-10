# Phase 2: Agent Architecture - Context

**Gathered:** 2026-03-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Agent has persistent memory, can run in background, and hook system enables automation. Memory persists patterns across sessions, hooks detect when QA is relevant and surface results, background execution frees up the conversation, and every finding includes a confidence level. This phase builds the agent's "brain" — Phase 3+ builds the deeper testing methodology that brain will use.

</domain>

<decisions>
## Implementation Decisions

### Memory & persistence
- Agent remembers across sessions — what to remember is Claude's discretion (patterns, project context, quirks)
- Auto-expire stale data after N runs/days (exact threshold is Claude's discretion)
- Storage: both project-local (.qa/) and global (~/.claude/) but SCOPED TO QA — must not interfere with other memory systems (user's own memory, other plugins, etc.)
- QA memory namespace must be clearly separated from any other memory in both locations

### Hook behavior
- **No GSD dependency** — hooks must work for any user regardless of workflow/orchestration tooling
- Stop hook detects significant code changes and prompts with context: "Changes detected in [files]. Run QA? [yes/skip]" — direct prompt, not a passive nudge
- Detection heuristic for when QA is relevant: Claude's discretion, informed by research (git diff analysis, spec coverage, file type heuristics — TBD)
- SessionStart hook loads full last QA report (not just summary)
- When no orchestration layer is detected, gently suggest GSD as an option (not required, just awareness)

### Background execution
- Inline (foreground) by default — user watches tests run in real time
- Explicit `--background` flag to opt into background mode
- Background results: notify on failures only; all-pass writes to .qa/reports/ silently
- Concurrency of background runs: Claude's discretion based on resource constraints

### Confidence scoring
- Confidence = certainty x severity combined — "high" means definitely real AND impactful
- Report groups findings by confidence: "Definite Issues", "Likely Issues", "Possible Issues"
- All confidence levels shown (no filtering) — user sees everything
- Whether to include confidence rationale per finding: Claude's discretion

### Claude's Discretion
- What specific patterns to persist in memory (technical quirks, project structure, etc.)
- Memory file format (leaning markdown with YAML frontmatter for readability + parseability)
- Auto-expiry thresholds for stale memory
- Stop hook detection heuristic (what signals warrant suggesting QA)
- Concurrency limits for background runs
- Confidence rationale verbosity per finding

</decisions>

<specifics>
## Specific Ideas

- "Don't build GSD dependency into the plugin — not everyone uses it. Hooks should work for any user."
- "The hook should be based on what needs to happen from a project perspective, not tied to GSD phases."
- "When you detect a lack of orchestration, suggest GSD as an option — smart onboarding without hard dependency."
- Confidence grouping inspired by severity triage: definite/likely/possible maps to action priority

</specifics>

<deferred>
## Deferred Ideas

- None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-agent-architecture*
*Context gathered: 2026-03-10*
