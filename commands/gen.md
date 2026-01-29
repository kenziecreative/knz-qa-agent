---
name: qa:gen
description: Generate a QA spec from a description, existing documentation, or by exploring the application
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
Generate a QA test specification from various sources: natural language description, existing documentation, or by exploring a live application.
</objective>

<arguments>
- `<spec-name>` (required): Name for the new spec file
- `--from <source>` (optional): Source to generate from (file path or URL)
- `--project <path>` (optional): Project to save spec to. Defaults to current directory.
- `<description>` (optional): Natural language description of what to cover
</arguments>

<usage-modes>

**Mode 1: Generate from description**
```
/qa:gen login "User authentication flow with email/password, including forgot password"
```

**Mode 2: Generate from existing documentation**
```
/qa:gen checkout --from ./docs/checkout-flow.md
```

**Mode 3: Generate by exploring the app**
```
/qa:gen user-profile --from http://localhost:3000/profile "Explore the user profile page and generate a spec"
```
</usage-modes>

<process>

<step name="parse-input">
**Parse Input**

Extract:
1. **spec-name**: Required name for the spec file
2. **--from**: Optional source (file path or URL)
3. **--project**: Where to save (default: current directory)
4. **description**: Optional description of coverage
</step>

<step name="gather-source">
**Gather Source Material**

**If --from is a file path:**
1. Read the documentation file
2. Extract feature descriptions, user flows, acceptance criteria
3. Note any edge cases or requirements mentioned

**If --from is a URL:**
1. Navigate to the URL
2. Take screenshot
3. Identify page structure, interactive elements, forms
4. Note visible states and user flows

**If description only:**
1. Use description to understand scope
2. Infer typical scenarios for that type of feature
</step>

<step name="generate-spec">
**Generate Spec**

Create spec with standard structure:

```markdown
# {Feature Name}

## Overview
{Generated from source or description}

## Base URL
{From --from URL or placeholder: http://localhost:3000}

## Test Scenarios

### 1. {Primary Happy Path}
**Steps:**
{Generated steps}

**Expected:**
{Generated expectations}

### 2. {Secondary Flow}
...

### 3. {Error Handling}
...

## Edge Cases to Verify
{Standard edge cases for this feature type}

## Things to Watch For
{Performance, security, accessibility concerns}

---
*Generated spec - review for accuracy*
*Items marked [TODO] or [VERIFY] need attention*
```
</step>

<step name="mark-uncertainties">
**Mark Uncertainties**

Use markers for items needing human review:

- `[TODO: ...]` - Placeholder that must be filled in
- `[VERIFY: ...]` - Assumption that should be confirmed
- `[OBSERVED: ...]` - Behavior seen during exploration (might not be intentional)
</step>

<step name="save-spec">
**Save Spec**

1. Ensure `.qa/` directory exists: `mkdir -p {project}/.qa`
2. Write spec to `{project}/.qa/{spec-name}.md`
3. Confirm to user
</step>

<step name="confirm">
**Confirm Creation**

```
✅ Generated spec: .qa/{spec-name}.md

Source: {description / file path / URL exploration}

Scenarios generated:
- {scenario 1}
- {scenario 2}
- ...

⚠️ Review needed:
- {count} [TODO] items to fill in
- {count} [VERIFY] items to confirm
- {count} [OBSERVED] behaviors to validate

Next: Review the spec, then run with /qa:run --spec {spec-name}
```
</step>

</process>

<generation-guidelines>

## For Different Feature Types

**Authentication:**
- Login happy path
- Invalid credentials
- Empty form submission
- Session persistence
- Logout flow
- Password reset (if applicable)
- Edge: SQL injection, XSS attempts

**Forms:**
- Valid submission
- Validation errors for each field
- Empty submission
- Boundary values (min/max length)
- Special characters
- Edge: rapid submission, back button

**Search:**
- Results returned for valid query
- No results handling
- Empty query
- Special characters in query
- Pagination (if applicable)
- Edge: very long queries

**E-commerce/Checkout:**
- Add to cart
- Update quantity
- Remove item
- Apply discount code
- Payment flow
- Edge: empty cart checkout, expired session

## Standard Edge Cases

Always consider:
- Empty/null inputs
- Very long strings (1000+ chars)
- Special characters: `<script>`, `'; DROP TABLE`, emoji
- Rapid repeated actions
- Browser back/forward
- Page refresh mid-flow
- Multiple tabs

## Standard Things to Watch For

- **Performance**: Page load < 3s, API calls < 1s
- **Security**: No tokens in URLs, proper input escaping
- **Accessibility**: Keyboard navigation, focus indicators, labels
- **Console**: No errors, minimize warnings
</generation-guidelines>
