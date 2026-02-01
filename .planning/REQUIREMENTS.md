# Requirements: QA Agent v0.2

**Defined:** 2026-01-31
**Core Value:** Tests verify features actually work for users

## v0.2 Requirements

Requirements for CLI testing support. Each maps to roadmap phases.

### Mode Detection

- [ ] **MODE-01**: Agent detects CLI mode from `## Type: cli` header in spec
- [ ] **MODE-02**: Agent defaults to web mode when no type header present (backwards compatible)
- [ ] **MODE-03**: `/qa:run` executes CLI specs without additional flags

### CLI Assertions

- [ ] **ASSERT-01**: Agent can verify exit code matches expected value
- [ ] **ASSERT-02**: Agent can verify stdout contains/equals/matches pattern
- [ ] **ASSERT-03**: Agent can verify stderr contains/equals/is empty
- [ ] **ASSERT-04**: Agent can verify file exists at path
- [ ] **ASSERT-05**: Agent can verify file does not exist at path
- [ ] **ASSERT-06**: Agent can verify file contains text
- [ ] **ASSERT-07**: Agent can verify directory exists
- [ ] **ASSERT-08**: Agent can verify JSON field equals value (using jq)
- [ ] **ASSERT-09**: Agent can verify command succeeds (exit 0)
- [ ] **ASSERT-10**: Agent can verify duration under threshold

### Execution Modes

- [ ] **EXEC-01**: Agent can run commands locally via Bash tool
- [ ] **EXEC-02**: Agent uses project root as default working directory
- [ ] **EXEC-03**: Agent supports per-scenario working directory override
- [ ] **EXEC-04**: Agent can run commands in Docker containers
- [ ] **EXEC-05**: Agent pulls Docker image when not present locally
- [ ] **EXEC-06**: Agent builds Docker image when `build_context` specified
- [ ] **EXEC-07**: Agent mounts volumes as specified in spec
- [ ] **EXEC-08**: Agent cleans up containers after test (--rm)
- [ ] **EXEC-09**: Agent captures stdout and stderr separately

### Path Handling

- [ ] **PATH-01**: Agent expands `~` to host $HOME in local mode
- [ ] **PATH-02**: Agent expands `~` to container $HOME in Docker mode
- [ ] **PATH-03**: Agent expands `./` relative to project root in local mode
- [ ] **PATH-04**: Agent expands `$PROJECT` to project root

### Interactive Commands

- [ ] **INTER-01**: Agent pipes stdin inputs to commands (from `**Input:**` block)
- [ ] **INTER-02**: Agent applies 30-second default timeout
- [ ] **INTER-03**: Agent supports per-scenario timeout override
- [ ] **INTER-04**: Agent reports clear message on timeout

### Environment Variables

- [ ] **ENV-01**: Agent loads environment variables from `.env` file
- [ ] **ENV-02**: Agent passes environment variables to commands
- [ ] **ENV-03**: Agent masks sensitive values in reports
- [ ] **ENV-04**: Agent warns if required variables are missing

### Setup and Teardown

- [ ] **SETUP-01**: Agent runs setup actions before each scenario
- [ ] **SETUP-02**: Agent runs teardown actions after each scenario
- [ ] **SETUP-03**: Agent runs teardown even when scenario fails
- [ ] **SETUP-04**: Agent logs teardown failures as warnings (not failures)
- [ ] **SETUP-05**: Docker mode uses fresh container per scenario (implicit teardown)

### Reporting

- [ ] **REPORT-01**: CLI reports include command executed
- [ ] **REPORT-02**: CLI reports include exit code (actual vs expected)
- [ ] **REPORT-03**: CLI reports include stdout/stderr output
- [ ] **REPORT-04**: CLI reports include file system check results
- [ ] **REPORT-05**: CLI reports include duration
- [ ] **REPORT-06**: CLI reports follow same structure as web reports

### Command Updates

- [ ] **CMD-01**: `/qa:init --type cli` creates CLI spec template
- [ ] **CMD-02**: `/qa:gen --command` generates spec from command
- [ ] **CMD-03**: `/qa:gen --from` generates spec from existing test script
- [ ] **CMD-04**: `/qa:check` works with CLI specs automatically

### Documentation

- [ ] **DOC-01**: README mentions both web and CLI testing
- [ ] **DOC-02**: README has CLI Testing section parallel to web docs
- [ ] **DOC-03**: README documents `Type: cli` and CLI-specific fields
- [ ] **DOC-04**: README includes CLI spec examples

## Future Requirements

Deferred to later milestones.

### Advanced CLI

- **ADV-01**: Parallel scenario execution
- **ADV-02**: Test matrix (multiple env configs)
- **ADV-03**: CI/CD integration helpers

## Out of Scope

| Feature | Reason |
|---------|--------|
| Screen reader testing | Separate accessibility agent project |
| Color contrast | Separate accessibility agent project |
| Complex ARIA | Separate accessibility agent project |
| GUI testing (non-web) | Different tooling required (e.g., Appium) |
| Windows batch/PowerShell | Unix-first, can add later |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| (populated by roadmapper) | | |

**Coverage:**
- v0.2 requirements: 39 total
- Mapped to phases: 0
- Unmapped: 39

---
*Requirements defined: 2026-01-31*
*Last updated: 2026-01-31 after initial definition*
