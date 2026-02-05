# Architecture

**Analysis Date:** 2026-02-05

## Pattern Overview

**Overall:** CLI-driven layered library with provider adapters

**Key Characteristics:**
- CLI entrypoint delegates to command handlers in `cmd/chronicle/cli/commands/`
- Core domain types and workflows live in `chronicle/release/`
- External provider logic is implemented as adapters under `chronicle/release/releasers/`

## Layers

**CLI Layer:**
- Purpose: Parse flags and dispatch commands
- Location: `cmd/chronicle/cli/` and `cmd/chronicle/cli/commands/`
- Contains: Cobra commands, config binding, presenter selection
- Depends on: `chronicle/release/`, `chronicle/release/releasers/github/`, `internal/git/`, `internal/log/`
- Used by: `cmd/chronicle/main.go`

**Domain Layer:**
- Purpose: Represent release data and compute changelog info
- Location: `chronicle/release/`
- Contains: `Release`, `Description`, change types, summarizer interfaces, version speculation
- Depends on: `chronicle/release/change/`, `internal/log/`, `internal/time_helper.go`
- Used by: `cmd/chronicle/cli/commands/`, `chronicle/release/format/`

**Provider Adapter Layer:**
- Purpose: Fetch and summarize release data from GitHub
- Location: `chronicle/release/releasers/github/`
- Contains: `Summarizer`, GitHub API fetchers, changelog end tag resolver, version speculation
- Depends on: `internal/git/`, `chronicle/release/`, `chronicle/release/change/`
- Used by: `cmd/chronicle/cli/commands/create_github_worker.go`

**Infrastructure Layer:**
- Purpose: Git access and cross-cutting utilities
- Location: `internal/git/`, `internal/log/`, `internal/bus/`, `internal/regex_helpers.go`
- Contains: git interface/impl, logging singleton, bus publisher, regex/time helpers
- Depends on: `github.com/go-git/go-git/v5` and standard library
- Used by: `chronicle/release/releasers/github/`, `cmd/chronicle/cli/commands/`

## Data Flow

**Create Changelog Flow:**

1. CLI starts in `cmd/chronicle/main.go` and builds the application in `cmd/chronicle/cli/cli.go`.
2. Command `create` in `cmd/chronicle/cli/commands/create.go` validates repo path via `internal/git/is_repository.go`.
3. Worker selection in `cmd/chronicle/cli/commands/create_github_worker.go` constructs a GitHub summarizer from `chronicle/release/releasers/github/summarizer.go`.
4. `chronicle/release/ChangelogInfo` in `chronicle/release/changelog_info.go` fetches the starting release, collects changes, and builds `chronicle/release/Description`.
5. Output formatting uses a presenter from `chronicle/release/format/markdown/presenter.go` or `chronicle/release/format/json/presenter.go`.
6. Presenter writes to stdout via `presenter.Present(os.Stdout)` in `cmd/chronicle/cli/commands/create.go`.

**Next Version Flow:**

1. CLI command `next-version` in `cmd/chronicle/cli/commands/next_version.go` builds a minimal `createConfig`.
2. The same worker path in `cmd/chronicle/cli/commands/create_github_worker.go` produces a `release.Description` with a speculative version via `chronicle/release/version_speculator.go`.
3. The computed version is written to stdout in `cmd/chronicle/cli/commands/next_version.go`.

**State Management:**
- Stateless execution per invocation; transient state held in `createConfig` in `cmd/chronicle/cli/commands/create_config.go`.
- Cross-cutting logger and event bus are global singletons in `chronicle/lib.go` and `internal/log/log.go`.

## Key Abstractions

**Summarizer:**
- Purpose: Abstract release/changes retrieval
- Examples: `chronicle/release/summarizer.go`, `chronicle/release/releasers/github/summarizer.go`
- Pattern: Interface + provider implementation

**VersionSpeculator:**
- Purpose: Compute next version from changes
- Examples: `chronicle/release/version_speculator.go`, `chronicle/release/releasers/github/version_speculator.go`
- Pattern: Interface + provider implementation

**Presenter:**
- Purpose: Format release descriptions
- Examples: `chronicle/release/format/markdown/presenter.go`, `chronicle/release/format/json/presenter.go`
- Pattern: Factory selection in `cmd/chronicle/cli/commands/create_presenter.go`

## Entry Points

**CLI Binary:**
- Location: `cmd/chronicle/main.go`
- Triggers: `chronicle` command
- Responsibilities: Build `clio` app and run command tree

**Library Initialization:**
- Location: `chronicle/lib.go`
- Triggers: `clio` initializer in `cmd/chronicle/cli/cli.go`
- Responsibilities: Inject logger and event bus into internal packages

## Error Handling

**Strategy:** Propagate errors upward and fail fast

**Patterns:**
- Wrap errors with context in `chronicle/release/changelog_info.go`
- Early returns on validation failures in `cmd/chronicle/cli/commands/create.go`

## Cross-Cutting Concerns

**Logging:** `internal/log/log.go` provides a singleton logger used across layers
**Validation:** CLI arg validation in `cmd/chronicle/cli/commands/create.go` and `cmd/chronicle/cli/commands/next_version.go`
**Authentication:** Not handled directly in code; GitHub API access is configured via `cmd/chronicle/cli/options/github.go`

---

*Architecture analysis: 2026-02-05*
