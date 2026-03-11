# Phase 5: Spec Format v2 - Context

**Gathered:** 2026-03-11
**Status:** Ready for planning

<domain>
## Phase Boundary

Evolve the QA spec format to support scenario dependencies, data-driven testing, tags/filtering, environment profiles, and accessibility depth controls. This phase changes the spec format and the agent's ability to parse/act on it. New testing capabilities (monitoring, reporting) belong in Phase 6.

</domain>

<decisions>
## Implementation Decisions

### Dependency syntax
- When a dependency fails, dependent scenarios are **skipped with a note** (e.g., "Skipped: dependency 'Login' failed") — no attempt to run them
- Chaining depth and reference style (name vs ID): Claude's discretion
- Setup/teardown semantics: Claude's discretion

### Data-driven design
- All data-driven design decisions left to Claude's discretion:
  - Where data sets live (inline, external, or both)
  - How results report per-variant (per-line vs grouped)
  - Whether pattern generation is supported
  - Variable substitution vs contextual interpretation

### Tags & filtering
- **Standard tags + custom**: Built-in tags like `smoke`, `critical`, `regression`, `a11y` with defined meanings, plus freeform custom tags
- Tag combination logic (AND/OR), negation support, auto-suggestion by generator: Claude's discretion

### Environment profiles
- All environment profile design decisions left to Claude's discretion:
  - Where profiles are defined (per-spec, shared config, or both)
  - What values profiles contain (URLs, credentials, timeouts, flags)
  - Default behavior when no --env flag is passed
  - Secrets handling

### Claude's Discretion
- Dependency chaining depth (single-level vs transitive resolution)
- Dependency reference style (scenario name vs explicit ID)
- Setup/teardown support within dependency system
- Data set storage location and format
- Data-driven result presentation
- Pattern-based test data generation
- Template variables vs contextual interpretation for data-driven steps
- Tag combination logic and negation syntax
- Auto-tagging in /qa:gen
- Environment profile location and structure
- Default environment behavior
- Secrets handling in profiles
- Accessibility depth section format (`## Accessibility Focus` trigger)

</decisions>

<specifics>
## Specific Ideas

- README will need extensive updating to document all new spec format features — plan should account for documentation as a deliverable
- Standard tags should include at minimum: `smoke`, `critical`, `regression`, `a11y`
- `/qa:gen --a11y-depth deep` should generate specs with accessibility sections (per success criteria)
- `/qa:run --tag` and `/qa:run --env` are the user-facing interfaces for filtering and profile selection

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 05-spec-format-v2*
*Context gathered: 2026-03-11*
