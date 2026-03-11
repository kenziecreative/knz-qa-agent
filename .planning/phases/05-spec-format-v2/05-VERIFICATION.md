---
phase: 05-spec-format-v2
verified: 2026-03-11T00:00:00Z
status: passed
score: 6/6 must-haves verified
gaps: []
human_verification:
  - test: "Run /qa:gen login --a11y-depth deep and verify Accessibility Focus section is present in output"
    expected: "Generated spec includes ## Accessibility Focus section with all four areas listed"
    why_human: "qa-gen is a skill invoked by Claude, not statically verifiable code — cannot execute skills programmatically"
  - test: "Run /qa:run --project <path> --tag smoke and verify only smoke-tagged scenarios reach the agent"
    expected: "Agent receives filtered scenario list; non-smoke scenarios are not in the prompt"
    why_human: "Tag filtering is runtime behavior in the qa-run skill prompt generation — cannot unit-test markdown prompts"
  - test: "Run /qa:run with a spec containing depends_on and fail the dependency scenario"
    expected: "Dependent scenario is marked Skipped with note referencing the failed dependency"
    why_human: "Dependency skip behavior is runtime agent behavior — requires actual test execution to observe"
---

# Phase 5: Spec Format v2 Verification Report

**Phase Goal:** Spec format supports dependencies, data-driven testing, selective execution, and environment profiles
**Verified:** 2026-03-11
**Status:** passed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | Specs can express scenario dependencies (`depends_on`) | VERIFIED | `depends_on:` field documented in SPEC-FORMAT.md (7 occurrences), demonstrated in sample-login-spec.md (scenarios 5 and 6 reference `1. Successful Login`), agent handles in "Spec Format v2 Features" section |
| 2 | Specs support data-driven scenarios (multiple data sets, one scenario definition) | VERIFIED | `## Data Sets` section documented in SPEC-FORMAT.md with `{variable}` substitution syntax, implemented in sample-login-spec.md (3 variants for Failed Login), agent instructions specify per-variant execution and reporting |
| 3 | Specs support tags and `/qa:run --tag` filters by them | VERIFIED | `tags:` field documented in SPEC-FORMAT.md; `--tag` flag in qa-run.md frontmatter arguments, Modes 5/6/7, and parsing section (OR logic, comma-separated); all 6 sample spec scenarios have tags |
| 4 | Specs support environment profiles and `/qa:run --env` selects the active profile | VERIFIED | `## Environments` section in SPEC-FORMAT.md with full syntax; `--env` flag in qa-run.md with backward compat fallback chain; sample spec has local/staging profiles; agent uses Active Environment block |
| 5 | Specs support `## Accessibility Focus` section to trigger Tier 2 depth | VERIFIED | Section documented in SPEC-FORMAT.md with four named areas; agent "Accessibility Focus Trigger" subsection in qa-tester.md distinguishes Tier 1 (absent) vs Tier 2 (present); sample spec includes `- form-accessibility` |
| 6 | `/qa:gen --a11y-depth deep` generates specs with accessibility sections | VERIFIED | `--a11y-depth` flag in qa-gen.md frontmatter, parsing section, and all three generation modes (description, documentation, live URL) include conditional step to add `## Accessibility Focus` section when `deep` |

**Score:** 6/6 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `examples/SPEC-FORMAT.md` | v2 spec format documentation | VERIFIED | 402 lines; contains depends_on (7x), Data Sets (4x), tags: (13x), Environments (7x), Accessibility Focus (5x); backward compatibility section present |
| `examples/sample-login-spec.md` | Sample spec with all v2 features | VERIFIED | 208 lines; has Environments section (local+staging), Accessibility Focus (form-accessibility), tags on all 6 scenarios, depends_on on scenarios 5 and 6, Data Sets with 3 variants on scenario 2 |
| `agents/qa-tester.md` | Agent with v2 interpretation instructions | VERIFIED | 734 lines; "Spec Format v2 Features" section added after Session Structure with all five subsections: Dependencies, Data-Driven, Tags, Environment Profiles, Accessibility Focus Trigger |
| `skills/qa-run.md` | Skill with --tag and --env flags | VERIFIED | 246 lines; frontmatter includes `--tag` and `--env`; Modes 5/6/7 added; parsing section documents both flags; execution flow has environment resolution, tag filtering, and dependency auto-inclusion steps; Agent Invocation template includes Active Environment, Tag Filter, Execution Notes blocks |
| `skills/qa-gen.md` | Skill with --a11y-depth and v2 generation | VERIFIED | 204 lines; `--a11y-depth` flag in frontmatter and parsing section; v2 generation steps in all three modes; Output template includes Environments section and conditional Accessibility Focus section; Tag Assignment Guide added; new --a11y-depth deep examples |
| `README.md` | User documentation for all v2 features | VERIFIED | 392 lines; --tag/--env examples in /qa:run section; --a11y-depth in /qa:gen section; v2 Features subsection with tags, dependencies, data-driven, environments, and accessibility focus; Accessibility Tier 2 updated from "Planned" to "activated via spec" |
| `skills/qa-init.md` | Init templates with v2 format sections | VERIFIED | 138 lines; Writing Specs list includes tags, dependencies, environments, and accessibility focus entries; smoke-test template has `tags: [smoke]` on all 3 scenarios and basic Environments section; After Initialization references v2 docs |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `skills/qa-run.md` | `agents/qa-tester.md` | Agent Invocation prompt template | VERIFIED | qa-run.md includes structured prompt blocks: Active Environment (profile name/base_url/credentials/timeouts), Tag Filter (active tags/filtered scenario names/auto-included dependencies), Execution Notes (dependency skip and data-driven variant instructions) |
| `skills/qa-gen.md` | `examples/SPEC-FORMAT.md` | Generated spec follows v2 format | VERIFIED | qa-gen Output template includes Environments section (optional) and Accessibility Focus section (conditional on --a11y-depth deep); tag assignment matches standard tags defined in SPEC-FORMAT.md |
| `examples/SPEC-FORMAT.md` | `agents/qa-tester.md` | Format sections defined in SPEC-FORMAT understood by agent | VERIFIED | Agent has subsections for each v2 feature; four accessibility focus area names are consistent between SPEC-FORMAT.md (`focus-management`, `page-structure`, `interactive-elements`, `form-accessibility`) and agent valid area names |
| `agents/qa-tester.md` | Tier 2 accessibility methodology | Accessibility Focus trigger controls depth | VERIFIED | Agent "Accessibility Focus Trigger" subsection explicitly maps each area name to specific Structured Accessibility subsections (e.g., `form-accessibility` → Error Summary, Error Summary Links, aria-invalid, Focus on First Error) |

### Requirements Coverage

SPEC-01 through SPEC-08 requirements are addressed by the six success criteria in the ROADMAP. Based on the PLAN files, all targeted requirements map to verified artifacts:

| Requirement Group | Status | Evidence |
|------------------|--------|----------|
| SPEC-01 (depends_on) | SATISFIED | SPEC-FORMAT.md, sample spec, qa-tester agent |
| SPEC-02 (data-driven) | SATISFIED | SPEC-FORMAT.md, sample spec, qa-tester agent |
| SPEC-03 (tags + --tag filter) | SATISFIED | SPEC-FORMAT.md, qa-run.md, qa-tester agent |
| SPEC-04 (environments + --env) | SATISFIED | SPEC-FORMAT.md, qa-run.md, qa-tester agent |
| SPEC-05 (Accessibility Focus) | SATISFIED | SPEC-FORMAT.md, sample spec, qa-tester agent |
| SPEC-06 (--a11y-depth deep) | SATISFIED | qa-gen.md |

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `README.md` | 259 | Accessibility focus area names differ from canonical definitions | Warning | README lists `landmarks-and-headings`, `keyboard-navigation`, `touch-and-zoom`; canonical names in SPEC-FORMAT.md and agent are `page-structure`, `interactive-elements`, `focus-management`, `form-accessibility`. A user reading only README would try incorrect area names. |

No blocker anti-patterns found. The README area name inconsistency is documentation drift — the authoritative sources (SPEC-FORMAT.md and the agent) are correct and consistent with each other.

### Human Verification Required

#### 1. Tag Filtering Runtime Behavior

**Test:** Create a spec with smoke and regression scenarios. Run `/qa:run --tag smoke`. Inspect what the agent receives.
**Expected:** Agent prompt contains only smoke-tagged scenarios in the "Scenarios to run" list; regression scenarios are absent.
**Why human:** Tag filtering is implemented in the qa-run skill's natural language instructions for Claude — it cannot be unit tested without invoking the skill.

#### 2. Dependency Skip Behavior

**Test:** Run a spec where scenario A has `depends_on: [scenario that will fail]`. Trigger the dependency failure.
**Expected:** Dependent scenario shows "Skipped: dependency '[name]' failed — [reason]" rather than a test failure.
**Why human:** Skip behavior is runtime agent behavior — observable only by running actual tests.

#### 3. Data-Driven Variant Execution

**Test:** Run the sample-login-spec.md against the login page (or any spec with `## Data Sets`).
**Expected:** Each data set variant runs separately; results show "Scenario [variant-name]: PASS/FAIL" per variant.
**Why human:** Per-variant execution requires actual agent invocation with a live application.

#### 4. Environment Profile Override

**Test:** Run `/qa:run --project <path> --spec login --env staging` against a spec with an Environments section.
**Expected:** Agent report header shows "Environment: staging"; navigation uses the staging base_url, not the spec's Base URL.
**Why human:** Profile selection and override behavior requires end-to-end skill execution.

#### 5. `/qa:gen --a11y-depth deep` Output

**Test:** Run `/qa:gen checkout --a11y-depth deep "Checkout flow"`.
**Expected:** Generated spec contains `## Accessibility Focus` section with all four areas; happy path scenario has `tags: [smoke, critical]`.
**Why human:** Generation output requires invoking the skill with Claude.

### Gaps Summary

No gaps blocking goal achievement. All six success criteria are verified through structural examination of the codebase.

One documentation inconsistency was found: the README's `## Accessibility Focus` section lists non-canonical area names (`landmarks-and-headings`, `keyboard-navigation`, `touch-and-zoom`) that do not match the canonical names defined in SPEC-FORMAT.md and used by the agent (`page-structure`, `interactive-elements`, `focus-management`, `form-accessibility`). This is a warning but not a blocker — the agent and SPEC-FORMAT.md are authoritative and correct; the README is wrong only in this specific enumeration. A user reading the README alone could be confused when trying to specify focus areas, but the linked SPEC-FORMAT.md has the correct names.

---

_Verified: 2026-03-11_
_Verifier: Claude (gsd-verifier)_
