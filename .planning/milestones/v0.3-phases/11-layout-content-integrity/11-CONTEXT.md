# Phase 11: Layout & Content Integrity - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase implements the layout & content integrity methodology in `qa-tester.md` — the Tier 2 visual checks triggered by `## Visual Focus: layout-integrity` in a spec. It covers four requirement areas: spacing/alignment/grid integrity/clipping verification (LAYOUT-01), cross-page structural consistency (LAYOUT-02), realistic-data overflow testing (LAYOUT-03), and placeholder/draft content detection (LAYOUT-04). Phase 8 created the spec format infrastructure; Phase 9 established the dual pattern (eval + visual judgment). This phase fills in the methodology for layout and content integrity.

</domain>

<decisions>
## Implementation Decisions

### Spacing & Alignment Checks (LAYOUT-01)
- **D-01:** Dual pattern — `evaluate()` sibling comparison as primary (High confidence) + screenshot visual judgment as supplement (Medium confidence). Follows the established dual pattern from Phase 9.
- **D-02:** Eval probes query `getBoundingClientRect()` and `getComputedStyle()` on sibling element sets (cards, list items, nav items, table rows), compute inter-element gaps, and flag outliers where spacing deviates from the sibling median.
- **D-03:** Parent-child padding balance checked via `getComputedStyle()` — compare padding-top/bottom and padding-left/right for symmetry within containers. Asymmetric padding flagged as Medium confidence (design intent may be intentional).
- **D-04:** Grid/flex integrity checked via computed `display`, `gap`, `align-items`, `justify-content` properties — structural validation that grid/flex containers have consistent gap values and alignment.
- **D-05:** Content clipping detected via `scrollWidth > clientWidth` or `scrollHeight > clientHeight` — same overflow detection as existing DESIGN-02 text overflow, extended to ALL visible elements (not just text). High confidence when eval-confirmed.
- **D-06:** Screenshot supplements eval for complex layouts where DOM structure doesn't yield clean sibling sets — agent takes viewport screenshot and visually assesses alignment and spacing balance at Medium confidence.
- **D-07:** Distinct from Phase 9 DESIGN-01 spacing: DESIGN-01 compares live vs reference image. LAYOUT-01 operates WITHOUT a reference — it infers correctness from internal consistency (sibling comparison, padding symmetry).

### Cross-Page Structural Consistency (LAYOUT-02)
- **D-08:** Structural fingerprint + link/content sampling approach. Eval probes check landmark presence AND sample nav/footer link text per page.
- **D-09:** Structural fingerprint via eval: query `[role="banner"]`, `[role="navigation"]`, `[role="main"]`, `[role="contentinfo"]`, `header`, `nav`, `footer`, `main` elements. Record element count, child count, and landmark role presence per page.
- **D-10:** Content sampling: extract link text arrays from header/nav/footer regions per page. Compare link counts and link text across pages. Missing region = High confidence finding. Link count deviation = Medium confidence (some pages legitimately have different nav).
- **D-11:** Distinct from Phase 9 DESIGN-05: DESIGN-05 compares CSS computed style values (fontFamily, fontSize, etc.) across pages. LAYOUT-02 compares structural/DOM consistency — same header structure, same footer presence, same nav links.
- **D-12:** Screenshot per page confirms layout position of shared regions (header at top, footer at bottom) as visual supplement.

### Realistic-Data Overflow Testing (LAYOUT-03)
- **D-13:** `evaluate()` textContent injection as primary mechanism. Agent injects test content directly into DOM elements via eval, then checks for overflow. No route interception needed as default path.
- **D-14:** Tiered test content vocabulary:
  - Long Latin strings (150+ characters) — basic overflow stress
  - German compound words (`Donaudampfschifffahrtsgesellschaft`) — word-break stress
  - Large currency values (`$1,234,567,890.99`) — number container overflow
  - Long email addresses (`firstname.lastname+tag@very-long-subdomain.example.com`) — form field overflow
- **D-15:** Injection targets one logical section at a time (nav labels, table cells, card titles, form fields) rather than page-wide — keeps overflow attribution clear when logging findings.
- **D-16:** Overflow detection via `scrollWidth > clientWidth` or `scrollHeight > clientHeight` after injection — same mechanism as DESIGN-02 and LAYOUT-01 clipping checks. High confidence when eval-confirmed.
- **D-17:** Screenshot after injection as visual supplement for perceptual layout break assessment (Medium confidence).
- **D-18:** Route interception available as spec-driven escalation path — when a spec explicitly calls for locale-driven or API-response-based content testing, the agent uses `playwright-cli route` to mock responses with long data (follows existing route pattern from STATE-01).

### Placeholder & Draft Content Detection (LAYOUT-04)
- **D-19:** Dual approach — `evaluate()` regex/keyword scan as primary (High confidence) + LLM screenshot judgment for ambiguous cases (Medium confidence).
- **D-20:** Eval regex scan covers:
  - Lorem ipsum and variants (`lorem`, `ipsum`, `dolor sit amet`, `consectetur adipiscing`)
  - TODO/FIXME/HACK/XXX strings in visible content
  - Placeholder image URLs (`placehold.it`, `via.placeholder`, `lorempixel`, `picsum.photos`, `dummyimage`)
  - Generic placeholder text patterns (`[Your Name]`, `[Company Name]`, `example.com`, `test@test.com`, `123-456-7890`)
- **D-21:** Scan scope includes visible text (`innerText`), `alt` attributes, `title` attributes, `aria-label` attributes, and `<meta>` content attributes — not just what's visible on screen.
- **D-22:** LLM screenshot judgment covers:
  - Vague CTAs ("Click here", "Learn more", "Submit" without context) — Medium confidence
  - Casing inconsistency (mixed Title Case and sentence case across similar elements) — Medium confidence
  - Placeholder images (gray boxes, stock photo watermarks) — Medium confidence via visual inspection
- **D-23:** Spec-level allowlist for known false positives — if a product is literally named "TODO" or a CTA "Learn more" is intentional, the spec can declare `placeholder_allowlist: [TODO, Learn more]` to suppress those findings.

### Overarching Pattern
- **D-24:** All four areas follow the dual pattern established in Phase 9: `evaluate()` for deterministic/measurable checks (High confidence) + visual judgment via screenshots for perceptual quality (Medium confidence).

### Claude's Discretion
- Exact JS evaluate snippets for sibling comparison and structural fingerprinting
- Threshold values for spacing deviation (how much gap variation triggers a finding)
- Exact regex patterns for placeholder detection
- How to structure the methodology sections in qa-tester.md (section naming, ordering)
- Which sibling element sets to prioritize when many exist on a page
- How to normalize link text for cross-page comparison (case, whitespace, trailing slashes)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Spec Format Infrastructure (Phase 8 output)
- `agents/qa-tester.md` — Lines 733-747: Visual Focus Trigger including `layout-integrity` area definition
- `agents/qa-tester.md` — Line 742: `layout-integrity` area stub ("spacing, alignment, grid integrity, content clipping")
- `agents/qa-tester.md` — Line 747: Pending note to remove ("Tier 2 visual methodology for `layout-integrity`... will be added in Phases 11-12")
- `examples/SPEC-FORMAT.md` — Visual Focus section reference with `layout-integrity` area
- `skills/run/SKILL.md` — Lines 142-149: Visual Focus forwarding block in agent invocation template

### Existing Patterns to Extend
- `agents/qa-tester.md` — Lines 749-1097: Design Verification + UX State Verification methodology (Phases 9-10 output — dual eval+visual pattern, confidence framework, section structure to mirror)
- `agents/qa-tester.md` — Lines 820-841: Text overflow detection (DESIGN-02) — same `scrollWidth > clientWidth` mechanism reused by LAYOUT-01 and LAYOUT-03
- `agents/qa-tester.md` — Lines 1021-1068: Cross-Page Consistency (DESIGN-05) — complements LAYOUT-02; DESIGN-05 = CSS values, LAYOUT-02 = structural/DOM consistency
- `agents/qa-tester.md` — Lines 355-375: Confidence Summary Table (extend with LAYOUT-* entries)

### Requirements
- `.planning/REQUIREMENTS.md` §LAYOUT-01 — Spacing consistency, element alignment, grid/flex integrity, content clipping
- `.planning/REQUIREMENTS.md` §LAYOUT-02 — Header/footer, component, interaction pattern consistency across pages
- `.planning/REQUIREMENTS.md` §LAYOUT-03 — Test with realistic-length content, report where layout breaks
- `.planning/REQUIREMENTS.md` §LAYOUT-04 — Detect placeholder text, filler data, casing inconsistency, vague CTAs

### Prior Phase Context
- `.planning/phases/08-spec-format-extensions/08-CONTEXT.md` — Visual Focus tiering decisions (D-01 through D-03)
- `.planning/phases/09-design-verification/09-CONTEXT.md` — Dual pattern decisions (D-15), confidence framework, methodology structure
- `.planning/phases/10-ux-state-verification/10-CONTEXT.md` — Active triggering, state matrix, cleanup mandates

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **`playwright-cli eval`** — run arbitrary JS for computed styles, DOM queries, overflow checks
- **`playwright-cli screenshot`** — visual state capture for perceptual checks
- **`playwright-cli snapshot`** — accessibility tree for element identification
- **`playwright-cli route`** — API interception for escalation path (LAYOUT-03 locale testing)
- **Text overflow detection** (qa-tester.md lines 820-841) — `scrollWidth > clientWidth` pattern, directly reusable for LAYOUT-01 and LAYOUT-03
- **Cross-page consistency** (qa-tester.md lines 1021-1068) — page sampling pattern, complemented by LAYOUT-02 structural comparison
- **Confidence framework** — High/Medium/Low with rationale, established throughout qa-tester.md

### Established Patterns
- **Dual pattern** (eval + visual judgment) — established in Phase 9, all areas follow it
- **Visual Focus Trigger** — `layout-integrity` area stub exists at line 742, references Phase 11
- **Landmark query pattern** — accessibility checks already query `[role="banner"]`, `[role="navigation"]`, etc. — reusable for LAYOUT-02 structural fingerprint

### Integration Points
- `agents/qa-tester.md` — New Layout & Content Integrity methodology section goes here (after UX State Verification)
- `agents/qa-tester.md` line 747 — Remove "Tier 2 visual methodology pending Phase 11" note for `layout-integrity`
- `agents/qa-tester.md` Confidence Summary Table — Extend with LAYOUT-* confidence entries

</code_context>

<specifics>
## Specific Ideas

### Standalone Spacing vs Reference-Based Spacing
Phase 9's DESIGN-01 compares spacing against a design reference image. LAYOUT-01 is the standalone version — no reference exists, so the agent infers "correct" from internal consistency. The key technique is sibling comparison: if 5 cards have 16px gaps and one has 24px, the outlier is flagged. This is a different methodology than "compare to reference" and produces different findings.

### Tiered Test Content for Overflow
The overflow testing uses a deliberate vocabulary of edge-case strings rather than random long text. Each string type targets a different overflow failure mode: Latin for basic length, German compounds for word-break, numbers for fixed-width containers, emails for form fields. This makes findings actionable — "card title overflows with German compound words" tells the developer exactly what to fix.

### Placeholder Detection Scope
Scanning extends beyond visible text to alt, title, aria-label, and meta tags. Placeholder content in these attributes is often missed during development but visible to users in tooltips, screen readers, and search results.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 11-layout-content-integrity*
*Context gathered: 2026-04-20*
