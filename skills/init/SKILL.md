---
name: qa:init
description: Initialize QA structure in a project by creating .qa directory and template spec
argument-hint: "path"
---

# /qa:init

Initialize QA testing structure in a project.

## Usage

```
/qa:init ~/Projects/my-app
```

Or if already in the project directory:
```
/qa:init .
```

## What This Does

1. Creates the `.qa/` directory in the specified project
2. Creates a `README.md` with quick reference for spec format
3. Creates a `smoke-test.md` template as a starting point

## Execution

1. Parse the path argument (default to current directory if not provided)
2. Create the directory structure:
   ```
   {project}/
   └── .qa/
       ├── README.md
       └── smoke-test.md
   ```
3. Confirm creation to the user

## Template: README.md

```markdown
# QA Specs

This directory contains test specifications for automated QA testing.

## Running Tests

```bash
# Run all specs
/qa:run --project .

# Run a specific spec
/qa:run --project . --spec smoke-test

# Run against a different URL
/qa:run --project . --spec smoke-test --url https://staging.example.com
```

## Writing Specs

Specs are Markdown files with the following structure:

- **Overview**: What is being tested and why
- **Base URL**: Default URL to test against
- **Personas**: User types to test as (with credentials)
- **Viewports**: Screen sizes to test
- **Test Scenarios**: Step-by-step tests with expected outcomes
- **Tags**: Label scenarios with `tags: [smoke, critical, regression, a11y]` for selective execution
- **Dependencies**: Use `depends_on: Scenario Name` when scenarios require prior scenarios to pass
- **Environments**: Define `## Environments` with profiles for local/staging/production
- **Accessibility Focus**: Add `## Accessibility Focus` section to trigger deep a11y testing
- **Edge Cases**: Specific edge cases to verify
- **Things to Watch For**: Performance, security, accessibility concerns

See `smoke-test.md` for an example.
```

## Template: smoke-test.md

```markdown
# Smoke Test

## Overview
Quick health check to verify the application is running and core functionality works.

## Base URL
http://localhost:3000

## Environments

### local
- **base_url**: http://localhost:3000

## Test Scenarios

### 1. Application Loads
tags: [smoke]
**Steps:**
1. Navigate to the home page

**Expected:**
- Page loads without errors
- No console errors
- Main content is visible

### 2. Navigation Works
tags: [smoke]
**Steps:**
1. Click each link in the main navigation

**Expected:**
- Each page loads
- No 404 errors
- No console errors

### 3. Core Feature Works
tags: [smoke]
**Steps:**
1. [Describe your app's core feature]

**Expected:**
- [Describe expected behavior]

## Things to Watch For
- Page load time should be under 3 seconds
- No JavaScript errors in console
- No failed network requests (except expected 401s for auth)
```

## After Initialization

Inform the user:
1. The `.qa/` directory has been created
2. They should customize `smoke-test.md` for their application
3. They can add more spec files as needed
4. Run tests with `/qa:run --project {path}`
5. Specs support tags, dependencies, data-driven scenarios, environments, and accessibility depth — see the [Spec Format docs](examples/SPEC-FORMAT.md) for full reference
