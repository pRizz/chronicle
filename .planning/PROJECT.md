# Chronicle Error Handling Audit

## What This Is

An audit of the Chronicle CLI and Git/GitHub integrations to surface bugs and improve error messages so they are descriptive and actionable. The immediate output is a structured report, with fixes planned afterward.

## Core Value

Users can understand and act on any error without guesswork.

## Requirements

### Validated

- ✓ CLI can generate changelogs from GitHub issues/PRs — existing
- ✓ CLI can compute next version from GitHub changes — existing
- ✓ Output supports Markdown and JSON formats — existing

### Active

- [ ] Produce a comprehensive error audit report for CLI commands, Git/GitHub integrations, and release/changelog generation
- [ ] Identify bugs and error paths with unclear or missing actionability guidance
- [ ] Recommend actionable error messages with concrete remediation or workaround steps

### Out of Scope

- Implementing fixes during the audit phase — defer until after report
- Expanding audit to non-prioritized areas beyond CLI, Git/GitHub, and release/changelog flows — defer unless findings require it

## Context

- Go-based CLI (`cmd/chronicle/cli`) with domain logic in `chronicle/release/` and GitHub adapters in `chronicle/release/releasers/github/`.
- Prior issue: creating a GitHub release failed due to missing existing release without actionable error guidance.
- Audit output should live at `.planning/reports/error-audit.md`.

## Constraints

- **Tech stack**: Go with Cobra/clio — align changes to existing patterns
- **Process**: Audit report first, fixes later — maintain separation
- **Reporting**: Include severity, file/symbol, reproduction, current behavior, recommended message/action — required structure
- **Testing**: Update tests when error messages change — required

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Audit before fixes | Ensure coverage and prioritization before changes | — Pending |
| Focus scope on CLI, Git/GitHub, release/changelog | Highest user-facing impact | — Pending |
| Report location at `.planning/reports/error-audit.md` | Keep audit artifacts under planning | — Pending |

---
*Last updated: 2026-02-05 after initialization*
