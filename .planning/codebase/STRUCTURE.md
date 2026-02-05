# Codebase Structure

**Analysis Date:** 2026-02-05

## Directory Layout

```
chronicle/
├── chronicle/          # Library packages (release domain, formatting)
├── cmd/                # CLI entrypoint and commands
├── internal/           # Internal-only utilities (git, log, bus)
├── .github/            # CI workflows and issue templates
├── .make/              # Build tooling for release workflows
└── README.md           # Project overview and usage
```

## Directory Purposes

**`chronicle/`:**
- Purpose: Core library and release domain logic
- Contains: `release/` domain, `release/format/` presenters, `release/releasers/` adapters
- Key files: `chronicle/lib.go`, `chronicle/release/changelog_info.go`

**`cmd/`:**
- Purpose: CLI application and command definitions
- Contains: Cobra commands, flag parsing, config binding
- Key files: `cmd/chronicle/main.go`, `cmd/chronicle/cli/cli.go`

**`internal/`:**
- Purpose: Internal-only utilities and infra helpers
- Contains: git access, logging singleton, event bus, helpers
- Key files: `internal/git/interface.go`, `internal/log/log.go`

**`.github/`:**
- Purpose: GitHub workflows and templates
- Contains: CI workflows and issue templates
- Key files: `.github/workflows/release.yaml`, `.github/ISSUE_TEMPLATE/bug_report.md`

**`.make/`:**
- Purpose: Release tooling and supporting Go tooling
- Contains: build scripts and dependency pins
- Key files: `.make/main.go`, `.make/go.mod`

## Key File Locations

**Entry Points:**
- `cmd/chronicle/main.go`: CLI binary entry

**Configuration:**
- `.chronicle.yaml`: project config template
- `.golangci.yaml`: lint configuration

**Core Logic:**
- `chronicle/release/changelog_info.go`: changelog assembly flow
- `chronicle/release/releasers/github/summarizer.go`: GitHub summarizer implementation
- `internal/git/`: git interactions used by summarizer and CLI

**Testing:**
- `chronicle/release/releasers/github/*_test.go`: GitHub summarizer tests
- `internal/git/*_test.go`: git helper tests

## Naming Conventions

**Files:**
- Snake-case for Go files with feature names: `create_config.go`, `changelog_info.go`
- Test files use `_test.go`: `summarizer_test.go`

**Directories:**
- Package-based names: `release/`, `format/`, `releasers/`, `github/`

## Where to Add New Code

**New Feature:**
- Primary code: `chronicle/release/` for domain logic, `cmd/chronicle/cli/commands/` for CLI wiring
- Tests: `chronicle/release/*_test.go` or `cmd/chronicle/cli/commands/*_test.go`

**New Component/Module:**
- Implementation: `chronicle/release/` (domain) or `chronicle/release/releasers/` (provider adapters)

**Utilities:**
- Shared helpers: `internal/` (only for internal packages, not exported)

## Special Directories

**`.planning/codebase/`:**
- Purpose: Generated codebase maps for GSD workflows
- Generated: Yes
- Committed: Not applicable

---

*Structure analysis: 2026-02-05*
