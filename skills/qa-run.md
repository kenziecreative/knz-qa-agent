---
name: run
description: Run QA tests against a web application using spec files or ad-hoc instructions
arguments: "[--project <path>] [--spec <name>] [--url <base-url>] [--tag <tag>] [--env <profile>] [task]"
allowed-tools: Read, Write, Bash, Grep, Glob, Agent, mcp__playwright__*
---

# /qa:run

Run QA tests against a web application.

## Usage Modes

### Mode 1: Run a spec file
```
/qa:run --project ~/Projects/my-app --spec login
```
This runs the spec at `~/Projects/my-app/.qa/login.md`

### Mode 2: Run all specs in a project
```
/qa:run --project ~/Projects/my-app
```
This runs all `.qa/*.md` specs in the project.

### Mode 3: Ad-hoc testing
```
/qa:run --url http://localhost:3000 "Test that the homepage loads without console errors and all navigation links work"
```
This runs an ad-hoc test without a spec file.

### Mode 4: Override base URL
```
/qa:run --project ~/Projects/my-app --spec login --url https://staging.example.com
```
This runs the login spec but against staging instead of the URL in the spec.

### Mode 5: Run filtered by tag
```
/qa:run --project ~/Projects/my-app --tag smoke
```
Runs only scenarios tagged with `smoke` across all specs.

### Mode 6: Run with environment profile
```
/qa:run --project ~/Projects/my-app --spec checkout --env staging
```
Runs the checkout spec using the `staging` environment profile.

### Mode 7: Combined filtering
```
/qa:run --project ~/Projects/my-app --tag critical --env staging
```
Runs only `critical`-tagged scenarios against the staging environment.

## Parsing Arguments

From the user's input, extract:

1. **project**: Path to the project containing `.qa/` specs (optional)
2. **spec**: Name of a specific spec file to run (without .md extension) (optional)
3. **url**: Base URL to test against (optional, can come from spec)
4. **task**: Ad-hoc test description in quotes (optional, used when no spec)
5. **tag**: Tag name to filter scenarios (optional). Only scenarios with this tag will run. Multiple tags can be comma-separated (e.g., `--tag smoke,critical`) — scenario must match ANY listed tag (OR logic).
6. **env**: Environment profile name to activate (optional). Selects the matching profile from the spec's `## Environments` section.

## Execution Flow

### If a spec is provided:

1. Read the spec file from `{project}/.qa/{spec}.md`
2. Parse the spec to understand:
   - Base URL (can be overridden by --url flag)
   - Personas to test as
   - Viewports to test at
   - Test scenarios
   - Edge cases
   - Things to watch for
3. **Environment resolution**: If `--env` flag provided, find the matching profile in `## Environments`. Extract base_url, credentials, timeouts. If profile not found, warn user and fall back to spec defaults. If no `--env` flag and spec has `## Environments`, use the first profile. If spec has no `## Environments`, use `## Base URL` (backward compatible).
4. **Tag filtering**: If `--tag` flag provided, filter scenario list to only include scenarios whose `tags:` field contains at least one of the requested tags. If no scenarios match, inform user and exit. If no `--tag` flag, run all scenarios.
5. **Dependency resolution**: After filtering, check `depends_on:` fields. If a filtered scenario depends on a scenario that was filtered OUT, include that dependency scenario automatically (it must run for the dependent to work). Inform the agent which scenarios were auto-included as dependencies.
6. Launch the `qa-tester` agent with the spec content and instructions
7. If spec defines viewports: Test each scenario at each viewport (default: mobile 375px, tablet 768px, desktop 1280px if spec doesn't specify)
8. If spec defines personas: Test scenarios as each persona, handling auth flows between switches
9. The agent will work through the spec systematically

### If ad-hoc task is provided:

1. Verify a URL was provided
2. Launch the `qa-tester` agent with the URL and task description
3. The agent will use its judgment to thoroughly test the described functionality

### If neither spec nor task:

Ask the user what they want to test.

## Agent Invocation

Spawn the `qa-tester` agent with a prompt structured like:

```
## Test Assignment

**Mode**: [Spec-based / Ad-hoc]
**Base URL**: {url}
**Project**: {project or "N/A"}

{If spec-based:}
## Spec Content
{full content of the spec file}

{If environment profile active:}
## Active Environment
**Profile**: {profile_name}
**Base URL**: {profile.base_url}
**Credentials**: {profile.credentials or "use spec defaults"}
**Timeouts**: {profile.timeouts or "use defaults"}

{If tag filtering active:}
## Tag Filter
**Active tags**: {tag_list}
**Scenarios to run**: {filtered scenario names}
**Auto-included dependencies**: {dependency scenario names, if any}

{If spec has dependencies:}
## Execution Notes
- Scenarios have dependencies. If a dependency fails, skip dependent scenarios.
- Process scenarios in listed order — dependencies resolve top-to-bottom.

{If spec has data-driven scenarios:}
## Execution Notes
- Some scenarios have data sets. Run each data set as a separate variant.
- Report per-variant: "Scenario X [variant-name]: PASS/FAIL"

{If ad-hoc:}
## Task
{user's task description}

## Instructions
Execute this test assignment following your QA methodology. Remember to:
- Take screenshots at key points
- Check console after every interaction
- Monitor network requests
- Try relevant edge cases
- Report findings with full context
- Escalate if you see repeated failures
- Assign a confidence level to every finding:
  - **High**: Definitely real AND impactful (certainty x severity)
  - **Medium**: Likely real but impact uncertain, OR definitely real but low severity
  - **Low**: Possible issue, needs further investigation or human judgment
- Group findings in your summary by confidence: "Definite Issues" (high), "Likely Issues" (medium), "Possible Issues" (low)
- Include a brief rationale for each confidence assignment
- Show ALL findings — do not filter out low-confidence items
- Monitor network requests during every interaction:
  - Flag failed requests (4xx/5xx) and correlate with what the UI shows
  - Flag slow API calls (> 1s default) — note the endpoint and response time
  - Note any mixed content (HTTP on HTTPS) or over-fetching patterns
  - Include network findings in your confidence-grouped summary
- Test across viewports: Run each scenario at each viewport defined in the spec (default: mobile/tablet/desktop). Check for responsive issues, touch target sizes at mobile, and horizontal scrolling
- Test across personas: If spec defines personas, run scenarios as each persona. Handle login/logout between persona switches. Verify role-based content visibility
- When testing forms, apply exhaustive form testing methodology:
  - Test all validation states (empty, invalid, boundary values, injection strings)
  - Verify error messages are visible, associated with correct fields, and helpful
  - Test all submission states (loading indicator, success feedback, error handling, double-submit prevention)
  - Check field type attributes and autofill behavior
  - For multi-step forms: test back/forward navigation and data persistence across steps
- Check for SPA-specific issues during navigation:
  - Verify URL and page title update on route changes
  - Test browser back/forward through client-side routes
  - Check state persistence across navigation (form data, filters, selections)
  - Watch for hydration mismatches (console warnings, content flashing)
- Test error recovery for critical flows:
  - Use Playwright route interception to simulate API failures when spec requests error testing
  - Verify error messages are user-friendly (no raw stack traces)
  - Check that recovery paths exist (retry button, navigate away, dismiss error)
- Perform structured accessibility checks on every page/flow:
  - Focus management: When modals/dialogs open, verify focus moves in, is trapped (Tab cycles within), and returns to trigger on close
  - Skip links: Tab once on page load — check for skip link, activate it, verify focus moves to target
  - Heading hierarchy: Audit h1-h6 tree — single h1, no skipped levels. Flag component-internal gaps as Medium confidence
  - Landmark regions: Verify main, banner, contentinfo, navigation roles exist
  - Page title: Verify title updates on SPA route changes
  - Lang attribute: Check html element has valid lang attribute
  - aria-live: When dynamic content appears, verify aria-live region exists and receives content
  - Touch targets: At mobile viewport, measure all interactive elements — flag any below 44x44px
  - Color sole indicator: When testing state changes, verify non-color indicators exist (text, icon, border)
  - Reduced motion: Use emulateMedia to set prefers-reduced-motion: reduce, verify animations stop
  - Zoom: At desktop viewport, simulate 200% text zoom and check for overflow or content loss
  - Form errors: After submission with errors, check for error summary, links to fields, aria-invalid on errored fields, and focus on first error

Begin testing.
```

## Output

The agent will provide:
- Step-by-step progress through the test
- Screenshots as evidence
- Console/network observations
- Final summary with:
  - Overall result (Pass / Fail / Pass with Concerns)
  - Definite Issues (high confidence) — these need immediate attention
  - Likely Issues (medium confidence) — should be investigated
  - Possible Issues (low confidence) — worth noting for future review
  - Observations (things that worked but are worth noting)
  - Recommendations (if any)
  - Network observations (slow calls, failed requests, over-fetching, mixed content)
  - SPA issues (route sync, history, state persistence, hydration)
  - Error recovery observations (how the app handles failures)
  - Accessibility findings (focus management, page structure, skip links, landmarks, touch targets, reduced motion, zoom, form error patterns)

## Examples

**Run the checkout spec on local:**
```
/qa:run --project ~/Projects/store --spec checkout
```

**Run all specs against staging:**
```
/qa:run --project ~/Projects/store --url https://staging.store.com
```

**Quick ad-hoc test:**
```
/qa:run --url http://localhost:3000 "Verify the search function returns relevant results and handles empty queries gracefully"
```

**Test a specific flow on mobile viewport:**
```
/qa:run --url http://localhost:3000 "Test the checkout flow on mobile viewport (375x667), ensure all buttons are tappable and forms are usable"
```

**Run only smoke tests:**
```
/qa:run --project ~/Projects/store --tag smoke
```

**Run critical tests on staging:**
```
/qa:run --project ~/Projects/store --tag critical --env staging
```

**Run multiple tags:**
```
/qa:run --project ~/Projects/store --tag smoke,regression
```
