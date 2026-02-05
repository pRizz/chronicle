# Feature Research

**Domain:** Error-handling audit reports for CLI tools
**Researched:** Thursday Feb 5, 2026
**Confidence:** MEDIUM

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Command coverage inventory | Audits must state what was checked | MEDIUM | Enumerate commands, flags, and execution paths; note gaps or skipped areas |
| Error taxonomy and categorization | Users need to triage and route fixes | MEDIUM | Classify as user input, environment, network, auth, system/bug; align with documented error types where available |
| Actionable message assessment | CLI guidance expects clear fixes | MEDIUM | Evaluate "what happened / why / what to do" and provide suggested copy |
| Exit code and stderr/stdout compliance | Scripts depend on correct signaling | LOW | Verify non-zero exit codes on failure and error output on stderr |
| Repro steps and context | Engineers need to fix issues quickly | MEDIUM | Capture command invocation, inputs, environment notes, and observed output |
| Severity/impact rating | Helps prioritize fixes | LOW | Rate user impact (blocker/major/minor) and scope (single command vs system-wide) |
| Evidence snippets | Audits need traceable proof | LOW | Include error output excerpts or logs; avoid noisy raw stack traces by default |
| Remediation checklist | Teams need a concrete next step list | LOW | Bullet list of fixes with ownership hints (CLI, GitHub, Git, formatting) |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Consistency audit across commands | Prevents fragmented UX and drift | MEDIUM | Cross-check wording, error patterns, and help suggestions for similar failures |
| Suggested fix copy with variants | Reduces rewrite time for maintainers | MEDIUM | Provide recommended phrasing and fallback guidance per error type |
| Machine-readable report output | Enables CI checks and trend tracking | MEDIUM | JSON/CSV export alongside Markdown report |
| Prioritized remediation plan | Focuses limited time on highest value | MEDIUM | Score by frequency, impact, and ease of fix |
| Known-guideline alignment scoring | Ties findings to accepted guidance | LOW | Map findings to CLI UX guidance; cite rule IDs or sections |
| Regression guard recommendations | Prevents reintroducing bad errors | HIGH | Suggest tests or linting rules for critical error paths |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Automatic code changes | "Fix it for me" appeal | Risky without context; breaks workflows | Provide suggested patch snippets and let maintainers decide |
| Single "quality score" | Simple to compare versions | Hides critical issues behind averages | Use severity buckets and top issues list |
| Full stack traces in report by default | More data feels safer | Drowns signal and exposes internal details | Include terse excerpts with optional deep logs |
| Silent normalization of errors | Makes output look clean | Hides mismatches and makes debugging harder | Flag inconsistencies explicitly with evidence |

## Feature Dependencies

```
[Command coverage inventory]
    └──requires──> [Execution harness / ability to trigger errors]
                       └──requires──> [Environment/config capture]

[Error taxonomy and categorization]
    └──requires──> [Command coverage inventory]

[Actionable message assessment]
    └──requires──> [Error taxonomy and categorization]

[Consistency audit across commands] ──enhances──> [Actionable message assessment]

[Machine-readable report output] ──enhances──> [Remediation checklist]
```

### Dependency Notes

- **Error taxonomy and categorization requires command coverage inventory:** you cannot classify what you didn't discover.
- **Actionable message assessment requires error taxonomy:** recommendations depend on error type (user vs system vs external).
- **Consistency audit enhances actionable assessment:** reused phrasing and guidance improve UX and reduce drift.

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept.

- [ ] Command coverage inventory — proves audit scope and gaps
- [ ] Error taxonomy and categorization — enables triage and ownership
- [ ] Actionable message assessment — core value for clearer errors
- [ ] Exit code and stderr/stdout compliance — scripting correctness
- [ ] Remediation checklist — turns report into a to-do list

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Consistency audit across commands — once baseline data exists
- [ ] Suggested fix copy with variants — after taxonomy stabilizes
- [ ] Machine-readable report output — when integrating with CI

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Regression guard recommendations — requires deeper test strategy
- [ ] Prioritized remediation plan — depends on historical usage data

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Command coverage inventory | HIGH | MEDIUM | P1 |
| Actionable message assessment | HIGH | MEDIUM | P1 |
| Consistency audit across commands | MEDIUM | MEDIUM | P2 |
| Machine-readable report output | MEDIUM | MEDIUM | P2 |
| Suggested fix copy with variants | MEDIUM | MEDIUM | P2 |
| Regression guard recommendations | MEDIUM | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | Competitor A | Competitor B | Our Approach |
|---------|--------------|--------------|--------------|
| Actionable errors guidance | CLIG: emphasize human-readable fixes | Azure CLI: structured error types and recommendations | Audit for missing "what/why/fix" and suggest copy |
| stderr/stdout and exit codes | CLIG: stdout for output, stderr for errors, non-zero failures | Azure CLI: error typing and messaging rules | Check for stream and exit code correctness per command |
| Error handling customization | oclif: command-level and global catch | Azure CLI: specific error types and recommendations | Identify where error flow is inconsistent or uncaught |

## Sources

- https://clig.dev/#error-handling
- https://raw.githubusercontent.com/Azure/azure-cli/dev/doc/error_handling_guidelines.md
- https://oclif.io/docs/error_handling

---
*Feature research for: error-handling audit reports for CLI tools*
*Researched: Thursday Feb 5, 2026*
