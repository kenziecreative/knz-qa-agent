---
name: report
description: Generate a summary report from recent QA test runs
arguments: "[--project <path>] [--format <format>]"
---

# /qa:report

Generate a summary report from QA testing.

## Usage

```
/qa:report --project ~/Projects/my-app
/qa:report --project ~/Projects/my-app --format markdown
/qa:report --project ~/Projects/my-app --format json
```

## Purpose

After running multiple specs or test sessions, this compiles findings into a structured report suitable for:
- Sharing with the team
- Creating issues/tickets
- Documentation

## Report Sections

### Executive Summary
- Total specs run
- Overall pass/fail/concern counts
- Critical issues found

### Detailed Findings

For each spec tested:
- Spec name and description
- Result (Pass / Fail / Pass with Concerns)
- Scenarios that passed
- Scenarios that failed (with details)
- Observations and warnings

### Issues to Address

Consolidated list of problems found:
- Severity (Critical / Major / Minor)
- Description
- Where it was found (spec, scenario)
- Evidence (screenshots, console errors, network failures)
- Suggested next steps

### Recommendations

Based on findings:
- Immediate fixes needed
- Areas that need more testing
- Technical debt observations
- Performance concerns

### Test Coverage Notes

What was tested and what wasn't:
- Specs that were run
- Personas covered
- Viewports tested
- Edge cases verified
- Gaps in coverage

## Format Options

### Markdown (default)
Standard Markdown suitable for README, wiki, or documentation systems.

### JSON
Structured data suitable for:
- Importing into issue trackers
- CI/CD pipeline consumption
- Custom reporting tools

```json
{
  "generated_at": "2024-01-15T10:30:00Z",
  "summary": {
    "total_specs": 5,
    "passed": 3,
    "failed": 1,
    "concerns": 1
  },
  "specs": [...],
  "issues": [...],
  "recommendations": [...]
}
```

## Execution

1. Look for recent test results in the conversation context
2. If no recent results, inform user they need to run tests first
3. Compile findings into the requested format
4. Output to console or save to file if requested

## Example Output (Markdown)

```markdown
# QA Test Report

**Generated:** January 15, 2024 10:30 AM
**Project:** ~/Projects/my-app

## Executive Summary

| Metric | Count |
|--------|-------|
| Specs Run | 5 |
| Passed | 3 |
| Failed | 1 |
| Pass with Concerns | 1 |

**Critical Issues:** 1
**Major Issues:** 2
**Minor Issues:** 3

## Critical Issues

### 🔴 Payment form submits without validation
**Spec:** checkout.md > Scenario 3
**Severity:** Critical

The payment form can be submitted with empty credit card fields.
This bypasses client-side validation entirely.

**Evidence:**
- Console shows no validation errors
- Network shows POST to /api/checkout with empty payment object
- Server returns 500 (expects payment data)

**Recommended Action:** Add client-side validation before form submission.

---

## Passed Specs

✅ **login.md** - All 6 scenarios passed
✅ **navigation.md** - All 4 scenarios passed
✅ **search.md** - All 3 scenarios passed

## Concerns

⚠️ **user-profile.md** - Passed with concerns
- Scenario 2 (Profile Update) took 3.2 seconds - potential performance issue
- Console shows deprecation warning for date library

## Recommendations

1. **Immediate:** Fix payment validation (Critical)
2. **This Sprint:** Address slow profile update
3. **Backlog:** Update deprecated date library
```
