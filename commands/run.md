---
name: qa:run
description: Run QA tests against a web application using spec files or ad-hoc instructions
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
Run comprehensive QA tests against a web application. Can run from spec files or ad-hoc task descriptions.
</objective>

<arguments>
- `--project <path>` (optional): Path to project containing `.qa/` specs
- `--spec <name>` (optional): Name of specific spec file to run (without .md extension)
- `--url <base-url>` (optional): Base URL to test against (overrides spec URL)
- `<task>` (optional): Ad-hoc test description in quotes
</arguments>

<usage-modes>

**Mode 1: Run a spec file**
```
/qa:run --project ~/Projects/my-app --spec login
```
Runs the spec at `~/Projects/my-app/.qa/login.md`

**Mode 2: Run all specs in a project**
```
/qa:run --project ~/Projects/my-app
```
Runs all `.qa/*.md` specs in the project.

**Mode 3: Ad-hoc testing**
```
/qa:run --url http://localhost:3000 "Test that the homepage loads without console errors and all navigation links work"
```
Runs an ad-hoc test without a spec file.

**Mode 4: Override base URL**
```
/qa:run --project ~/Projects/my-app --spec login --url https://staging.example.com
```
Runs the login spec against staging instead of the URL in the spec.
</usage-modes>

<process>

<step name="parse-arguments">
**Parse Arguments**

Extract from user input:
1. **project**: Path to project with `.qa/` specs
2. **spec**: Specific spec file name
3. **url**: Base URL for testing
4. **task**: Ad-hoc test description

If neither spec nor task provided, ask what to test.
</step>

<step name="load-spec-or-task">
**Load Spec or Prepare Task**

If spec provided:
1. Read `{project}/.qa/{spec}.md`
2. Parse sections: Overview, Base URL, Personas, Viewports, Scenarios, Edge Cases, Things to Watch For
3. Use `--url` flag to override Base URL if provided

If ad-hoc task:
1. Verify URL was provided
2. Prepare to test based on task description
</step>

<step name="execute-tests">
**Execute Tests**

For each test scenario:

1. **Setup**: Navigate to starting point, establish baseline
2. **Execute**: Perform the steps described
3. **Verify**: Check expected outcomes
4. **Document**: Screenshot, console state, network activity

After each interaction:
- Take screenshot
- Check console for errors/warnings
- Note any performance concerns
- Look for visual anomalies
</step>

<step name="edge-cases">
**Test Edge Cases**

Unless spec says otherwise, try:
- Empty inputs on forms
- Very long text / special characters
- Rapid clicks / interruptions
- Back button / refresh mid-flow
- State persistence after refresh
</step>

<step name="compile-results">
**Compile Results**

Provide summary:

```
## QA Test Results: {spec name or "Ad-hoc Test"}

**URL**: {base url}
**Date**: {timestamp}

### Overall Result: {PASS / FAIL / PASS WITH CONCERNS}

### Scenarios Tested

| Scenario | Result | Notes |
|----------|--------|-------|
| {name} | ✅ Pass | {notes} |
| {name} | ❌ Fail | {reason} |
| {name} | ⚠️ Concern | {issue} |

### Issues Found
{detailed description of any failures}

### Observations
{things that worked but are worth noting}

### Recommendations
{suggested follow-ups}
```
</step>

<step name="write-report">
**Write Report File**

After completing the test run, write a report to `.qa/reports/`:

1. Create directory if needed: `mkdir -p .qa/reports`
2. Write report to `.qa/reports/run-{spec-name}-{timestamp}.md`
   - For ad-hoc tests use: `.qa/reports/run-adhoc-{timestamp}.md`

Timestamp format: `YYYY-MM-DD-HHMMSS`

**Report template:**

```markdown
# QA Run: {Spec Name or "Ad-hoc Test"}

**Date:** {YYYY-MM-DD HH:MM}
**URL:** {base url tested}
**Spec:** {spec file path or "Ad-hoc"}
**Result:** {PASSED / FAILED / PASSED WITH CONCERNS}

## Summary

| Metric | Count |
|--------|-------|
| Scenarios | {n} |
| Passed | {n} |
| Failed | {n} |
| Concerns | {n} |

## Scenarios

### {Scenario Name}
**Result:** ✅ Pass / ❌ Fail / ⚠️ Concern

**Steps Executed:**
1. {step and result}
2. {step and result}

**Expected:** {what should happen}
**Actual:** {what happened}

**Console:** {any relevant output}
**Network:** {any relevant requests/responses}

---

{Repeat for each scenario}

## Edge Cases Tested
{List edge cases and results}

## Issues Found

### {Issue Title}
**Severity:** Critical / Major / Minor
**Scenario:** {where found}
**Description:** {details}
**Evidence:** {console errors, screenshots, etc.}

## Observations
{Things worth noting even if not failures}

## Recommendations
{Next steps}
```

Also append a summary line to `.qa/reports/HISTORY.md` (create if doesn't exist):

```markdown
| {date} | {spec name} | {PASS/FAIL/CONCERN} | {scenarios passed}/{total} | {one-line summary} |
```
</step>

</process>

<qa-methodology>
## Deep QA Methodology

You test like a skeptical human, not a script.

### Before Each Test
- Take baseline screenshot
- Check console for pre-existing errors
- Understand what this feature is supposed to accomplish

### During Each Interaction
1. Screenshot before and after
2. Check console for errors, warnings, even info logs
3. Inspect network requests - did expected calls happen?
4. Assess timing - flag anything > 1 second
5. Look for visual anomalies
6. Test keyboard path, not just mouse (see Accessibility section)

### Pattern Recognition
Track what's been tested and what failed. If same test fails 3+ times:
> "This has failed repeatedly. The implementation approach may need reconsideration, not just another patch."

### Confidence Levels
- **High**: Tested happy path and key edge cases, all behaved as expected
- **Medium**: Works for tested scenarios, couldn't verify some areas
- **Low**: Passed basic check, but something feels off

### Spec Skepticism
If observed behavior differs from spec:
- Note the discrepancy
- Assess whether app behavior seems intentional or buggy
- Ask for clarification if ambiguous
</qa-methodology>

<accessibility>
## Baseline Accessibility Checks

These are part of standard QA - not a separate pass. While testing functionality, also verify:

### Keyboard Navigation (Required)
For every interactive element:
- **Tab order**: Can you Tab to it? Is the order logical?
- **Focus visible**: Is there a clear focus indicator?
- **Activation**: Does Enter/Space work on buttons? Enter on forms?
- **Escape**: Do modals/dropdowns close with Escape?
- **Arrow keys**: Do menus, tabs, selects respond to arrows?

**How to test:** Before clicking with mouse, try keyboard first. Tab to element, verify focus visible, press Enter/Space.

### Form Accessibility (Required)
- Labels associated with every input
- Error messages connected to fields (aria-describedby)
- Required fields indicated (visually and programmatically)
- Focus moves to first invalid field after error

### Images and Media (Required)
- Meaningful images have descriptive alt text
- Decorative images have empty alt=""
- Icon-only buttons have aria-label

### Semantic HTML (Required)
- Buttons are `<button>`, not clickable divs
- Links are `<a href>`, not clickable spans
- Headings follow hierarchy (h1 → h2 → h3)
- Lists use proper list elements

### ARIA Basics (Required)
- Expandable elements have aria-expanded
- Current page indicated with aria-current
- Dynamic updates use aria-live regions
- Modal focus is trapped appropriately

### Reporting

Include in the Accessibility section of report:
- ✅ "All form fields labeled, keyboard navigable"
- ⚠️ "Button works but no visible focus indicator"
- ❌ "Cannot Tab to submit button"
- ❌ "Modal has no escape key handler"

**Severity:**
- **Blocker**: Task impossible via keyboard
- **Major**: Significant barrier (no focus indicators, unlabeled fields)
- **Minor**: Suboptimal but functional (focus order slightly off)

### Out of Scope
These need specialized tools - flag for follow-up but don't block:
- Screen reader testing
- Color contrast measurement
- Complex ARIA widget patterns
</accessibility>

<spec-format>
## Spec File Format

Specs are Markdown in `.qa/` directory:

```markdown
# Feature Name

## Overview
What this feature does and why it matters.

## Base URL
http://localhost:3000

## Personas (optional)
- unauthenticated user
- logged-in user (test@example.com / password123)

## Viewports (optional)
- desktop (1280x720)
- mobile (375x667)

## Test Scenarios

### Scenario 1: Happy Path
**Steps:**
1. Navigate to /page
2. Click button
3. Fill form
4. Submit

**Expected:**
- Success message appears
- No console errors
- Redirects to /next

### Scenario 2: Error Handling
...

## Edge Cases to Verify
- Empty input behavior
- Invalid data handling

## Things to Watch For
- Performance thresholds
- Security concerns
- Accessibility requirements
```
</spec-format>
