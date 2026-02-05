# Coding Conventions

**Analysis Date:** 2026-02-05

## Naming Patterns

**Files:**
- Lowercase snake_case for Go files, with `_test.go` for tests, e.g. `chronicle/release/changelog_info.go`, `internal/git/head_test.go`
- Snapshot fixtures live in `__snapshots__`, e.g. `chronicle/release/format/markdown/__snapshots__/presenter_test.snap`
- Test fixtures live under `testdata/`, e.g. `internal/git/testdata/`, `chronicle/release/releasers/github/testdata/`

**Functions:**
- Exported functions/types use PascalCase, unexported use lowerCamelCase, e.g. `internal/git/interface.go`, `chronicle/release/format/markdown/presenter.go`

**Variables:**
- lowerCamelCase for locals/fields, e.g. `chronicle/release/releasers/github/summarizer.go`

**Types:**
- PascalCase for types and interfaces, e.g. `chronicle/release/release.go`, `internal/git/interface.go`

## Code Style

**Formatting:**
- Tool used: `gofmt` + `goimports` (via `golangci-lint` formatters)
- Key settings: configured in `.golangci.yaml` under `formatters`

**Linting:**
- Tool used: `golangci-lint`
- Key rules: enabled linters and limits in `.golangci.yaml` (e.g., `funlen`, `gocyclo`, `gosec`, `revive`, `staticcheck`)

## Import Organization

**Order:**
1. Go standard library
2. Third-party dependencies
3. Local module imports

**Example:** `chronicle/release/format/markdown/presenter_test.go`

**Path Aliases:**
- Not detected; use full module paths like `github.com/anchore/chronicle/...`

## Error Handling

**Patterns:**
- Return `error` from functions; construct errors with `fmt.Errorf`, e.g. `internal/git/interface.go`
- Tests use `require.NoError` for setup/critical errors and `assert.*` for validation, e.g. `chronicle/release/format/markdown/presenter_test.go`

## Logging

**Framework:** `github.com/anchore/go-logger`

**Patterns:**
- Use wrapper functions in `internal/log/log.go` (`Infof`, `Warnf`, `Errorf`, etc.)

## Comments

**When to Comment:**
- Exported types/functions have doc comments, e.g. `chronicle/release/release.go`, `internal/log/log.go`

**JSDoc/TSDoc:**
- Not applicable (Go doc comments instead)

## Function Design

**Size:** Keep helpers small; test helpers call `t.Helper()`, e.g. `chronicle/release/format/markdown/presenter_test.go`

**Parameters:** Prefer explicit parameters; avoid global state (uses small structs and config objects), e.g. `chronicle/release/releasers/github/summarizer.go`

**Return Values:** Return values plus `error` when operations can fail, e.g. `internal/git/interface.go`

## Module Design

**Exports:** Package-level functions/types per directory, e.g. `internal/git`, `chronicle/release`

**Barrel Files:** Not used; import directly from package paths

---

*Convention analysis: 2026-02-05*
