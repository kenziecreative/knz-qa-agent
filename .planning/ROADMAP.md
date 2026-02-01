# Roadmap: QA Agent v0.2 (CLI Testing Support)

## Overview

This milestone extends the QA agent to support CLI/terminal application testing alongside existing web testing. The journey starts with mode detection, builds local execution with assertions, adds environment handling and interactive commands, enables Docker isolation, and finishes with polished commands and documentation.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Mode Detection** - Agent detects CLI vs web mode from spec headers
- [ ] **Phase 2: Local Execution + Assertions** - Agent runs CLI commands and verifies results
- [ ] **Phase 3: Environment + Interactive** - Agent handles env vars, setup/teardown, stdin, timeouts
- [ ] **Phase 4: Docker Execution** - Agent runs commands in isolated containers
- [ ] **Phase 5: Polish** - Updated commands, reporting, and documentation

## Phase Details

### Phase 1: Mode Detection
**Goal**: Agent knows whether to execute CLI or web mode based on spec content
**Depends on**: Nothing (first phase)
**Requirements**: MODE-01, MODE-02, MODE-03
**Success Criteria** (what must be TRUE):
  1. Spec with `## Type: cli` header triggers CLI execution mode
  2. Spec without type header (or with `Type: web`) triggers web execution mode
  3. `/qa:run` executes CLI specs without needing additional flags

Plans:
- [ ] 01-01: TBD

### Phase 2: Local Execution + Assertions
**Goal**: Agent can run CLI commands locally and verify their results
**Depends on**: Phase 1 (mode detection enables CLI path)
**Requirements**: EXEC-01, EXEC-02, EXEC-03, EXEC-09, PATH-03, PATH-04, ASSERT-01, ASSERT-02, ASSERT-03, ASSERT-04, ASSERT-05, ASSERT-06, ASSERT-07, ASSERT-08, ASSERT-09, ASSERT-10
**Success Criteria** (what must be TRUE):
  1. Agent executes commands via Bash tool and captures exit code
  2. Agent captures stdout and stderr as separate streams
  3. Commands run from project root by default, or per-scenario working directory
  4. `./` paths resolve relative to project root, `$PROJECT` expands correctly
  5. Agent verifies exit code, stdout/stderr content, file system state, JSON fields, and duration

Plans:
- [ ] 02-01: TBD

### Phase 3: Environment + Interactive
**Goal**: Agent handles environment variables, setup/teardown, stdin, and timeouts
**Depends on**: Phase 2 (execution works, now make it realistic)
**Requirements**: ENV-01, ENV-02, ENV-03, ENV-04, SETUP-01, SETUP-02, SETUP-03, SETUP-04, SETUP-05, INTER-01, INTER-02, INTER-03, INTER-04, PATH-01, PATH-02
**Success Criteria** (what must be TRUE):
  1. Agent loads environment variables from `.env` file and passes to commands
  2. Agent masks sensitive values in output (never logs API keys)
  3. Agent runs setup before / teardown after each scenario (even on failure)
  4. Agent pipes stdin inputs from `**Input:**` block to commands
  5. Commands have 30-second default timeout, overridable per-scenario
  6. `~` expands to host $HOME in local mode

Plans:
- [ ] 03-01: TBD

### Phase 4: Docker Execution
**Goal**: Agent can run commands in isolated Docker containers
**Depends on**: Phase 3 (local execution patterns established, now isolate)
**Requirements**: EXEC-04, EXEC-05, EXEC-06, EXEC-07, EXEC-08
**Success Criteria** (what must be TRUE):
  1. Agent runs commands inside Docker containers
  2. Agent pulls Docker image when not present locally
  3. Agent builds Docker image when `build_context` specified in spec
  4. Agent mounts volumes as specified in spec
  5. Agent cleans up containers after test (--rm behavior)
  6. `~` expands to container $HOME in Docker mode

Plans:
- [ ] 04-01: TBD

### Phase 5: Polish
**Goal**: Commands updated for CLI support, reporting complete, documentation finished
**Depends on**: Phase 4 (everything works, now polish the interface)
**Requirements**: REPORT-01, REPORT-02, REPORT-03, REPORT-04, REPORT-05, REPORT-06, CMD-01, CMD-02, CMD-03, CMD-04, DOC-01, DOC-02, DOC-03, DOC-04
**Success Criteria** (what must be TRUE):
  1. CLI reports include command, exit code, stdout/stderr, file system checks, duration
  2. CLI reports follow same structure as web reports
  3. `/qa:init --type cli` creates CLI spec template
  4. `/qa:gen --command` and `/qa:gen --from` generate CLI specs
  5. README documents both web and CLI testing as first-class citizens

Plans:
- [ ] 05-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Mode Detection | 0/TBD | Not started | - |
| 2. Local Execution + Assertions | 0/TBD | Not started | - |
| 3. Environment + Interactive | 0/TBD | Not started | - |
| 4. Docker Execution | 0/TBD | Not started | - |
| 5. Polish | 0/TBD | Not started | - |
