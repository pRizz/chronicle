 # Stack Research
 
 **Domain:** Go CLI codebase audit (error handling + reporting)
 **Researched:** 2026-02-05
 **Confidence:** MEDIUM
 
 ## Recommended Stack
 
 ### Core Technologies
 
 | Technology | Version | Purpose | Why Recommended |
 |------------|---------|---------|-----------------|
 | Go toolchain | 1.24.x | Build + first-party analyzers | Matches project `go.mod`, ensures toolchain parity for vet/test. Confidence: HIGH |
 | golangci-lint | v2.8.0 | Aggregate linters + reporting | Standard Go lint runner with built-in output formats including SARIF/JSON for audit reports. Confidence: HIGH |
 | govulncheck | v1.1.4 | Dependency vulnerability + reachable-call analysis | Official Go vulnerability scanner with JSON/SARIF/OpenVEX outputs for reporting. Confidence: HIGH |
 
 ### Supporting Libraries
 
 | Library | Version | Purpose | When to Use |
 |---------|---------|---------|-------------|
 | Staticcheck | 2025.1.1 | Deeper correctness checks | Run standalone for JSON streaming output or tighter rule control beyond golangci-lint defaults. Confidence: HIGH |
 | gosec | v2.22.11 | Security-focused static analysis | Use when CLI touches filesystem, networking, or crypto paths that need security findings. Confidence: HIGH |
 | errcheck | v1.9.0 | Detect unchecked errors | Use when auditing actionable error paths and missed error handling. Confidence: HIGH |
 | nilaway | v0.0.0-20251208195206-89df5f7e6199 | Nil panic detection | Optional for high-risk packages; untagged release implies extra validation. Confidence: MEDIUM |
 
 ### Development Tools
 
 | Tool | Purpose | Notes |
 |------|---------|-------|
 | `go vet` | Heuristic bug detection | Supports JSON output; use alongside linters, not alone. |
 | `go test -json` | Machine-readable test results | Standard JSON stream via `test2json`, useful for audit evidence. |
 
 ## Installation
 
 ```bash
 # Core
 go install github.com/golangci/golangci-lint/cmd/golangci-lint@v2.8.0
 go install golang.org/x/vuln/cmd/govulncheck@v1.1.4
 
 # Supporting
 go install honnef.co/go/tools/cmd/staticcheck@2025.1.1
 go install github.com/securego/gosec/v2/cmd/gosec@v2.22.11
 go install github.com/kisielk/errcheck@v1.9.0
 go install go.uber.org/nilaway/cmd/nilaway@v0.0.0-20251208195206-89df5f7e6199
 ```
 
 ## Alternatives Considered
 
 | Recommended | Alternative | When to Use Alternative |
 |-------------|-------------|-------------------------|
 | golangci-lint | Multiple standalone linters only | Use if you need per-linter config isolation or very custom output pipelines. |
 | govulncheck | Third-party SCA scanners | Use only if you must unify with a company-wide SCA product. |
 
 ## What NOT to Use
 
 | Avoid | Why | Use Instead |
 |-------|-----|-------------|
 | `go vet` alone | Official docs note vet uses heuristics and is not comprehensive. | Combine golangci-lint + staticcheck + govulncheck. |
 | Unversioned tool installs | Breaks audit repeatability and report diffs. | Pin exact versions as above. |
 
 ## Stack Patterns by Variant
 
 **If you need GitHub code-scanning or CI artifacts:**
 - Use `golangci-lint` and `govulncheck` SARIF outputs.
 - Because SARIF integrates cleanly with GitHub code scanning.
 
 **If you need a human audit report:**
 - Use JSON outputs and a small report aggregator to emit Markdown.
 - Because JSON is stable and easy to post-process into `.planning/reports/error-audit.md`.
 
 ## Version Compatibility
 
 | Package A | Compatible With | Notes |
 |-----------|-----------------|-------|
 | Go 1.24.x | Staticcheck 2025.1.1 | Release notes add Go 1.24 support. |
 | Go 1.24.x | errcheck v1.9.0 | Release notes drop Go 1.18–1.21 support. |
 | golangci-lint v2.8.0 | SARIF output | Output configuration supports SARIF format. |
 
 ## Sources
 
 - https://github.com/golangci/golangci-lint/releases — latest release (v2.8.0)
 - https://golangci-lint.run/docs/configuration/file/ — output formats incl. SARIF/JSON
 - https://staticcheck.dev/changes/2025.1 — 2025.1/2025.1.1 release notes
 - https://staticcheck.dev/docs/running-staticcheck/cli/formatters/ — output formats
 - https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck — version + SARIF/JSON/OpenVEX
 - https://go.dev/cmd/vet — vet capabilities and JSON output
 - https://pkg.go.dev/cmd/test2json — `go test -json` output format
 - https://github.com/securego/gosec/releases — latest gosec version (v2.22.11)
 - https://github.com/kisielk/errcheck/releases — latest errcheck version (v1.9.0)
 - https://pkg.go.dev/go.uber.org/nilaway/cmd/nilaway — latest pseudo-version
 
 ---
 *Stack research for: Go CLI codebase audit (error handling + reporting)*
 *Researched: 2026-02-05*
