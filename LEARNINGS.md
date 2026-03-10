# Learnings & Approach

This document captures the decisions, patterns, and lessons learned while building the QA Agent. It's intended to guide future related projects to follow the same proven patterns.

## Project Context

**Goal:** Create a QA agent that tests web applications like a human would - with intuition, pattern recognition, and the ability to escalate when something isn't working.

**Key Insight:** The value of an AI-powered QA agent isn't just automation - it's *judgment*. Traditional E2E tests follow scripts. This agent thinks about whether the results make sense.

---

## Architecture Decisions

### Plugin Architecture

**Decision:** Implemented as a Claude Code plugin with skills, agents, and hooks.

**Why:**

- Self-contained and portable — clone the repo, it works
- Supports agents with dedicated system prompts and tool restrictions
- Supports hooks for automation (SessionStart, Stop events)
- Bundles MCP server configuration via `.mcp.json`
- Skills are auto-invocable by Claude when relevant
- Version-controllable as a single unit

**Pattern:**

```text
plugin-root/
├── .claude-plugin/
│   ├── plugin.json
│   └── .mcp.json
├── agents/qa-tester.md
├── skills/qa-run.md, qa-check.md, ...
├── hooks/hooks.json
└── examples/
```

**Evolution note:** Originally implemented as global commands in `~/.claude/commands/qa/`. Commands have been fully removed. Skills are the only invocation pattern — auto-invocable, modern Claude Code standard, and more encapsulated than standalone command files.

### Skill Structure

**Decision:** Each skill is a self-contained Markdown file with YAML frontmatter.

**Why this structure:**

- Frontmatter defines metadata Claude Code needs
- `allowed-tools` restricts what tools the skill can use (security/focus)
- XML-like tags (`<objective>`, `<process>`) help Claude parse sections
- Steps are named for clarity in long skills
- Methodology section is where you teach *how to think*, not just what to do
- Examples ground the abstract instructions in concrete output

### Tool Access

**Decision:** Explicitly list allowed tools in frontmatter.

**Key tools for web QA:**

- `mcp__playwright__*` - Playwright browser automation
- `Read`, `Write` - File operations (specs, reports)
- `Bash` - Creating directories, running commands
- `Glob`, `Grep` - Finding files

**Why explicit:**

- Focuses the agent on relevant tools
- Prevents accidental use of tools that might cause issues
- Documents what capabilities the skill needs

---

## Testing Methodology

### "Test Like a Human" Philosophy

**Core insight:** The difference between useful QA and frustrating QA is whether it catches real problems or just generates noise.

**What humans do that scripts don't:**

1. Notice when something "feels off" even if it technically works
2. Try obvious edge cases without being told
3. Dig into *why* something failed, not just *that* it failed
4. Recognize patterns across multiple failures
5. Question whether the spec is right when behavior seems intentional

**Baked into the agent:**

- Take screenshots at key points (evidence)
- Check console after every interaction (not just for errors - warnings matter)
- Note timing (slow is a bug even if it "works")
- Try keyboard path, not just mouse clicks
- Report confidence levels, not just pass/fail

### Pattern Recognition & Escalation

**Key insight:** An agent that retries the same failing test forever is worse than useless.

**The 3-failure rule:**

1. First failure: Report issue, expect fix
2. Second failure: Report issue, note the pattern
3. Third failure: **Escalate** - "This has failed repeatedly. The implementation approach may need reconsideration."

**Why this matters:**

- Prevents infinite loops of "fix -> test -> fail -> fix -> test -> fail"
- Forces humans to step back and think about root causes
- Distinguishes "bug" from "fundamental misunderstanding"

### Spec Skepticism

**Decision:** The agent should question specs, not blindly follow them.

**Scenarios:**

- Spec says X, app does Y, Y seems reasonable -> Ask which is correct
- Spec says X, app does Y, Y seems wrong -> Report as bug with reasoning
- Spec is ambiguous -> Ask for clarification

**Why:**

- Specs can be outdated or wrong
- Observed behavior might be intentional improvement
- Agent's job is to find truth, not enforce documents

---

## Accessibility Integration

### Three-Tier Model

**Decision:** Split accessibility into three tiers:

**Tier 1 - Baseline (baked into every test):**

- Keyboard navigation and focus indicators
- Form labels and error association
- Semantic HTML (buttons, links, headings)
- Basic ARIA (expanded states, live regions)
- Alt text on images

**Tier 2 - Structured (dedicated phase in QA agent, Phase 4):**

- Focus management in modals (move in, trap, return on close)
- Skip links
- Heading hierarchy audits
- Landmark regions
- Touch target sizing (44x44px minimum)
- Zoom to 200% behavior
- `prefers-reduced-motion` respect
- Form error patterns (summary, field links, aria-invalid)
- Color as sole indicator
- Page title and lang attribute

**Tier 3 - Specialized (separate accessibility agent, future):**

- Screen reader output verification
- Color contrast ratio calculations
- Full WCAG 2.2 AA compliance audit
- Complex ARIA widget patterns
- Cognitive accessibility

**Why three tiers:**

- Tier 1 is what any developer should check — it's just good testing practice
- Tier 2 can be done with Playwright alone — no external tools needed
- Tier 3 requires specialized tooling (axe-core, CDP, assistive technology)
- Each tier has clear boundaries preventing scope creep

### Integration, Not Separation

**Decision:** Accessibility checks are part of functional testing, not a separate pass.

**Why:**

- Same browser session, same context
- "Does the button work?" includes "Can I tab to it?"
- One report, one set of feedback to the developer
- Reduces overhead and context switching

---

## Report Design

### Persistent Reports

**Decision:** Write reports to `.qa/reports/` in the project, not just conversation output.

**Structure:**

```text
.qa/reports/
├── HISTORY.md                    # Running log of all QA activity
├── check-phase-3-2024-01-29.md   # Individual check reports
└── run-login-2024-01-29.md       # Individual run reports
```

**Why:**

- Creates audit trail
- Can review history ("when did this break?")
- Reports survive conversation context
- Can be committed to git for team visibility

### HISTORY.md Pattern

**Decision:** Append one-line summaries to a running history file.

**Format:**

```markdown
| Date | Test | Result | Summary |
| ---- | ---- | ------ | ------- |
| 2024-01-29 | Phase 3 | PASS | Login flow verified |
| 2024-01-29 | Phase 4 | FAIL | Cart API returns 500 |
```

**Why:**

- Quick glance at project health over time
- Identifies patterns ("Phase 4 has failed 3 times this week")
- Low overhead (one line per test)

### Report Content

**Decision:** Reports should be actionable, not just informational.

**Required elements:**

- What was tested (phase goal, spec name)
- What happened (pass/fail/concern with evidence)
- Why it matters (severity, impact)
- What to do next (fix, proceed, escalate)

**Evidence includes:**

- Console output
- Network request/response details
- Screenshots (referenced)
- Specific element selectors

---

## GSD Integration

### Project-Level Configuration

**Decision:** QA behavior is configured per-project in PROJECT.md, not globally.

**Why:**

- Different projects have different needs
- Base URL varies per project
- Some projects don't need browser QA at all

**Pattern:**

```markdown
## Verification

This is a web application. After completing each phase:

1. Run `/qa:check --url http://localhost:3000` to verify functionality
```

GSD reads project context, sees this instruction, follows it.

### Phase Awareness

**Decision:** `/qa:check` reads GSD phase context to understand what to test.

**Sources (in order):**

1. `.planning/ROADMAP.md` - Current phase and goal
2. `.planning/phases/{N}/PLAN.md` - What was planned
3. `.planning/phases/{N}/GOAL.md` - Phase objective

**Why:**

- Agent knows *what was supposed to be built*
- Can verify deliverables, not just "does something work"
- Doesn't require human to re-explain the phase

---

## Spec Format

### Natural Language Specs

**Decision:** Specs are written in natural language Markdown, not a DSL.

**Why:**

- Humans can read and write them without learning syntax
- Agent can interpret intent, not just follow steps
- Easy to review in PRs
- Can evolve with the feature

### Spec Location

**Decision:** Specs live in `.qa/` directory of each project.

**Why:**

- Version controlled with the code
- Lives near what it tests
- Easy to find and update
- Can be reviewed with feature PRs

---

## Lessons for Future Agents

### What Worked

1. **Plugin architecture** - Self-contained, portable, supports agents/hooks/MCP
2. **Methodology sections** - Teaching *how to think* not just *what to do*
3. **Escalation rules** - Prevents infinite loops, forces human judgment
4. **Persistent reports** - Creates accountability and audit trail
5. **Project-level config** - Flexible, doesn't impose on projects that don't need it
6. **Natural language specs** - Leverages agent's strength (interpretation)
7. **Skills over commands** - Auto-invocable, modern Claude Code pattern

### What to Avoid

1. **Mixing domains** - CLI testing muddied the web focus; keep agents domain-specific
2. **Overly rigid output formats** - Let the agent adapt to what it finds
3. **Assuming specs are correct** - Always allow for spec skepticism
4. **Separate accessibility pass** - Integrate into functional testing
5. **Stale tool references** - Keep tool lists current as Claude Code evolves

### Patterns to Reuse

1. **Skill file structure** - Frontmatter + objective + process + methodology + examples
2. **3-failure escalation** - Applies to any repetitive task
3. **HISTORY.md log** - Quick audit trail for any agent that produces reports
4. **"Test like a human" framing** - Useful for any agent that evaluates quality
5. **Three-tier capability model** - Baseline/structured/specialized prevents scope creep

---

## For the Accessibility Agent

When building the Advanced Accessibility Agent (Tier 3), apply these patterns:

1. **Use plugin architecture** - Same structure as QA agent
2. **Same report location** - `.a11y/reports/` with HISTORY.md
3. **Integrate with QA** - Can be called from `/qa:check` or standalone
4. **Escalation for repeated issues** - If same contrast issue appears 3x, note pattern
5. **WCAG as "spec"** - But allow skepticism ("this technically fails but is fine in context")
6. **Actionable reports** - Not just "this failed" but "here's the fix"

The accessibility agent has a more technical job (axe-core, CDP, contrast calculations) but the *approach* should be the same: test like a human expert would, provide judgment not just data, and escalate when patterns suggest deeper issues.
