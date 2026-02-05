# Testing Patterns

**Analysis Date:** 2026-02-05

## Test Framework

**Runner:**
- Go toolchain `go test`
- Config: Not detected (default Go test configuration)

**Assertion Library:**
- `github.com/stretchr/testify` (`assert`, `require`) in tests like `internal/git/head_test.go`, `chronicle/release/releasers/github/gh_issue_test.go`

**Run Commands:**
```bash
make test          # Run test suite via .make tasks (Makefile)
make unit          # Unit tests (documented in DEVELOPING.md)
make snapshot      # Update snapshot tests
```

## Test File Organization

**Location:**
- Co-located with source files in the same package directory, e.g. `internal/git/head_test.go` next to `internal/git/head.go`

**Naming:**
- `*_test.go` for test files, e.g. `chronicle/release/changelog_info_test.go`

**Structure:**
```
internal/git/
  head.go
  head_test.go
chronicle/release/format/markdown/
  presenter.go
  presenter_test.go
  __snapshots__/presenter_test.snap
```

## Test Structure

**Suite Organization:**
```go
tests := []struct {
	name    string
	path    string
	expects string
}{ /* ... */ }

for _, test := range tests {
	t.Run(test.name, func(t *testing.T) {
		actual, err := HeadTag(test.path)
		assert.NoError(t, err)
		assert.Equal(t, test.expects, actual)
	})
}
```
Example from `internal/git/head_test.go`.

**Patterns:**
- Table-driven tests with `t.Run`, e.g. `internal/git/head_test.go`, `chronicle/release/releasers/github/gh_issue_test.go`
- `require.NoError` for setup checks, `assert.*` for expectations, e.g. `chronicle/release/format/markdown/presenter_test.go`

## Mocking

**Framework:** None detected (use hand-rolled fakes/stubs)

**Patterns:**
```go
type mockGitter struct {
	timestamp time.Time
	git.Interface
}

func (m mockGitter) SearchForTag(tagRef string) (*git.Tag, error) {
	a, err := m.Interface.SearchForTag(tagRef)
	if a != nil {
		a.Timestamp = m.timestamp
	}
	return a, err
}
```
Example from `chronicle/release/releasers/github/summarizer_test.go`.

**What to Mock:**
- Interfaces for external dependencies (git, release fetchers) with in-test structs or function fields, e.g. `chronicle/release/releasers/github/summarizer_test.go`

**What NOT to Mock:**
- Small pure functions; test directly with table cases, e.g. `chronicle/release/releasers/github/gh_issue_test.go`

## Fixtures and Factories

**Test Data:**
```go
snaps.MatchSnapshot(t, string(actual))
```
Snapshot tests in `chronicle/release/format/markdown/presenter_test.go` with outputs in `chronicle/release/format/markdown/__snapshots__/presenter_test.snap`.

**Location:**
- `testdata/` directories hold git repo fixtures and scripts, e.g. `internal/git/testdata/`, `chronicle/release/releasers/github/testdata/`

## Coverage

**Requirements:** Not detected

**View Coverage:**
```bash
go test ./... -cover
```

## Test Types

**Unit Tests:**
- Core logic verified with table-driven tests and assertions, e.g. `internal/git/head_test.go`, `chronicle/release/format/markdown/presenter_test.go`

**Integration Tests:**
- Git-based tests use fixture repositories under `testdata/` and invoke `git` commands, e.g. `chronicle/release/releasers/github/summarizer_test.go`

**E2E Tests:**
- Not detected

## Common Patterns

**Async Testing:**
```go
// Not detected (no async patterns observed in tests like `internal/git/head_test.go`)
```

**Error Testing:**
```go
assert.NoError(t, err)
require.NoError(t, err)
```
Used across tests such as `internal/git/head_test.go` and `chronicle/release/format/markdown/presenter_test.go`.

---

*Testing analysis: 2026-02-05*
