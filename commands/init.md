---
name: qa:init
description: Initialize QA structure in a project by creating .qa directory and template spec
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

<objective>
Initialize QA testing structure in a project by creating the `.qa/` directory and starter templates.
</objective>

<arguments>
- `<path>` (optional): Path to the project. Defaults to current directory.
</arguments>

<process>

<step name="create-structure">
**Create Directory Structure**

```bash
mkdir -p {project}/.qa
```

Create the following files:
- `{project}/.qa/README.md` - Quick reference
- `{project}/.qa/smoke-test.md` - Starter template
</step>

<step name="create-readme">
**Create README.md**

Write to `{project}/.qa/README.md`:

```markdown
# QA Specs

Test specifications for automated QA testing with `/qa:run`.

## Running Tests

```bash
# Run all specs
/qa:run --project .

# Run a specific spec
/qa:run --project . --spec smoke-test

# Run against a different URL
/qa:run --project . --url https://staging.example.com
```

## Quick Verification

After completing a feature or phase:
```bash
/qa:check --url http://localhost:3000
```

## Writing Specs

Each spec is a Markdown file with:
- **Overview**: What's being tested
- **Base URL**: Default URL
- **Test Scenarios**: Steps and expected outcomes
- **Edge Cases**: Specific cases to try
- **Things to Watch For**: Performance, security, accessibility

See `smoke-test.md` for an example.

## Spec Format

```markdown
# Feature Name

## Base URL
http://localhost:3000

## Test Scenarios

### Scenario Name
**Steps:**
1. Do something
2. Do something else

**Expected:**
- Expected outcome
- No console errors
```
```
</step>

<step name="create-template">
**Create smoke-test.md**

Write to `{project}/.qa/smoke-test.md`:

```markdown
# Smoke Test

## Overview
Quick health check to verify the application is running and core functionality works.

## Base URL
http://localhost:3000

## Test Scenarios

### 1. Application Loads
**Steps:**
1. Navigate to the home page

**Expected:**
- Page loads without errors
- No console errors
- Main content is visible

### 2. Navigation Works
**Steps:**
1. Click each link in the main navigation

**Expected:**
- Each page loads
- No 404 errors
- No console errors

### 3. Core Feature Works
**Steps:**
1. [TODO: Describe your app's core feature]

**Expected:**
- [TODO: Describe expected behavior]

## Things to Watch For
- Page load time should be under 3 seconds
- No JavaScript errors in console
- No failed network requests (except expected 401s)

## Known Issues
None currently tracked.
```
</step>

<step name="confirm">
**Confirm Creation**

Tell the user:

```
✅ QA structure initialized at {project}/.qa/

Created:
- .qa/README.md - Quick reference guide
- .qa/smoke-test.md - Starter template (customize this!)

Next steps:
1. Edit .qa/smoke-test.md for your application
2. Add more specs as needed (e.g., login.md, checkout.md)
3. Run tests with: /qa:run --project {project}
4. Quick verify with: /qa:check --url http://localhost:3000
```
</step>

</process>
