# Pitfalls Research

**Domain:** Go CLI error-handling audit for Git/GitHub integrations
**Researched:** 2026-02-05
**Confidence:** LOW

## Critical Pitfalls

### Pitfall 1: Conflating user-actionable errors with system failures

**What goes wrong:**
Audit focuses on correctness of text but does not classify errors, so users get the same message for missing config, bad flags, auth failures, and transient network issues. This hides the action users should take next.

**Why it happens:**
CLI error paths are scattered across commands, and audits focus on error strings instead of error taxonomy and exit behavior.

**How to avoid:**
Define a small error taxonomy (user input, environment, auth/permissions, network/transient, internal bug) and map each command's error paths to it. Require every error message to include a next step when user-actionable.

**Warning signs:**
Users report "I got an error but don't know what to do." Errors omit next steps or point to incorrect actions.

**Phase to address:**
Phase 1: Error-path inventory and taxonomy

---

### Pitfall 2: Losing root-cause context through string-only errors

**What goes wrong:**
Audits miss error wrapping or context propagation, so messages look generic ("failed to run git") and omit the underlying cause (exit code, stderr, operation).

**Why it happens:**
Errors are created with formatted strings rather than wrapped with context, and audits read only surfaced messages without tracing the call chain.

**How to avoid:**
Require error wrapping with operation context and preserve original error details (exit code, stderr, HTTP status, request context). Add a checklist for each integration call site.

**Warning signs:**
Errors lack the operation name, a status code, or a command; repeated "failed" messages across different paths.

**Phase to address:**
Phase 1: Error-path inventory and taxonomy

---

### Pitfall 3: Ignoring exit codes and stdout/stderr conventions

**What goes wrong:**
Audit reports improvements to messages, but CLI still exits with 0 on failure, or writes errors to stdout. This breaks scripting and automation.

**Why it happens:**
Audits focus on human-facing text and skip the contract a CLI has with automation (exit codes, stderr usage).

**How to avoid:**
Define an exit code map for common failure categories and enforce stderr for errors. Verify through tests or a matrix in the audit report.

**Warning signs:**
CI or scripts cannot detect failures reliably. Logs show errors in stdout.

**Phase to address:**
Phase 2: CLI command audit and contract validation

---

### Pitfall 4: Missing GitHub API failure modes (rate limits, permissions, pagination)

**What goes wrong:**
Audits assume 404 or empty results mean "not found", missing that it could be permissions or rate limits. Error paths do not surface actionable guidance.

**Why it happens:**
Audits do not model GitHub API failure modes or check for rate limit headers and permission scopes.

**How to avoid:**
Document GitHub API failure modes, map responses to clear user guidance (token scopes, rate limit retry, missing repo access), and verify pagination handling for result sets.

**Warning signs:**
Users see "not found" for private repos they can access with different tokens; rate limit errors appear as generic failures.

**Phase to address:**
Phase 2: Integration audit (GitHub)

---

### Pitfall 5: Overlooking partial-success and side-effect rollback

**What goes wrong:**
An error occurs after creating a release/changelog artifact, but the CLI reports failure without telling the user what was already created. This leads to duplicates or confusion.

**Why it happens:**
Audit focuses on "failure" paths and misses multi-step workflows with side effects.

**How to avoid:**
Identify multi-step commands and document side effects per step. Require error messages to state what was completed and how to recover.

**Warning signs:**
Duplicate releases or changelog entries; users unsure if a partial operation succeeded.

**Phase to address:**
Phase 3: Error message UX and recovery guidance

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| String-only errors without wrapping | Faster to implement | Loses cause context; hard to audit | Never |
| One generic error handler for all commands | Simpler control flow | No command-specific guidance | Only for MVP prototypes |
| No exit code mapping | Less design work | Automation breaks; hard to detect failures | Never |
| Ignoring context cancellation in Git/GitHub calls | Fewer parameters | Hangs on slow networks | Only in local-only commands |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| GitHub API | Treating 404 as missing resource | Distinguish 404 vs 403 and check scopes; surface token guidance |
| GitHub API | Ignoring rate limit headers | Parse rate limit headers; give retry-after guidance |
| GitHub API | Missing pagination | Iterate pages and detect truncated results |
| Git CLI | Assuming git exists and is in PATH | Detect git availability and provide install guidance |
| Git CLI | Swallowing stderr from git | Capture stderr and include in wrapped error context |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| API call per PR/issue without batching | Slow audits and timeouts | Use GraphQL/batched requests or caching | Large repos (1k+ PRs) |
| Full git history scan per command | Long runtimes on big repos | Limit range, use tags or commit window | Repos with long history (10k+ commits) |
| Re-running identical GitHub queries on retries | Rate limits hit quickly | Cache in-memory per run | Multiple retries in CI |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Printing token values in error messages | Secret leakage in logs | Redact auth tokens before logging |
| Echoing raw API responses to stdout | Accidental exposure of sensitive data | Log minimal context to stderr |
| Including local file paths in public output | Leaks developer machine info | Keep paths in debug output only |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Error messages lack next steps | Users stuck and abandon | Include explicit next action or link to docs |
| Inconsistent terminology across commands | Confusion in troubleshooting | Standardize terms (repo, release, changelog) |
| Mixed stdout/stderr output | Scripts break | Use stdout for data, stderr for errors |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Error taxonomy:** Often missing a category for auth/permissions — verify taxonomy is used across commands
- [ ] **Exit codes:** Often missing a map per failure type — verify non-zero exits are consistent
- [ ] **Rate limit handling:** Often missing retry guidance — verify error includes rate limit info
- [ ] **Partial success guidance:** Often missing "what already happened" — verify messages include side effects

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Conflated error types | MEDIUM | Reclassify errors and rewrite messages with action steps |
| Lost root cause context | MEDIUM | Add wrapping at integration boundaries and re-run audit |
| Exit code inconsistencies | LOW | Add exit code map and test with CLI harness |
| Missing GitHub failure modes | MEDIUM | Add rate limit and permission checks; update messages |
| Partial success confusion | HIGH | Add recovery guidance and idempotency checks |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Conflating error types | Phase 1: Inventory + taxonomy | Audit report includes taxonomy mapping |
| Lost root cause context | Phase 1: Inventory + taxonomy | Error samples show wrapped context |
| Exit code inconsistencies | Phase 2: CLI contract audit | CLI harness verifies exit codes and stderr |
| Missing GitHub failure modes | Phase 2: Integration audit | Cases for 403/404/rate limits documented |
| Partial success confusion | Phase 3: Message UX + recovery | Messages list side effects and recovery steps |

## Sources

- Personal experience with Go CLI audits and integration error paths (unverified)

---
*Pitfalls research for: Go CLI error-handling audit*
*Researched: 2026-02-05*
