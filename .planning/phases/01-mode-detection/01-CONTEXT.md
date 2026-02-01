# Phase 1: Mode Detection - Context

**Gathered:** 2026-01-31
**Status:** Ready for planning

<domain>
## Phase Boundary

Agent detects whether to execute CLI or web testing mode by reading spec content. This phase adds mode detection only — actual CLI execution is Phase 2.

</domain>

<decisions>
## Implementation Decisions

### Header parsing
- `## Type: cli` header triggers CLI execution mode
- No type header defaults to web mode (backward compatible)
- `## Type: web` explicitly sets web mode
- Parsing should be case-insensitive for the value (`cli`, `CLI`, `Cli` all work)

### Command behavior
- `/qa:run` handles both modes without additional flags
- Mode is detected from spec content, not command arguments
- Agent should indicate which mode was detected when starting execution

### Claude's Discretion
- CLI spec section structure (command format, expected output sections, etc.)
- Use existing web spec patterns as reference for CLI equivalents
- Header parsing implementation details
- How mode detection integrates with existing spec loading code

</decisions>

<specifics>
## Specific Ideas

No specific requirements — use web spec patterns as reference for CLI equivalents. The goal is consistency between web and CLI spec formats while accommodating CLI-specific needs.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-mode-detection*
*Context gathered: 2026-01-31*
