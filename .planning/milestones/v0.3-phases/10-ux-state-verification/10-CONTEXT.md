# Phase 10: UX State Verification - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase implements the UX state verification methodology in `qa-tester.md` — the Tier 2 visual checks triggered by `## Visual Focus: ux-states` in a spec. It covers four requirement areas: empty/loading/error/first-run state existence (STATE-01), interactive component state matrix sweep (STATE-02), interactive feedback quality including cursor/toast/scroll behavior (STATE-03), and animation/transition quality (STATE-04). Phase 8 created the spec format infrastructure (`## Visual Focus` tiering); Phase 9 established the dual pattern (eval probes + visual judgment). This phase fills in the methodology for UX states.

</domain>

<decisions>
## Implementation Decisions

### State Existence Detection (STATE-01)
- **D-01:** Active triggering as default methodology — the agent manipulates app state to force each UI state, then verifies it renders correctly. This goes beyond checking if markup exists — it confirms states actually fire under the right conditions.
- **D-02:** Per-state-type trigger heuristics:
  - **Empty state:** Clear storage (`localstorage-clear`, `cookie-delete`) + mock empty API response via `playwright-cli route` returning `[]`
  - **Loading state:** Intercept + delay API responses via `playwright-cli route` to hold the loading state visible
  - **Error state:** Route returning 5xx status codes (uses existing error simulation pattern from Phase 3 ERR-01)
  - **First-run/onboarding:** Clear all storage + cookies to simulate a fresh user session
- **D-03:** Passive DOM probing as fallback — when the spec provides no trigger guidance or when active manipulation would be destructive, the agent queries DOM for state-indicator patterns (empty containers, skeleton elements, error boundaries, onboarding markers) via `evaluate()`.
- **D-04:** State cleanup after each trigger — agent must restore app state before proceeding to the next state check to prevent state bleed.

### Interaction State Matrix (STATE-02)
- **D-05:** Interaction-first sweep — trigger states via hover/click/Tab like a user, eval to read computed styles after each trigger. Extends the existing DESIGN-04 hover/focus verification pattern to the full state matrix.
- **D-06:** Component selection via accessibility tree snapshot — sweep elements matching: `button`, `[role="button"]`, `a`, `input`, `select`, `textarea`, `[tabindex]`.
- **D-07:** State trigger sequence per component: default (baseline eval) → hover (`playwright-cli hover`) → focus (Tab to element) → active (screenshot during click — visual judgment only, mousedown is too momentary for eval) → disabled (find disabled variants via `[disabled]` or `aria-disabled` query) → loading/error/success (navigate to states where app exposes them, or use `playwright-cli route` for error simulation).
- **D-08:** Active state is visual-judgment-only (Medium confidence) — the mousedown state resolves in milliseconds, making eval capture unreliable. Screenshot during click sequence + visual assessment.

### Interactive Feedback (STATE-03)
- **D-09:** Cursor correctness via `getComputedStyle(el).cursor` eval probe — screenshots don't render cursors in headless Playwright. Check that buttons/links/interactive `[role]` elements return `pointer` and non-interactive elements return `default`/`auto`. High confidence when eval-confirmed.
- **D-10:** Toast/notification verification uses presence + disappearance assertions — assert visible after trigger action, then assert hidden/detached for auto-dismiss. Duration measurement only when spec explicitly declares a timing threshold (opt-in via spec, not default).
- **D-11:** Sticky header verification via two-step eval — (1) confirm `position: sticky` or `position: fixed` via computed style, (2) scroll via `mouse.wheel(0, 500)`, re-read `getBoundingClientRect().top` — value near 0 (or within declared `top` offset) confirms sticking. Screenshot after scroll as visual supplement.
- **D-12:** Content obscured by sticky header flagged as Medium confidence via visual judgment (supplements the eval probe).

### Animation & Transition Quality (STATE-04)
- **D-13:** CSS computed-style inference as primary mechanism — eval reads `transition`, `animation`, `will-change` properties. Checks for: layout-triggering properties (`width`/`height` transitions vs `transform`/`opacity`), missing transitions on interactive elements, duration inconsistency across same component class.
- **D-14:** No frame-rate measurement — Playwright exposes no FPS API. The agent is honest about this ceiling. Structural quality checks via eval are High confidence; perceived smoothness cannot be verified mechanically.
- **D-15:** Screenshot judgment limited to loading animation presence — during async operations, a screenshot can confirm a spinner/skeleton exists vs. static text. Binary check: animation present or absent.
- **D-16:** Complements existing reduced-motion checks (a11y Tier 2 lines 288-297) — those verify animation suppression when `prefers-reduced-motion: reduce` is set. This covers animation quality when animations ARE enabled.

### Overarching Pattern
- **D-17:** All four areas follow the dual pattern established in Phase 9: `evaluate()` for deterministic/measurable checks (High confidence) + visual judgment via screenshots for perceptual quality (Medium confidence).

### Claude's Discretion
- Exact JS evaluate snippets for state detection heuristics (empty state selectors, skeleton class patterns)
- Threshold values for sticky header position tolerance
- How to structure the methodology sections in qa-tester.md (section naming, ordering)
- Which interactive elements to prioritize in the state matrix sweep when many are present
- State cleanup implementation details (restore strategy after active triggering)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Spec Format Infrastructure (Phase 8 output)
- `agents/qa-tester.md` — Lines 733-747: Visual Focus Trigger including `ux-states` area definition
- `agents/qa-tester.md` — Lines 719-731: Accessibility Focus Trigger (pattern to mirror)
- `examples/SPEC-FORMAT.md` — Visual Focus section reference with `ux-states` area
- `skills/run/SKILL.md` — Lines 142-149: Visual Focus forwarding block in agent invocation template

### Existing Patterns to Extend
- `agents/qa-tester.md` — Lines 749-1019: Design Verification methodology (Phase 9 output — dual eval+visual pattern, confidence framework, section structure to mirror)
- `agents/qa-tester.md` — Lines 910-968: Color & Theme hover/focus state verification (DESIGN-04 — D-05 extends this pattern to full state matrix)
- `agents/qa-tester.md` — Lines 240-310: Structured Accessibility Tier 2 (methodology template precedent)
- `agents/qa-tester.md` — Lines 288-297: Reduced motion checks (D-16 complements, does not replace)
- `agents/qa-tester.md` — Lines 355-375: Confidence Summary Table (extend with STATE-* entries)

### Error Simulation Pattern
- `agents/qa-tester.md` — Error simulation via `playwright-cli route` (existing pattern used by D-02 for error states and D-07 for error state matrix)

### Requirements
- `.planning/REQUIREMENTS.md` §STATE-01 — Empty, loading, error, first-run states
- `.planning/REQUIREMENTS.md` §STATE-02 — Interaction state matrix sweep
- `.planning/REQUIREMENTS.md` §STATE-03 — Hover coverage, cursor, toast, scroll
- `.planning/REQUIREMENTS.md` §STATE-04 — Transition smoothness, consistency, loading animation

### Prior Phase Context
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — Visual Focus tiering decisions (D-01 through D-03)
- `.planning/phases/09-design-verification/09-CONTEXT.md` — Dual pattern decisions (D-15), confidence framework, methodology structure

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **`playwright-cli route`** — already available for intercepting network requests; used for error state triggering (D-02) and loading state delays
- **`playwright-cli hover`** — triggers hover state before eval reads computed styles (D-05, extends DESIGN-04)
- **`playwright-cli eval`** / **`playwright-cli run-code`** — run arbitrary JS for computed styles, DOM queries, cursor checks
- **`playwright-cli screenshot`** — visual state capture for perceptual checks
- **`playwright-cli snapshot`** — accessibility tree for interactive element identification (D-06)
- **`localstorage-clear`**, **`cookie-delete`** — storage manipulation for empty/first-run state triggering (D-02)
- **Confidence framework** — High/Medium/Low with rationale, established throughout qa-tester.md

### Established Patterns
- **Dual pattern** (eval + visual judgment) — established in Phase 9, all five design verification areas follow it
- **Visual Focus Trigger** — `ux-states` area stub exists at line 747, references Phase 10
- **DESIGN-04 hover/focus verification** — exact pattern to extend for full state matrix (D-05)
- **Error simulation** — `playwright-cli route` for 5xx responses, existing pattern from Phase 3

### Integration Points
- `agents/qa-tester.md` — New UX State Verification methodology section goes here
- `agents/qa-tester.md` line 747 — Remove "Tier 2 visual methodology pending Phase 10" note for `ux-states`
- `agents/qa-tester.md` Confidence Summary Table — Extend with STATE-* confidence entries

</code_context>

<specifics>
## Specific Ideas

### Active Triggering with Passive Fallback
The key insight from discussion: the agent should *force* states into existence (empty via clear storage, loading via route delay, error via 5xx route) rather than just checking if state markup exists in the DOM. Passive probing only catches what the DOM already shows — active triggering catches "state defined but never fires" bugs.

### Interaction-First as DESIGN-04 Extension
The state matrix sweep is a natural extension of the existing DESIGN-04 hover/focus color verification. Same tools, same eval-after-interaction pattern, but broader scope (8 states instead of 2) and broader assessment (not just color changes, but full visual/behavioral state changes).

### Honest Limitations
- Active state (mousedown) cannot be eval-captured — treated as visual-judgment-only (Medium confidence)
- Animation smoothness cannot be measured — no FPS API in Playwright. CSS property inference covers structural quality only.
- Cursor screenshots don't work in headless mode — eval probe is the only viable option

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 10-ux-state-verification*
*Context gathered: 2026-04-20*
