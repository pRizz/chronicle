# Architecture Research

**Domain:** CLI audit/reporting workflow for Go tooling
**Researched:** 2026-02-05
**Confidence:** MEDIUM

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        CLI Interface                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ Command │  │ Config/CLI │  │ Target Scope │  │ Logger   │ │
│  └────┬────┘  └─────┬──────┘  └──────┬───────┘  └────┬─────┘ │
│       │            │               │                │       │
├───────┴────────────┴───────────────┴────────────────┴───────┤
│                        Audit Pipeline                        │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────┐ → ┌───────────┐ → ┌────────────┐ → ┌───────┐ │
│  │ Collectors │   │ Analyzers │   │ Postprocess│   │ Report│ │
│  └─────┬──────┘   └─────┬─────┘   └─────┬──────┘   └───┬───┘ │
│        │                │              │               │    │
├────────┴────────────────┴──────────────┴───────────────┴────┤
│                    Data/Integration Layer                    │
│  ┌───────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐ │
│  │ Git/GH    │  │ File System│  │ Config     │  │ Caches   │ │
│  └───────────┘  └────────────┘  └────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Command/router | Parse flags, choose audit scope, orchestrate run | Cobra/urfave CLI command handler |
| Config | Merge CLI flags + config file + defaults | Config loader + validation |
| Target scope | Determine what to audit (commands, integrations, files) | Target registry + filters |
| Collectors | Extract facts from codebase and runtime context | AST walkers, file readers, Git/GH clients |
| Analyzers | Apply audit rules to collected facts | Rule modules returning findings |
| Postprocess | Normalize, dedupe, severity, sort | Issue processors + filters |
| Reporter | Format findings into Markdown/JSON/CLI output | Report writer + template renderer |
| Storage/caches | Optional memoization of expensive data | In-memory cache keyed by path/sha |

## Recommended Project Structure

```
cmd/
└── chronicle/
    └── cli/
        └── commands/
            └── audit.go              # CLI entrypoint for audit command
internal/
└── audit/
    ├── pipeline/                     # Orchestrates the audit run
    ├── scope/                        # Target selection and filters
    ├── collect/                      # Collectors (AST, files, Git, GitHub)
    ├── analyze/                      # Analyzer rules (error paths, messages)
    ├── issue/                        # Finding model + severity taxonomy
    ├── postprocess/                  # Dedup/sort/normalize processors
    └── report/                       # Markdown writer for reports
internal/
└── integrations/
    ├── git/                          # Git helpers used by collectors
    └── github/                       # GitHub API helpers used by collectors
.planning/
└── reports/
    └── error-audit.md                # Output report target
```

### Structure Rationale

- **`internal/audit/`:** keeps audit-specific logic isolated from release/changelog logic.
- **`internal/audit/collect/` + `internal/audit/analyze/`:** separates data gathering from rule evaluation so analyzers stay pure and testable.
- **`internal/audit/report/`:** isolates formatting and output paths for easy report changes.

## Architectural Patterns

### Pattern 1: Pipeline (collect → analyze → postprocess → report)

**What:** A linear pipeline where each stage consumes and emits well-defined data.
**When to use:** Any audit/reporting workflow that needs predictable execution and testable stages.
**Trade-offs:** Simple and debuggable; less flexible for branching workflows without extra orchestration.

**Example:**
```go
type Collector interface {
	Collect(ctx context.Context, scope Scope) (Facts, error)
}

type Analyzer interface {
	Analyze(ctx context.Context, facts Facts) ([]Issue, error)
}
```

### Pattern 2: Analyzer registry + rule modules

**What:** Register analyzers (rules) and execute them as a set; each emits issues.
**When to use:** When audit rules grow and need modular ownership and testing.
**Trade-offs:** Easy extensibility; requires consistent issue schema and normalization.

**Example:**
```go
type AnalyzerFunc func(context.Context, Facts) ([]Issue, error)

var Registry = []AnalyzerFunc{
	CheckErrorWrapping,
	CheckUserFacingMessages,
}
```

### Pattern 3: Issue postprocessing chain

**What:** A chain of processors that filter, dedupe, rank, and format issues.
**When to use:** When multiple analyzers overlap or issue prioritization matters.
**Trade-offs:** Adds a stage but enables consistent output and configurable severity rules.

## Data Flow

### Request Flow

```
CLI: chronicle audit
    ↓
Config + Scope resolution
    ↓
Collectors → Facts
    ↓
Analyzers → Issues
    ↓
Postprocess (dedupe, severity, sort)
    ↓
Reporter → .planning/reports/error-audit.md
```

### Key Data Flows

1. **Command audit flow:** CLI command → scope builder → pipeline → report writer.
2. **Integration audit flow:** Git/GitHub clients → collectors → facts → analyzers.

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| Single repo | In-memory facts + sequential analyzers are fine |
| Many packages in repo | Parallelize collectors/analyzers; cache AST and file reads |
| Multi-repo audits | Add workspace registry + persistent cache keyed by repo/sha |

### Scaling Priorities

1. **First bottleneck:** Repeated parsing and GitHub API calls; cache and rate-limit.
2. **Second bottleneck:** Large issue lists; stream postprocess and report rendering.

## Anti-Patterns

### Anti-Pattern 1: Mixing collection and analysis

**What people do:** Analyzers read files and make API calls directly.
**Why it's wrong:** Hard to test, hard to cache, and causes repeated I/O.
**Do this instead:** Keep I/O in collectors; analyzers operate on facts only.

### Anti-Pattern 2: Report formatting inside analyzers

**What people do:** Analyzers emit Markdown strings.
**Why it's wrong:** Output coupling makes it hard to change report format later.
**Do this instead:** Emit structured issues; format at the reporter stage.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| GitHub API | Client wrapper + rate-limit handling | Needed for PR/issue metadata |
| Git CLI | Adapter around git commands | Needed for tags, history, refs |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| CLI ↔ Audit pipeline | Direct function call | Keep CLI thin, pipeline owns workflow |
| Collectors ↔ Analyzers | Facts data model | Stable schema prevents tight coupling |
| Analyzers ↔ Reporter | Issue model | Normalize severity and location fields |

## Sources

- https://pkg.go.dev/golang.org/x/tools/go/analysis
- https://golangci-lint.run/docs/contributing/architecture
- https://opentelemetry.io/docs/collector/architecture

---
*Architecture research for: CLI audit/reporting workflow in Chronicle*
*Researched: 2026-02-05*
