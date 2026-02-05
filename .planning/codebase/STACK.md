# Technology Stack

**Analysis Date:** 2026-02-05

## Languages

**Primary:**
- Go 1.24.0 - CLI app, core library, internal utilities in `go.mod` and `cmd/chronicle/` and `chronicle/`

**Secondary:**
- Shell (sh) - repo scripts in `install.sh` and `internal/git/testdata/*.sh`

## Runtime

**Environment:**
- Go toolchain 1.24.0 configured in `go.mod`

**Package Manager:**
- Go modules in `go.mod`
- Lockfile: present in `go.sum`

## Frameworks

**Core:**
- Cobra 1.10.2 - CLI command structure in `cmd/chronicle/cli/commands/*.go` and dependency in `go.mod`
- Anchore clio - CLI framework and config loading in `cmd/chronicle/main.go` and dependency in `go.mod`
- go-git v5.16.2 - git operations in `internal/git/*.go` and dependency in `go.mod`

**Testing:**
- Testify 1.10.0 - assertions in `internal/git/*_test.go` and dependency in `go.mod`
- go-snaps 0.5.19 - snapshot tests in `chronicle/release/format/markdown/presenter_test.go` and dependency in `go.mod`
- go-cmp 0.7.0 - comparisons in `internal/git/tag_test.go` and dependency in `go.mod`

**Build/Dev:**
- go-make 0.0.2 - task runner in `.make/main.go` and dependency in `.make/go.mod`
- goreleaser - release packaging config in `.goreleaser.yaml`
- golangci-lint - lint config in `.golangci.yaml`

## Key Dependencies

**Critical:**
- githubv4 - GitHub GraphQL queries in `chronicle/release/releasers/github/*.go` and dependency in `go.mod`
- oauth2 - GitHub token auth in `chronicle/release/releasers/github/*.go` and dependency in `go.mod`
- go-git - local repo interrogation in `internal/git/*.go` and dependency in `go.mod`

**Infrastructure:**
- go-partybus - event bus abstraction in `internal/bus/bus.go` and dependency in `go.mod`
- go-presenter - output presentation in `cmd/chronicle/cli/commands/create_presenter.go` and dependency in `go.mod`

## Configuration

**Environment:**
- App config file `.chronicle.yaml` (log level) in `.chronicle.yaml`
- CLI config mapping in `cmd/chronicle/cli/options/*.go`

**Build:**
- Build orchestration in `Makefile` and `.make/main.go`
- Release build targets and packaging in `.goreleaser.yaml`
- Linting and formatting rules in `.golangci.yaml`

## Platform Requirements

**Development:**
- Go 1.24.0 required by `go.mod`
- go-make tasks executed via `Makefile` and `.make/main.go`

**Production:**
- Cross-compiled binaries for linux/darwin/windows configured in `.goreleaser.yaml`

---

*Stack analysis: 2026-02-05*
