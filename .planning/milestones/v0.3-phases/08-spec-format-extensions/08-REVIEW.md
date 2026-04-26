---
phase: 08-spec-format-extensions
reviewed: 2026-04-19T21:15:00Z
depth: standard
files_reviewed: 4
files_reviewed_list:
  - examples/SPEC-FORMAT.md
  - agents/qa-tester.md
  - skills/run/SKILL.md
  - examples/visual-testing-spec.md
findings:
  critical: 0
  warning: 3
  info: 3
  total: 6
status: issues_found
---

# Phase 8: Code Review Report

**Reviewed:** 2026-04-19T21:15:00Z
**Depth:** standard
**Files Reviewed:** 4
**Status:** issues_found

## Summary

Reviewed the spec format documentation, the qa-tester agent, the qa:run skill, and the visual testing example spec. These are agent prompt/instruction files (Markdown), not executable code, so the review focuses on correctness of cross-file contracts, consistency of documented interfaces, and potential for runtime misbehavior when the agent or skill follows these instructions.

Three warnings found: a missing argument definition that would cause parsing failures, a reference to a nonexistent CLI command, and inconsistent default viewport dimensions across files that could produce conflicting test behavior. Three info-level items around documentation consistency.

## Warnings

### WR-01: `--browsers` flag used but not defined in argument parsing

**File:** `skills/run/SKILL.md:58-71`
**Issue:** Mode 8 (line 58) shows usage of `--browsers` flag (`/qa:run --project ~/Projects/my-app --spec checkout --browsers`), but the "Parsing Arguments" section (lines 64-71) does not list `--browsers` as a parseable argument. The skill enumerates exactly 6 arguments (project, spec, url, task, tag, env) and `--browsers` is absent. When the skill attempts to parse user input containing `--browsers`, it has no instruction for how to handle it -- it may be silently ignored or misinterpreted as part of the task string.

**Fix:** Add `--browsers` to the Parsing Arguments section:

```markdown
7. **browsers**: Boolean flag to enable multi-browser testing (optional). When present, reads the browser engine list from the spec's `## Browsers` section and runs one invocation per engine. When absent, defaults to chromium only (even if spec has a `## Browsers` section).
```

Alternatively, if the intent is that the `## Browsers` section in a spec is always honored automatically (no flag needed), remove Mode 8 and the `--browsers` flag from the usage examples to avoid confusion.

### WR-02: Agent invocation template references nonexistent `playwright-cli run-code` command

**File:** `skills/run/SKILL.md:212`
**Issue:** The agent invocation template's accessibility instructions reference `playwright-cli run-code` as an alternative to `playwright-cli eval` for setting reduced-motion preferences. However, `run-code` is not a documented command in the agent's command reference (`agents/qa-tester.md`, lines 28-78). The agent's available commands only include `playwright-cli eval <expression> [element]`. If the agent follows this instruction and attempts `playwright-cli run-code`, the command will fail.

**Fix:** Remove the nonexistent command reference. Change line 212 from:

```markdown
  - Reduced motion: Use `playwright-cli eval` or `playwright-cli run-code` to set prefers-reduced-motion: reduce, verify animations stop
```

to:

```markdown
  - Reduced motion: Use `playwright-cli eval` to set prefers-reduced-motion: reduce, verify animations stop
```

### WR-03: Inconsistent default viewport dimensions between agent and spec format documentation

**File:** `agents/qa-tester.md:434` and `examples/SPEC-FORMAT.md:140-142`
**Issue:** The agent defines default viewports as mobile `375x812`, tablet `768x1024`, desktop `1280x800` (line 434 of qa-tester.md). The spec format documentation defines the common presets as desktop `1280x720`, tablet `768x1024`, mobile `375x667` (lines 140-142 of SPEC-FORMAT.md). The visual-testing-spec.md example uses `1280x720` and `375x667`, matching SPEC-FORMAT.md but not the agent.

When a spec does not explicitly declare viewports, the agent uses its own defaults (375x812, 1280x800). When a spec author writes viewports following the SPEC-FORMAT.md documentation, they use 375x667 and 1280x720. This means the mobile height differs by 145px and the desktop height differs by 80px depending on whether viewports are explicitly stated. Tests that check for vertical layout, content visibility below the fold, or fixed-position elements could produce different results depending on which set of dimensions applies.

**Fix:** Align the viewport dimensions. Either update the agent defaults (line 434 of qa-tester.md) to match the spec format documentation:

```markdown
- **Default viewports:** mobile (375x667), tablet (768x1024), desktop (1280x720)
```

Or update SPEC-FORMAT.md presets (lines 140-142) to match the agent. The agent dimensions (375x812 for iPhone X-style, 1280x800 for modern laptop) are arguably more current, but consistency is the priority.

## Info

### IN-01: Hardcoded example credentials in documentation

**File:** `examples/SPEC-FORMAT.md:19,118` and `examples/visual-testing-spec.md:14`
**Issue:** Example specs include plaintext credentials like `test@example.com / testpass123`, `admin@example.com / adminpass123`, `test@example.com / password123`. While these are documentation examples (not production code) and the SPEC-FORMAT.md explicitly advises using environment variables for real secrets (line 164), the examples model a pattern of embedding credentials directly in spec files. The `## Environments` section properly demonstrates `$STAGING_PASSWORD` syntax for secrets, but the simpler examples above it do not.

**Fix:** Consider adding a brief note near the basic Personas example (line 18-19) reminding spec authors to use env vars for real credentials, consistent with the guidance already present in the Environments section.

### IN-02: Mixed heading numbering conventions in `depends_on` examples across files

**File:** `examples/SPEC-FORMAT.md:80,392` and `examples/visual-testing-spec.md:65`
**Issue:** The spec format documentation shows two different `depends_on` referencing styles. In the basic structure example (line 80): `depends_on: Scenario 1` (heading was `### Scenario 1: [Name]`). In the Scenario Dependencies section (line 392): `depends_on: Successful Login` (heading was `### Successful Login`). The visual-testing-spec uses `depends_on: 1. Homepage Loads Correctly` (heading was `### 1. Homepage Loads Correctly`). All are internally consistent within their own examples, but a spec author might be confused about whether to include numbering in the dependency reference. The instruction at line 376 ("Dependencies reference scenarios by their exact name (the heading text after `###`)") is clear, but having varied styles across examples could cause confusion.

**Fix:** No code change needed. The documentation rule is clear. Optionally, standardize examples to use one heading style (either numbered or plain) consistently.

### IN-03: Visual Focus areas reference unimplemented phases

**File:** `agents/qa-tester.md:747` and `examples/SPEC-FORMAT.md:238-241`
**Issue:** The Visual Focus section references specific phases for unimplemented methodology: `design-verification` (Phase 9), `ux-states` (Phase 10), `layout-integrity` (Phase 11), `performance-responsive` (Phase 12). The agent handles this gracefully with the fallback note at line 747 ("Visual Focus requested for [areas] -- full methodology pending Phase [N]"). The visual-testing-spec.md example requests `design-verification` and `layout-integrity`, which would trigger this placeholder behavior. This is not a bug -- the forward-reference design is intentional -- but anyone running the example spec today would get the placeholder note rather than actual visual testing.

**Fix:** No code change needed. This is working as designed. The example spec could optionally include a comment noting that this demonstrates a future feature.

---

_Reviewed: 2026-04-19T21:15:00Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
