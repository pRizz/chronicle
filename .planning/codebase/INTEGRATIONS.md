# External Integrations

**Analysis Date:** 2026-02-05

## APIs & External Services

**GitHub:**
- GitHub GraphQL API v4 - release/issue/PR queries in `chronicle/release/releasers/github/*.go`
  - SDK/Client: `github.com/shurcooL/githubv4` in `go.mod`
  - Auth: `GITHUB_TOKEN` read in `chronicle/release/releasers/github/gh_issue.go`

**Notifications:**
- Slack webhook for release notifications in `.github/workflows/release.yaml`
  - SDK/Client: GitHub Action `8398a7/action-slack` in `.github/workflows/release.yaml`
  - Auth: `SLACK_WEBHOOK_URL` in `.github/workflows/release.yaml`

## Data Storage

**Databases:**
- Not detected (no DB clients in `go.mod`)
  - Connection: Not applicable in `go.mod`
  - Client: Not applicable in `go.mod`

**File Storage:**
- Local filesystem only (no storage SDKs in `go.mod`)

**Caching:**
- None detected in `go.mod`

## Authentication & Identity

**Auth Provider:**
- GitHub token auth for API calls in `chronicle/release/releasers/github/*.go`
  - Implementation: OAuth2 static token source in `chronicle/release/releasers/github/gh_pull_request.go`

## Monitoring & Observability

**Error Tracking:**
- None detected (no error tracking SDKs in `go.mod`)

**Logs:**
- Structured logging via `github.com/anchore/go-logger` in `internal/log/log.go`

## CI/CD & Deployment

**Hosting:**
- GitHub Releases (built via goreleaser) configured in `.goreleaser.yaml`

**CI Pipeline:**
- GitHub Actions workflows in `.github/workflows/*.yaml`
- CodeQL scanning in `.github/workflows/codeql-analysis.yml`
- GitHub Actions linting with zizmor in `.github/workflows/validate-github-actions.yaml`
- Release pipeline and tagging in `.github/workflows/release.yaml`
- Dependabot automation via shared workflow in `.github/workflows/dependabot-automation.yaml`

## Environment Configuration

**Required env vars:**
- `GITHUB_TOKEN` for GitHub API and release tagging in `chronicle/release/releasers/github/gh_release.go` and `.github/workflows/release.yaml`
- `SLACK_WEBHOOK_URL` for release notifications in `.github/workflows/release.yaml`
- `OSS_PROJECT_GH_TOKEN` for OSS project board automation in `.github/workflows/oss-project-board-add.yaml`
- `OSS_R2_INSTALL_ACCESS_KEY_ID` and `OSS_R2_INSTALL_SECRET_ACCESS_KEY` for install script release in `.github/workflows/release.yaml`
- `TOOLBOX_CLOUDFLARE_R2_ENDPOINT` for install script release in `.github/workflows/release.yaml`
- `TOOLBOX_AWS_ACCESS_KEY_ID` and `TOOLBOX_AWS_SECRET_ACCESS_KEY` for install script release in `.github/workflows/release.yaml`

**Secrets location:**
- GitHub Actions secrets referenced in `.github/workflows/release.yaml` and `.github/workflows/oss-project-board-add.yaml`

## Webhooks & Callbacks

**Incoming:**
- None; automation is GitHub Actions triggers in `.github/workflows/*.yaml`

**Outgoing:**
- Slack webhook for release notifications in `.github/workflows/release.yaml`

---

*Integration audit: 2026-02-05*
