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
- Issue counts by confidence level (high / medium / low)

### Detailed Findings

For each spec tested:
- Spec name and description
- Result (Pass / Fail / Pass with Concerns)
- Scenarios that passed
- Scenarios that failed (with details)
- Observations and warnings

### Definite Issues (High Confidence)

Issues that are certainly real and impactful. These need immediate attention.

For each issue:

- Confidence: High (rationale)
- Severity (Critical / Major / Minor)
- Description
- Where found (spec, scenario)
- Evidence (screenshots, console errors, network failures)
- Suggested next steps

### Likely Issues (Medium Confidence)

Issues that are probably real but need investigation to confirm impact.

For each issue:

- Confidence: Medium (rationale)
- Severity (Critical / Major / Minor)
- Description
- Where found (spec, scenario)
- Evidence (screenshots, console errors, network failures)
- Suggested next steps

### Possible Issues (Low Confidence)

Observations that might indicate problems. Worth noting for future review.

For each issue:

- Confidence: Low (rationale)
- Severity (Critical / Major / Minor)
- Description
- Where found (spec, scenario)
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
  "issues": [
    {
      "confidence": "high",
      "confidence_rationale": "Reproduced consistently; blocks core user flow",
      "severity": "Critical",
      "description": "Payment form submits without validation",
      "spec": "checkout.md > Scenario 3",
      "evidence": "Console shows no validation errors; server returns 500"
    }
  ],
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

**Definite Issues:** 1
**Likely Issues:** 2
**Possible Issues:** 1

## Definite Issues (High Confidence)

### Payment form submits without validation

- **Confidence:** High — reproduced consistently across 3 attempts; directly blocks checkout
- **Spec:** checkout.md > Scenario 3
- **Severity:** Critical

The payment form can be submitted with empty credit card fields.
This bypasses client-side validation entirely.

**Evidence:**
- Console shows no validation errors
- Network shows POST to /api/checkout with empty payment object
- Server returns 500 (expects payment data)

**Suggested next steps:** Add client-side validation before form submission.

---

## Passed Specs

✅ **login.md** - All 6 scenarios passed
✅ **navigation.md** - All 4 scenarios passed
✅ **search.md** - All 3 scenarios passed

## Possible Issues (Low Confidence)

⚠️ **user-profile.md** - Passed with concerns
- Scenario 2 (Profile Update) took 3.2 seconds - potential performance issue
  - Confidence: Low — only observed once, may be environment-related
- Console shows deprecation warning for date library
  - Confidence: Low — warning, not an error; no user-facing impact observed

## Recommendations

1. **Immediate:** Fix payment validation (Definite Issue — Critical)
2. **This Sprint:** Investigate slow profile update
3. **Backlog:** Update deprecated date library
```
