---
name: gen
description: Generate a QA spec from a description, existing documentation, or by exploring the application
arguments: "<spec-name> [--from <source>] [description]"
---

# /qa:gen

Generate a QA test specification.

## Usage Modes

### Mode 1: Generate from description
```
/qa:gen login "User authentication flow with email/password, including forgot password"
```

### Mode 2: Generate from existing documentation
```
/qa:gen checkout --from ~/Projects/store/docs/checkout-flow.md
```

### Mode 3: Generate by exploring the app
```
/qa:gen user-profile --from http://localhost:3000/profile "Explore the user profile page and generate a spec for testing it"
```

## Parsing Arguments

From the user's input, extract:

1. **spec-name**: Name for the new spec file (required)
2. **--from**: Source to generate from (optional)
   - If a file path: Read and use as reference
   - If a URL: Explore the page and infer testable behaviors
3. **description**: Natural language description of what to cover (optional)

## Generation Process

### From Description Only

1. Create a well-structured spec based on the description
2. Include standard sections:
   - Overview (from description)
   - Base URL (placeholder)
   - Test scenarios (inferred from description)
   - Common edge cases for that type of feature
   - Standard things to watch for
3. Mark areas that need human review with `[TODO: ...]`

### From Documentation

1. Read the source document
2. Extract:
   - Feature purpose and intent
   - User flows described
   - Edge cases mentioned
   - Acceptance criteria
3. Transform into spec format
4. Mark assumptions with `[VERIFY: ...]`

### From Live URL

1. Use the qa-tester agent to explore the URL
2. Identify:
   - Page structure and components
   - Interactive elements (forms, buttons, links)
   - Visible states and flows
3. Generate a spec that covers observed functionality
4. Mark as exploratory with `[OBSERVED: ...]` for behaviors that should be verified

## Output

Write the generated spec to the project's `.qa/` directory (prompt for project path if not clear).

Structure the output as:

```markdown
# {Feature Name}

## Overview
{Generated overview}

## Base URL
{URL from --from flag or [TODO: Set base URL]}

## Viewports
- Mobile: 375x812
- Tablet: 768x1024
- Desktop: 1280x800

## Personas
### Default User
- **Role:** Regular user
- **Auth:** [TODO: Set login credentials or auth method]

### Admin (if applicable)
- **Role:** Administrator
- **Auth:** [TODO: Set admin credentials or auth method]

## Test Scenarios

### 1. {Scenario Name}
{Generated scenario}

...

## Edge Cases to Verify
{Standard edge cases for this feature type}

### Form Testing (if forms are present)
- Empty submission of all required fields
- Invalid formats for typed fields (email, phone, date)
- Boundary values (min/max length, special characters)
- Rapid double-submit
- Multi-step navigation (if applicable): forward, backward, data persistence

## Things to Watch For
{Standard concerns plus any inferred from source}

---
*This spec was generated and should be reviewed for accuracy.*
*Items marked [TODO], [VERIFY], or [OBSERVED] need human attention.*
```

## Examples

**Generate auth spec from scratch:**
```
/qa:gen auth "Login, logout, password reset, and session management"
```

**Generate from a PRD:**
```
/qa:gen search --from ./docs/search-feature-prd.md
```

**Generate by exploring:**
```
/qa:gen settings --from http://localhost:3000/settings "Document all the settings options and generate tests for each"
```

## After Generation

Inform the user:
1. The spec has been created at `.qa/{spec-name}.md`
2. They should review items marked with [TODO], [VERIFY], or [OBSERVED]
3. Customize edge cases and thresholds for their specific needs
4. Run with `/qa:run --spec {spec-name}` when ready
