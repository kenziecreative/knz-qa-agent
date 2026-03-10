# Requirements: QA Agent v0.2

**Defined:** 2026-03-10
**Core Value:** Tests verify features actually work for users

## v0.2 Requirements

Requirements for making the web/UI QA agent robust. Each maps to roadmap phases.

### Plugin Foundation (Phase 1)

- [ ] **FOUND-01**: Skills are the primary invocation pattern (commands removed or converted)
- [ ] **FOUND-02**: Agent tool list includes `mcp__playwright__*`, `Read`, `Write`, `Bash`, `Glob`, `Grep`
- [ ] **FOUND-03**: Plugin bundles Playwright MCP configuration via `.mcp.json`
- [ ] **FOUND-04**: Plugin manifest (`plugin.json`) includes version, settings defaults
- [ ] **FOUND-05**: Stale `Task` tool references replaced with `Agent` in all files
- [ ] **FOUND-06**: `mcp__MCP_DOCKER__browser_*` references updated to `mcp__playwright__*`

### Agent Architecture (Phase 2)

- [ ] **ARCH-01**: qa-tester agent has `memory: project` for cross-session pattern learning
- [ ] **ARCH-02**: qa-tester agent supports `background: true` for non-blocking test execution
- [ ] **ARCH-03**: Plugin defines hooks in `hooks/hooks.json`
- [ ] **ARCH-04**: `Stop` hook suggests QA check when GSD phase context detected
- [ ] **ARCH-05**: `SessionStart` hook loads project QA context (last run results, known issues)
- [ ] **ARCH-06**: Agent reports include confidence levels (high/medium/low) for every finding

### Deep Web Testing (Phase 3)

#### Network Intelligence

- [ ] **NET-01**: Agent validates API response status codes during UI interactions
- [ ] **NET-02**: Agent flags slow API calls (> 1s by default, configurable per spec)
- [ ] **NET-03**: Agent detects failed network requests and correlates with UI behavior
- [ ] **NET-04**: Agent identifies over-fetching (API responses larger than what UI displays)
- [ ] **NET-05**: Agent checks for mixed content (HTTP on HTTPS pages)

#### Multi-Viewport Orchestration

- [ ] **VIEW-01**: Agent systematically tests each scenario at each viewport defined in spec
- [ ] **VIEW-02**: Agent detects responsive breakpoint issues (overlapping elements, hidden content)
- [ ] **VIEW-03**: Agent verifies touch target sizes at mobile viewports (minimum 44x44px)
- [ ] **VIEW-04**: Agent checks horizontal scrolling doesn't appear at any defined viewport

#### Multi-Persona Testing

- [ ] **PERS-01**: Agent runs scenarios as each persona defined in spec
- [ ] **PERS-02**: Agent handles authentication flows per persona (login/logout between personas)
- [ ] **PERS-03**: Agent verifies role-based content visibility (admin sees X, user sees Y)

#### Form Intelligence

- [ ] **FORM-01**: Agent tests all validation states (empty, invalid, boundary values)
- [ ] **FORM-02**: Agent verifies error message placement and association with fields
- [ ] **FORM-03**: Agent tests submission states (loading, success, error)
- [ ] **FORM-04**: Agent checks autofill behavior and field type attributes
- [ ] **FORM-05**: Agent tests multi-step forms with back/forward navigation

#### SPA Awareness

- [ ] **SPA-01**: Agent verifies client-side route changes update URL and page title
- [ ] **SPA-02**: Agent tests browser back/forward through client-side routes
- [ ] **SPA-03**: Agent checks state persistence across navigation
- [ ] **SPA-04**: Agent detects hydration mismatches (SSR content vs client render)

#### Error Recovery

- [ ] **ERR-01**: Agent tests behavior when API returns errors (4xx, 5xx)
- [ ] **ERR-02**: Agent verifies user-facing error messages are helpful (not raw stack traces)
- [ ] **ERR-03**: Agent checks recovery paths (can user retry? does page recover?)

### Structured Accessibility (Phase 4)

#### Focus Management

- [ ] **A11Y-01**: Agent verifies focus moves into modals/dialogs when opened
- [ ] **A11Y-02**: Agent verifies focus returns to trigger element when modal closes
- [ ] **A11Y-03**: Agent verifies focus is trapped within modals (Tab doesn't escape)
- [ ] **A11Y-04**: Agent verifies dynamically added content is announced (aria-live)
- [ ] **A11Y-05**: Agent verifies skip links exist and function correctly

#### Page Structure

- [ ] **A11Y-06**: Agent audits heading hierarchy (h1-h6 tree, no skipped levels, single h1)
- [ ] **A11Y-07**: Agent verifies landmark regions (main, nav, banner, contentinfo)
- [ ] **A11Y-08**: Agent checks page title changes on SPA navigation
- [ ] **A11Y-09**: Agent verifies `lang` attribute on html element

#### Interactive Elements

- [ ] **A11Y-10**: Agent measures touch target sizes (minimum 44x44px)
- [ ] **A11Y-11**: Agent verifies color is not sole indicator for state changes
- [ ] **A11Y-12**: Agent checks `prefers-reduced-motion` is respected (animations can be stopped)
- [ ] **A11Y-13**: Agent verifies zoom to 200% works without horizontal scroll or content loss

#### Form Accessibility

- [ ] **A11Y-14**: Agent verifies error summary exists after form submission with errors
- [ ] **A11Y-15**: Agent verifies error summary links to individual fields
- [ ] **A11Y-16**: Agent checks `aria-invalid` is set on errored fields
- [ ] **A11Y-17**: Agent verifies focus moves to first error after submission

### Spec Format v2 (Phase 5)

- [ ] **SPEC-01**: Specs support scenario dependencies (`depends_on: scenario-1`)
- [ ] **SPEC-02**: Specs support data-driven scenarios (test same flow with multiple data sets)
- [ ] **SPEC-03**: Specs support tags for selective execution (`tags: [smoke, critical, regression]`)
- [ ] **SPEC-04**: Specs support environment profiles (local, staging, production URLs and credentials)
- [ ] **SPEC-05**: Specs support `## Accessibility Focus` section for Tier 2 a11y depth
- [ ] **SPEC-06**: `/qa:gen` generates specs with a11y sections when `--a11y-depth deep` passed
- [ ] **SPEC-07**: `/qa:run` supports `--tag` flag to run subset of scenarios
- [ ] **SPEC-08**: `/qa:run` supports `--env` flag to select environment profile

### Continuous Monitoring & Reporting (Phase 6)

- [ ] **MON-01**: `/qa:monitor` skill sets up recurring test execution via `/loop`
- [ ] **MON-02**: Monitor runs smoke tests at configurable interval against target URL
- [ ] **MON-03**: Monitor alerts on regression (test that previously passed now fails)
- [ ] **RPT-01**: `/qa:report` reads from `.qa/reports/` directory (not just conversation context)
- [ ] **RPT-02**: Reports include trend analysis from HISTORY.md data
- [ ] **RPT-03**: Reports identify recurring failures across multiple runs
- [ ] **RPT-04**: Reports can generate GitHub issues from critical findings (via `gh` CLI)
- [ ] **RPT-05**: Cross-session report reading works without prior conversation context

## Out of Scope

| Feature | Reason |
|---------|--------|
| CLI/terminal testing | Different domain; separate agent project |
| Screen reader testing | Requires VoiceOver/NVDA/JAWS integration |
| Color contrast ratio calculation | Requires computed color analysis + WCAG math |
| Complex ARIA widget validation | Requires deep a11y expertise; accessibility agent |
| GUI testing (non-web) | Different tooling (Appium, etc.) |
| Parallel scenario execution | Complexity vs. value; sequential sufficient |
| Full WCAG 2.2 AA compliance audit | Scope of dedicated accessibility agent |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 through FOUND-06 | Phase 1 | Complete |
| ARCH-01 through ARCH-06 | Phase 2 | Pending |
| NET-01 through NET-05 | Phase 3 | Pending |
| VIEW-01 through VIEW-04 | Phase 3 | Pending |
| PERS-01 through PERS-03 | Phase 3 | Pending |
| FORM-01 through FORM-05 | Phase 3 | Pending |
| SPA-01 through SPA-04 | Phase 3 | Pending |
| ERR-01 through ERR-03 | Phase 3 | Pending |
| A11Y-01 through A11Y-17 | Phase 4 | Pending |
| SPEC-01 through SPEC-08 | Phase 5 | Pending |
| MON-01 through MON-03 | Phase 6 | Pending |
| RPT-01 through RPT-05 | Phase 6 | Pending |

Coverage: 64 total requirements, 64 mapped to phases, 0 unmapped.

---
*Requirements defined: 2026-03-10*
