---
name: qa:check
description: Verify that the current phase's deliverables actually work. Tests what was just built before moving on.
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - Task
  - mcp__MCP_DOCKER__browser_*
---

<objective>
Quick verification that the current phase's work actually functions. This is a gate check, not a full QA pass.

When an orchestration system like GSD completes a phase, it knows the code was written - but not whether it works. This command:

1. Understands what the phase was supposed to deliver
2. Runs targeted browser tests to verify those deliverables
3. Reports pass/fail with enough detail to act on
4. Blocks phase completion if issues are found
</objective>

<arguments>
- `--url <base-url>` (required): The URL to test against (e.g., http://localhost:3000)
- `--phase <number>` (optional): Specific phase to verify. If omitted, infers from current GSD context.
</arguments>

<process>

<step name="gather-context">
**Gather Phase Context**

Look for phase information:

1. Check `.planning/ROADMAP.md` for current phase and its goal
2. Check `.planning/phases/{phase}/PLAN.md` for what was planned
3. Check `.planning/phases/{phase}/GOAL.md` or phase description in roadmap

Extract:
- **Phase goal**: What user-facing functionality should now exist?
- **Key deliverables**: What specific features/flows were implemented?
- **Acceptance criteria**: Any explicit criteria from the plan?

If no phase context found, ask the user what they want to verify.
</step>

<step name="determine-tests">
**Determine What to Test**

Based on phase context, identify testable outcomes:

| Phase Type | What to Verify |
|------------|----------------|
| New feature | Feature exists and basic flow works |
| Bug fix | The bug no longer reproduces |
| Refactor | Existing functionality still works |
| API work | Endpoints return expected responses |
| UI work | Components render and are interactive |
</step>

<step name="run-verification">
**Run Browser Verification**

Use the browser tools to:

1. Navigate to the relevant page(s)
2. Take a screenshot of initial state
3. Perform the core user flow
4. Check console for errors after each interaction
5. Verify expected outcomes are visible
6. Take screenshot of final state

Focus on:
- Does the core functionality exist and work?
- Does it work for the happy path?
- Are there any obvious errors (console, network)?

This is NOT a full QA pass. Don't exhaustively test edge cases unless something seems wrong.
</step>

<step name="report-results">
**Report Results**

**If verification passes:**
```
✅ Phase {N} Verification: PASSED

Verified:
- {deliverable 1}: Works
- {deliverable 2}: Works

Notes:
- {any observations for later}

Ready to proceed to next phase.
```

**If verification fails:**
```
❌ Phase {N} Verification: FAILED

Issues Found:
1. {Issue description}
   - Evidence: {what was observed}
   - Console errors: {if any}
   - Severity: {Blocker / Major / Minor}

Recommendation: Address these issues before proceeding.
```

**If partial pass:**
```
⚠️ Phase {N} Verification: PASSED WITH CONCERNS

Working:
- {deliverable 1}: Works
- {deliverable 2}: Works

Concerns:
- {issue that's not a blocker but should be noted}

Recommendation: Can proceed, but schedule follow-up for noted concerns.
```
</step>

<step name="escalation">
**Failure Escalation**

Track verification attempts within a session:

- **1st failure**: Report issues, expect fixes
- **2nd failure**: Report issues, note the pattern
- **3rd failure**: Escalate:

> "Phase {N} has failed verification three times. The repeated failures suggest the implementation approach may have a fundamental issue. Recommend pausing to review:
> - Is the technical approach sound?
> - Are there missing requirements or misunderstandings?
> - Should this be broken into smaller phases?"
</step>

<step name="write-report">
**Write Report File**

After completing verification, write a report to `.qa/reports/`:

1. Create directory if needed: `mkdir -p .qa/reports`
2. Write report to `.qa/reports/check-{phase}-{timestamp}.md`

Timestamp format: `YYYY-MM-DD-HHMMSS`

**Report template:**

```markdown
# QA Check: Phase {N} - {Phase Name}

**Date:** {YYYY-MM-DD HH:MM}
**URL:** {base url tested}
**Result:** {PASSED / FAILED / PASSED WITH CONCERNS}

## Phase Goal
{What this phase was supposed to deliver}

## Verification Summary

| Check | Result | Notes |
|-------|--------|-------|
| {item} | ✅/❌/⚠️ | {details} |

## Detailed Findings

### What Worked
{List of things that passed verification}

### Issues Found
{If any - with severity, evidence, console errors}

### Observations
{Things worth noting even if not failures}

## Console Output
```
{Any relevant console messages}
```

## Screenshots
{Reference any screenshots taken during verification}

## Recommendation
{Next steps - proceed, fix issues, or escalate}
```

Also append a summary line to `.qa/reports/HISTORY.md` (create if doesn't exist):

```markdown
| {date} | Phase {N} | {PASS/FAIL/CONCERN} | {one-line summary} |
```

This creates a running log of all QA checks for the project.
</step>

</process>

<qa-methodology>
## How to Test Like a Human

You're not just checking if things exist - you're verifying they work correctly.

**For every interaction:**
1. Take a screenshot before and after
2. Check console for errors (not just red errors - warnings matter too)
3. Note timing - did it feel responsive?
4. Look for visual anomalies
5. Verify keyboard accessibility (see below)

**Root cause thinking:**
Don't just report "button doesn't work." Dig into why:
- Is there a console error?
- Did the network request fail?
- Is the element actually clickable or covered by something?

**Confidence levels:**
- High: "Works correctly, tested happy path and saw expected behavior"
- Medium: "Works for tested case, but couldn't verify edge cases"
- Low: "Passed basic check, but something feels off"
</qa-methodology>

<accessibility>
## Baseline Accessibility Checks

These checks are part of standard QA - not a separate pass. While testing functionality, also verify:

### Keyboard Navigation (Required)
For every interactive element tested:
- **Tab order**: Can you Tab to it? Is the order logical?
- **Focus visible**: Is there a clear focus indicator (ring, outline, highlight)?
- **Activation**: Does Enter/Space activate buttons? Does Enter submit forms?
- **Escape**: Do modals/dropdowns close with Escape?
- **Arrow keys**: Do menus, tabs, and selects respond to arrows?

**How to test:** Before clicking with mouse, try the keyboard path first. Tab to the element, verify focus is visible, press Enter/Space.

### Form Accessibility (Required)
- **Labels**: Every input must have a visible label or aria-label
- **Error messages**: Are they associated with the field (aria-describedby)?
- **Required fields**: Indicated visually AND with aria-required or required attribute?
- **Focus management**: After error, does focus move to first invalid field?

### Images and Media (Required)
- **Alt text**: Do meaningful images have descriptive alt text?
- **Decorative images**: Do decorative images have alt="" (empty)?
- **Icons**: Do icon-only buttons have aria-label or screen reader text?

### Semantic HTML (Required)
- Buttons are `<button>` not `<div onclick>`
- Links are `<a>` with href, not `<span onclick>`
- Headings follow hierarchy (h1 → h2 → h3, not skipping levels)
- Lists use `<ul>/<ol>` not styled divs
- Tables have proper headers if presenting data

### ARIA Basics (Required)
- **Expanded/collapsed**: Accordions, dropdowns have aria-expanded
- **Current state**: Navigation indicates current page with aria-current
- **Live regions**: Dynamic content updates use aria-live (or role="alert" for errors)
- **Hidden content**: Visually hidden but present content uses proper technique (not display:none if needs to be read)

### What to Report

In the accessibility section of the report, note:
- ✅ "Form keyboard-navigable, all fields labeled"
- ⚠️ "Submit button reachable by Tab but no visible focus indicator"
- ❌ "Modal cannot be closed with Escape key"
- ❌ "Icon button has no accessible name"

**Severity guidance:**
- **Blocker**: Cannot complete task via keyboard at all
- **Major**: Can complete but significant barrier (no focus indicators, unlabeled form fields)
- **Minor**: Suboptimal but functional (focus order slightly off, missing aria-expanded)

### Out of Scope (For Now)

These require specialized tooling or real assistive technology:
- Screen reader testing (NVDA, VoiceOver, JAWS)
- Color contrast measurements (use automated tools)
- Screen magnification behavior
- Cognitive accessibility assessment
- Complex ARIA widget patterns

Flag these for follow-up if the feature is complex, but don't block on them in standard QA.
</accessibility>

<examples>
**After completing a login feature phase:**
```
/qa:check --url http://localhost:3000

> Checking Phase 3: User Authentication
>
> Verifying: Login form exists and accepts credentials
> ✓ Login page loads at /login
> ✓ Form accepts email and password
> ✓ Submit with valid credentials redirects to dashboard
> ✓ No console errors
>
> ✅ Phase 3 Verification: PASSED
> Ready to proceed.
```

**After a phase that broke something:**
```
/qa:check --url http://localhost:3000

> Checking Phase 5: Shopping Cart
>
> Verifying: Add to cart functionality
> ✗ "Add to Cart" button exists but clicking does nothing
>   - Console shows: "TypeError: Cannot read property 'id' of undefined"
>   - Network shows: No request sent to /api/cart
>
> ❌ Phase 5 Verification: FAILED
>
> Issue: Add to cart click handler has a bug - product ID not being passed
> Severity: Blocker
>
> Address before proceeding.
```
</examples>
