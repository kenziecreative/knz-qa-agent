# QA Spec Format

Test specifications are written in Markdown and live in the `.qa/` directory of your project.

## Basic Structure

```markdown
# [Feature Name]

## Overview
Brief description of what this feature does and why it matters.

## Base URL
http://localhost:3000

## Personas (optional)
- unauthenticated user
- logged-in user (use test@example.com / testpass123)
- admin user (use admin@example.com / adminpass123)

## Viewports (optional)
- desktop (1280x720)
- mobile (375x667)

## Prerequisites (optional)
- User must have at least one item in cart
- Database should be seeded with test products

## Environments (optional)
### local
base_url: http://localhost:3000
credentials: test@example.com / localpass
timeouts: {}
flags: {}

### staging
base_url: https://staging.example.com
credentials: test@staging.example.com / $STAGING_PASSWORD
timeouts:
  api: 3000
flags:
  feature_x: true

## Accessibility Focus (optional)
- form-accessibility
- focus-management

## Test Scenarios

### Scenario 1: [Name]
tags: [smoke, critical]
Description of what to test.

**Steps:**
1. Navigate to /page
2. Click the "Action" button
3. Fill in the form with valid data
4. Submit

**Expected:**
- Success message appears
- User is redirected to /dashboard
- No console errors
- API call to /api/resource returns 200

### Scenario 2: [Name]
tags: [regression]
depends_on: Scenario 1
Description of a dependent scenario.

**Steps:**
1. Verify data persists from Scenario 1
2. Perform follow-up action

**Expected:**
- Follow-up state is correct

## Edge Cases to Verify
- What happens with empty input?
- What happens with very long input?
- What if the user double-clicks submit?

## Things to Watch For
- Performance: Page should load in under 2 seconds
- Security: Auth token should not appear in URL
- Accessibility: Form should be keyboard-navigable

## Known Issues (optional)
- #123: Submit button occasionally unresponsive on mobile (under investigation)
```

## Minimal Spec Example

For quick tests, you can use a minimal format:

```markdown
# Login Flow

## Base URL
http://localhost:3000

## Test Scenarios

### Happy Path
1. Go to /login
2. Enter valid credentials (test@example.com / password123)
3. Click "Sign In"

**Expected:**
- Redirects to /dashboard
- Shows welcome message with user's name
- No console errors
```

## Section Reference

### Overview
What is this feature? Why does it exist? This context helps the agent make better judgments about whether observed behavior is correct.

### Base URL
The root URL for the application. Can be overridden at runtime with `--url` flag. When an `## Environments` section is present, the active environment's `base_url` takes precedence over this value.

### Personas
Different user types to test as. Each should include credentials if authentication is needed. The agent will run through scenarios for each persona unless specific scenarios are marked for specific personas.

### Viewports
Screen sizes to test. The agent will run through scenarios at each viewport size. Common presets:
- desktop: 1280x720
- tablet: 768x1024
- mobile: 375x667

### Prerequisites
State that must exist before testing. This might be:
- Data in the database
- User account state
- Feature flags enabled
- External services running

### Environments (v2)

Named environment profiles for running the same spec against different targets (local, staging, production). Each profile is a subsection under `## Environments`.

**Profile fields:**

- `base_url` — overrides the spec's `## Base URL`
- `credentials` — credential values for this environment's personas
- `timeouts` — overrides default network thresholds (optional)
- `flags` — feature flags or test behavior flags (optional)

**Default behavior:** When no `--env` flag is passed, the first profile listed is used. If no `## Environments` section is present, the spec falls back to `## Base URL` (full backward compatibility).

**Secrets:** Credentials in specs should use placeholder values or environment variable references (e.g., `$STAGING_PASSWORD`). Real secrets should be provided via environment variables or `.qa/env.local` (gitignored — never commit real passwords).

**Example:**

```markdown
## Environments

### local
base_url: http://localhost:3000
credentials: test@example.com / devpassword
timeouts: {}
flags: {}

### staging
base_url: https://staging.example.com
credentials: qauser@staging.example.com / $STAGING_PASSWORD
timeouts:
  api: 3000
flags:
  feature_flags_enabled: true

### production
base_url: https://example.com
credentials: monitoring@example.com / $PROD_MONITOR_PASSWORD
timeouts:
  api: 5000
flags: {}
```

Select a profile at runtime: `/qa:run --env staging`

### Accessibility Focus (v2)

Controls the depth of accessibility testing. When present, the agent runs Tier 2 structured accessibility checks (the full methodology). When absent, only baseline Tier 1 checks run.

**Behavior:**

- Absent: Tier 1 only (keyboard navigation, focus indicators, alt text, color contrast, form labels)
- Present with no items listed: Full Tier 2 (all areas)
- Present with specific areas listed: Tier 2 for those areas only

**Available focus areas:**

- `focus-management` — modal focus trapping, skip links, aria-live regions
- `page-structure` — heading hierarchy, landmarks, page title, lang attribute
- `interactive-elements` — touch targets, color sole indicator, reduced motion, zoom
- `form-accessibility` — error summary, error links, aria-invalid, focus on first error

**Example (full Tier 2):**

```markdown
## Accessibility Focus
```

**Example (specific areas):**

```markdown
## Accessibility Focus
- focus-management
- form-accessibility
```

### Test Scenarios
The core of the spec. Each scenario should have:
- A descriptive name
- Steps (natural language, not code)
- Expected results (what should happen)

**Scenario-level v2 fields:**

- `tags:` — categorization labels for filtering (see Tags section)
- `depends_on:` — name of another scenario this one requires to have passed first (see Scenario Dependencies section)
- `## Data Sets` — subsection for data-driven variants (see Data-Driven Scenarios section)

### Tags (v2)

Tags are comma-separated labels applied to individual scenarios using the `tags:` field immediately after the scenario heading.

**Syntax:**

```markdown
### Scenario Name
tags: [smoke, critical]
```

**Standard tags with defined meanings:**

| Tag | Meaning |
| --- | ------- |
| `smoke` | Quick health check, run frequently (fast, critical-path only) |
| `critical` | Must-pass for deployment — a failure here blocks release |
| `regression` | Verifies previously broken functionality that was fixed |
| `a11y` | Accessibility-focused scenario requiring extra attention |

**Custom tags:** Any freeform string is valid as a tag. Custom tags are useful for grouping (e.g., `checkout`, `admin-only`, `needs-seeding`).

**Multiple tags:** Separate with commas. Order does not matter.

**Filtering:** Use `/qa:run --tag smoke` to run only scenarios with that tag. The agent does not filter — the qa-run skill handles filtering before the agent is invoked.

**Example:**

```markdown
### Successful Login
tags: [smoke, critical]
...

### Token Expiry Handling
tags: [regression, security]
...
```

### Scenario Dependencies (v2)

The `depends_on:` field links scenarios so that if a prerequisite fails, the dependent scenario is automatically skipped.

**Syntax:**

```markdown
### Dependent Scenario
depends_on: Prerequisite Scenario Name
```

**Behavior:**

- If the referenced scenario PASSED: the dependent scenario runs normally
- If the referenced scenario FAILED: the dependent scenario is SKIPPED with a note: "Skipped: dependency '[name]' failed — [failure reason]"
- If the referenced scenario hasn't run yet: it is run first before proceeding
- Dependencies reference scenarios by their exact name (the heading text after `###`)

**Chaining:** Single-level dependencies are supported. Scenario A depends on Scenario B; Scenario B depends on Scenario C. Run order resolves naturally top-to-bottom in the spec.

**Rationale:** Prevents noise from cascading failures. If "Add to Cart" fails because "Successful Login" failed, there's no value in also failing the checkout flow — a skip with a clear explanation is more useful than a false secondary failure.

**Example:**

```markdown
### Successful Login
tags: [smoke, critical]

**Steps:**
...

### Session Persistence
depends_on: Successful Login
tags: [regression]

**Steps:**
...
```

### Data-Driven Scenarios (v2)

When a scenario should run with multiple sets of input values, use a `## Data Sets` subsection. The agent runs the scenario once per data set and reports results per variant.

**Syntax:**

```markdown
### Scenario Name
tags: [regression]

**Steps:**
1. Navigate to /login
2. Enter email: {email}
3. Enter password: {password}
4. Click "Sign In"

**Expected:**
- {expected_result}

## Data Sets

### valid-user
email: test@example.com
password: password123
expected_result: Redirects to /dashboard

### expired-user
email: expired@example.com
password: password123
expected_result: Shows "Your account has expired" message

### locked-user
email: locked@example.com
password: password123
expected_result: Shows "Account locked" message with support contact
```

**Variable substitution:** Placeholders in `{variable_name}` format within steps and expected results are replaced with corresponding data set values before execution.

**Reporting:** Results are reported per variant:

- `Scenario Name [valid-user]: PASS`
- `Scenario Name [expired-user]: FAIL — Expected "account expired" message, got redirect to /dashboard`
- `Scenario Name [locked-user]: PASS`

**Partial failures:** If one variant fails, the remaining variants still run. The parent scenario is marked FAIL if any variant fails, with variant details listed.

**Grouping:** In the summary, all variants of a data-driven scenario are grouped under the parent scenario name.

### Edge Cases to Verify
Specific edge cases the agent should try. If omitted, the agent will use its judgment to try common edge cases.

### Things to Watch For

Specific concerns beyond basic functionality:

- Performance thresholds
- Security requirements
- Accessibility standards
- Browser compatibility notes

### Known Issues
Issues that are already tracked. This prevents the agent from re-reporting known problems and helps it distinguish between "new bug" and "known issue."

## Tips for Writing Good Specs

1. **Be specific about expected outcomes** - "Works correctly" is vague. "Shows success toast, redirects to /dashboard within 2 seconds, and API returns 201" is testable.

2. **Include the why** - If you explain why something should work a certain way, the agent can make better judgments about ambiguous situations.

3. **Note what's critical vs. nice-to-have** - If certain failures are blockers vs. minor issues, say so.

4. **Keep specs focused** - One feature per spec file. A spec for "the entire app" is too broad.

5. **Version your specs** - They should evolve with the feature. Outdated specs cause confusion.

6. **Use tags strategically** - A small set of `smoke` + `critical` scenarios enables fast pre-deploy checks without running the full suite.

7. **Use depends_on sparingly** - Only declare dependencies where a failure would genuinely make the dependent scenario meaningless to run. Not every scenario needs a dependency.

## Backward Compatibility

All v1 specs continue to work without modification. The v2 fields (`tags:`, `depends_on:`, `## Data Sets`, `## Environments`, `## Accessibility Focus`) are all optional. A spec with none of these fields behaves exactly as before.

## File Naming Convention

```
.qa/
├── auth/
│   ├── login.md
│   ├── logout.md
│   └── password-reset.md
├── checkout/
│   ├── cart.md
│   ├── payment.md
│   └── confirmation.md
└── smoke-test.md
```

Use directories to organize by feature area. Use `smoke-test.md` for quick overall health checks.
