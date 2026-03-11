# Phase 6: Continuous & Reporting - Context

**Gathered:** 2026-03-11
**Status:** Ready for planning

<domain>
## Phase Boundary

Loop-based monitoring of live applications with cross-session report reading, trend analysis, and GitHub issue creation from critical findings. The agent can run recurring smoke tests, detect regressions, persist results across sessions, and surface actionable trends.

</domain>

<decisions>
## Implementation Decisions

### Monitor behavior
- `/qa:monitor` invocation approach, scheduling mechanism, and integration with existing `/loop` skill are at Claude's discretion
- Monitor runs smoke-tagged scenarios by default (fast, focused on critical paths); user can override scope with `--tag`
- Monitoring targets a user-provided URL; no dev server management
- Alert approach: writes to `.qa/reports/` and surfaces findings via SessionStart hook on next session

### Report reading & trends
- `/qa:report` leads with the latest run results, followed by a trends section drawn from the last ~5 runs or 7 days (whichever is less)
- `--since` flag available for deeper historical analysis
- Trend presentation style is at Claude's discretion (text summaries vs mini tables — optimize for readability)

### Regression detection
- Regression = scenario that previously passed now fails (diff against baseline)
- Baseline established from most recent passing run
- Detection is automatic during monitoring runs

### GitHub issue creation
- Issue creation approach (auto vs ask-first, dedup strategy, labels, format) at Claude's discretion
- Uses `gh` CLI for issue creation

### Claude's Discretion
- `/qa:monitor` implementation: whether to wrap `/loop` or manage scheduling independently
- Monitor interval defaults
- Trend presentation format (text summary vs table)
- GitHub issue creation flow (auto for critical vs always confirm)
- Report file format and directory structure within `.qa/reports/`
- HISTORY.md structure for cross-session persistence
- Regression detection algorithm details

</decisions>

<specifics>
## Specific Ideas

- Report should answer "what just happened?" immediately, with "is this a pattern?" context below — not drown in data
- The value proposition of this phase is cross-session intelligence, not just re-running tests

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 06-continuous-reporting*
*Context gathered: 2026-03-11*
