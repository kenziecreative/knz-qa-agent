---
name: qa:report
description: Generate a summary report from recent QA test runs
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

<objective>
Compile QA findings from the current session into a structured report suitable for sharing with the team or creating issues.
</objective>

<arguments>
- `--format <format>` (optional): Output format - `markdown` (default) or `json`
- `--save <path>` (optional): Save report to file instead of displaying
</arguments>

<process>

<step name="gather-results">
**Gather Results**

Review the current conversation for QA test results:
- `/qa:check` verifications
- `/qa:run` test executions
- Any browser testing observations

If no recent test results found, inform user they need to run tests first.
</step>

<step name="compile-summary">
**Compile Summary**

Aggregate:
- Total specs/checks run
- Pass / Fail / Concern counts
- Critical issues found
- Observations made
</step>

<step name="generate-report">
**Generate Report**

**Markdown format:**

```markdown
# QA Test Report

**Generated:** {date/time}
**Session:** {summary of what was tested}

## Executive Summary

| Metric | Count |
|--------|-------|
| Tests Run | {n} |
| Passed | {n} |
| Failed | {n} |
| Concerns | {n} |

**Critical Issues:** {count}
**Major Issues:** {count}
**Minor Issues:** {count}

## Critical Issues

### 🔴 {Issue Title}
**Location:** {spec/scenario}
**Severity:** Critical

{Description}

**Evidence:**
- {console error / screenshot / network failure}

**Recommended Action:** {what to do}

---

## Major Issues

### 🟠 {Issue Title}
...

## Minor Issues

### 🟡 {Issue Title}
...

## Passed Tests

✅ **{spec/test name}** - {brief note}
...

## Observations

Things that worked but are worth noting:
- {observation}
...

## Recommendations

1. **Immediate:** {critical fixes}
2. **This Sprint:** {major issues}
3. **Backlog:** {minor issues, tech debt}

## Test Coverage

**What was tested:**
- {list of specs/areas}

**What wasn't tested:**
- {gaps in coverage}

**Suggested additional testing:**
- {areas that need more attention}
```

**JSON format:**

```json
{
  "generated_at": "{ISO timestamp}",
  "summary": {
    "total_tests": 0,
    "passed": 0,
    "failed": 0,
    "concerns": 0
  },
  "issues": [
    {
      "severity": "critical|major|minor",
      "title": "",
      "location": "",
      "description": "",
      "evidence": [],
      "recommendation": ""
    }
  ],
  "passed_tests": [
    {
      "name": "",
      "notes": ""
    }
  ],
  "observations": [],
  "recommendations": [],
  "coverage": {
    "tested": [],
    "not_tested": [],
    "suggested": []
  }
}
```
</step>

<step name="output">
**Output Report**

If `--save` provided:
1. Write to specified path
2. Confirm: "Report saved to {path}"

Otherwise:
1. Display report in conversation
</step>

</process>

<severity-guidelines>

## Issue Severity

**Critical (🔴):**
- Feature completely broken
- Data loss possible
- Security vulnerability
- Blocks core user flow

**Major (🟠):**
- Feature partially broken
- Poor user experience
- Performance significantly degraded
- Workaround exists but is not acceptable

**Minor (🟡):**
- Cosmetic issues
- Edge case failures
- Minor UX friction
- Console warnings (not errors)
</severity-guidelines>
