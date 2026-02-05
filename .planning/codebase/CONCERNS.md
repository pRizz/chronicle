# Codebase Concerns

**Analysis Date:** 2026-02-05

## Tech Debt

**GitHub API clients:**
- Issue: Auth token creation and rate-limit handling are duplicated with TODOs for DI and rate limit handling.
- Files: `chronicle/release/releasers/github/gh_release.go`, `chronicle/release/releasers/github/gh_pull_request.go`, `chronicle/release/releasers/github/gh_issue.go`
- Impact: Makes testing harder and hides rate-limit failure paths.
- Fix approach: Centralize GraphQL client construction with injected auth/token source and explicit rate-limit handling.

**Git remote parsing:**
- Issue: Remote URL parsing relies on a regex and assumes an `origin` remote.
- Files: `internal/git/remote.go`
- Impact: Fails for non-standard git config layouts, missing origin, or enterprise host naming.
- Fix approach: Use go-git config parsing (or git plumbing) with fallback handling for non-origin remotes.

**Changelog speculation behavior:**
- Issue: Version speculation behavior is hard-coded with TODO to make configurable.
- Files: `chronicle/release/changelog_info.go`
- Impact: Limits control over SemVer bump policy and can produce unexpected versions.
- Fix approach: Expose speculation behavior via config/CLI flags and persist to config.

**Changelog notice content:**
- Issue: Notice field is always empty with TODO placeholder.
- Files: `chronicle/release/changelog_info.go`
- Impact: Downstream presenters cannot show release notices even when needed.
- Fix approach: Add notice generation based on config or repository metadata.

**Provider support:**
- Issue: Worker selection only supports GitHub; other providers are not implemented.
- Files: `cmd/chronicle/cli/commands/create.go`
- Impact: Limits adoption in GitLab/Bitbucket or non-GitHub workflows.
- Fix approach: Add provider interface and corresponding workers, then expose via CLI.

**Change type source of truth:**
- Issue: Change type titles are reconstructed in the CLI with TODO for a single source of truth.
- Files: `cmd/chronicle/cli/commands/create_github_worker.go`
- Impact: Risk of drift between config and change type definitions.
- Fix approach: Centralize change type definitions in a shared package or config loader.

## Known Bugs

**File handle leak in RemoteURL:**
- Symptoms: `.git/config` file is opened without being closed.
- Files: `internal/git/remote.go`
- Trigger: Any call to `RemoteURL` on a repository.
- Workaround: None.

**Silent empty remote URL:**
- Symptoms: Returns an empty URL with no error when no `origin` entry matches.
- Files: `internal/git/remote.go`
- Trigger: Repos without an `origin` remote or with non-matching config formatting.
- Workaround: Ensure an `origin` remote exists with the expected config format.

**Tag-only lookup for refs:**
- Symptoms: `SearchForTag` fails for commit hashes or other tree-ish refs.
- Files: `internal/git/tag.go`
- Trigger: Users pass commit refs for `since`/`until` in changelog generation.
- Workaround: Use tag refs only.

## Security Considerations

**Not detected:**
- Risk: Not detected in code review of `internal/git/*.go` and `chronicle/release/releasers/github/*.go`.
- Files: `internal/git/*.go`, `chronicle/release/releasers/github/*.go`
- Current mitigation: Standard Go library usage.
- Recommendations: Add token validation and request timeouts for external API calls.

## Performance Bottlenecks

**Full-history GraphQL pagination:**
- Problem: Fetchers page through all releases, issues, and PRs with no early exit by date range.
- Files: `chronicle/release/releasers/github/gh_release.go`, `chronicle/release/releasers/github/gh_issue.go`, `chronicle/release/releasers/github/gh_pull_request.go`
- Cause: Pagination always continues until `HasNextPage` is false.
- Improvement path: Add date-based early exit and server-side query constraints.

**Heavy GraphQL payloads:**
- Problem: Queries include labels and closing issues, increasing response size.
- Files: `chronicle/release/releasers/github/gh_pull_request.go`, `chronicle/release/releasers/github/gh_issue.go`
- Cause: Nested fields are always requested regardless of config needs.
- Improvement path: Make query fields conditional based on configuration flags.

## Fragile Areas

**Git config parsing:**
- Files: `internal/git/remote.go`
- Why fragile: Regex assumes specific formatting and an `origin` remote.
- Safe modification: Add robust parsing with clear fallbacks and explicit errors.
- Test coverage: `internal/git/remote_test.go` lacks cases for missing origin and alternate remote names.

**Changelog end tag logic:**
- Files: `chronicle/release/releasers/github/find_changelog_end_tag.go`
- Why fragile: Any error fetching a release fails the operation; TODO indicates missing error-type checks.
- Safe modification: Distinguish not-found errors from transient API failures.
- Test coverage: `chronicle/release/releasers/github/find_changelog_end_tag_test.go` lacks transient error scenarios.

**GitHub URL parsing:**
- Files: `chronicle/release/releasers/github/summarizer.go`
- Why fragile: Only handles `git@` and `http(s)` URLs; other schemes return empty owner/repo.
- Safe modification: Add support for `ssh://` and validate host against config.
- Test coverage: `chronicle/release/releasers/github/summarizer_test.go` does not cover `ssh://` URLs.

**First-release flow:**
- Files: `chronicle/release/changelog_info.go`
- Why fragile: Returns an error when no releases exist (TODO for first-release support).
- Safe modification: Fall back to first commit date for initial release.
- Test coverage: `chronicle/release/changelog_info_test.go` does not cover no-release repos.

## Scaling Limits

**GitHub API rate limits:**
- Current capacity: Unbounded pagination across PRs/issues/releases.
- Limit: Exhausts GitHub GraphQL rate limits on large repos.
- Scaling path: Rate-limit awareness, backoff, and cached checkpoints.
- Files: `chronicle/release/releasers/github/gh_release.go`, `chronicle/release/releasers/github/gh_pull_request.go`, `chronicle/release/releasers/github/gh_issue.go`

## Dependencies at Risk

**Not detected:**
- Risk: Not detected in dependency set.
- Impact: None observed.
- Migration plan: Not applicable.
- Files: `go.mod`

## Missing Critical Features

**Non-GitHub providers:**
- Problem: Only GitHub is supported for changelog generation.
- Blocks: GitLab/Bitbucket users cannot generate changelogs.
- Files: `cmd/chronicle/cli/commands/create.go`

## Test Coverage Gaps

**CLI flows untested:**
- What's not tested: CLI flag parsing and command execution paths.
- Files: `cmd/chronicle/cli/commands/*.go`
- Risk: CLI regressions without coverage.
- Priority: Medium

**JSON presenter untested:**
- What's not tested: JSON changelog output formatting.
- Files: `chronicle/release/format/json/presenter.go`
- Risk: Output changes or regressions go undetected.
- Priority: Medium

**GitHub release fetcher untested:**
- What's not tested: GraphQL release query parsing and error handling.
- Files: `chronicle/release/releasers/github/gh_release.go`
- Risk: API contract changes break release lookup without tests.
- Priority: Medium

---

*Concerns audit: 2026-02-05*
