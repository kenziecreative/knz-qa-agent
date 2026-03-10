# Phase 3: Deep Web Testing - Context

**Gathered:** 2026-03-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Sophisticated testing methodology for network inspection, responsive/multi-viewport testing, persona orchestration, form validation, SPA behavior, and error recovery. The agent gains deeper testing capabilities beyond basic scenario execution. Accessibility testing is a separate phase (Phase 4).

</domain>

<decisions>
## Implementation Decisions

### Network intelligence
- Claude's discretion on slow request thresholds (sensible defaults by request type)
- Claude's discretion on which failed requests to flag (context-aware, not blanket)
- Tiered approach: observe network traffic by default during normal test flows; actively simulate failures (route interception, throttling) only when spec explicitly requests error recovery scenarios
- Claude's discretion on report detail level for network findings

### Viewport orchestration
- Default viewport set: mobile (375px), tablet (768px), desktop (1280px)
- Specs can override — narrow to specific viewports or add custom sizes
- Every scenario runs at each viewport in the active set unless spec narrows it

### Persona orchestration
- Auth method is spec-defined per persona — some personas use full login flow, others can use token/cookie swap
- Claude's discretion on which scenarios benefit from multi-persona testing (not every scenario at every persona)
- Claude's discretion on whether to verify access denials for unauthorized personas

### Form testing methodology
- Exhaustive boundary testing: every validation rule boundary — min/max lengths, special characters, SQL injection-like strings, XSS attempts, invalid formats, empty required fields
- All submission states tested: loading indicator, success feedback, error handling, double-submit prevention
- Claude's discretion on multi-step form depth (forward + backward, data persistence, skip-ahead)
- Claude's discretion on file upload testing feasibility

### SPA awareness
- Claude's discretion on SPA navigation checks (URL sync, back/forward, deep links, scroll position, title updates)
- Claude's discretion on state persistence testing across route changes

### Error recovery
- Tiered approach: observe natural errors by default; simulate failures via Playwright route interception when spec includes error recovery scenarios
- Claude's discretion on crash recovery strategy (screenshot + continue vs retry)

### Claude's Discretion
- Network: slow thresholds, failure flagging logic, report verbosity
- Viewports: none (defaults defined above, spec overrides)
- Personas: scenario-persona pairing, access denial verification
- Forms: multi-step depth, file upload handling
- SPA: navigation check depth, state persistence scope
- Errors: crash recovery strategy

</decisions>

<specifics>
## Specific Ideas

- Network testing mirrors the tiered pattern from Phase 2's confidence system — observe by default, go deep when asked
- Form testing should be exhaustive at the boundary level — this is where real bugs hide
- All four submission states (loading, success, error, double-submit) matter for every form

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 03-deep-web-testing*
*Context gathered: 2026-03-10*
