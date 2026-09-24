# Security Audit — 2026-09-24

## Summary
- Issues found: 0 | Auto-fixed: 0 | Unresolved: 0
- Status: PASSED
- Merged 2 upstream commits (develop): telemetry command-label fix (src/core/tracking.rs, npm_cmd.rs, ruff_cmd.rs, discover/rules.rs). No dependency changes.

## Fixed Issues
None.

## Raw Scanner Output
Target: worktree dazzling-mccarthy-c6a2e3, HEAD before commit 44f30642

### OSV-Scanner
Scanned Cargo.lock (213 packages) + 7 pom.xml fixtures: "No issues found"

### Trivy (vuln, secret, misconfig)
Cargo.lock: 0 vulnerabilities; pom.xml fixtures: 0; oc_pods.json: 0 misconfigurations; 0 secrets.

### Gitleaks (filesystem, --no-git, 440 MB, 36 hits) — triaged all as false positives
- Security reports/*.md (past audit reports quoting example tokens): stripe-access-token / generic-api-key
- SECURITY.md:153, tests/guard_integration_test.rs:168,186: documentation/test fixtures for the guard feature (fake stripe-style tokens)
- src/cmds/cloud/aws_cmd.rs:1864,1865,1909,2023: AWS CLI JSON test fixtures (nextForwardToken etc.)
- scripts/benchmark/cloud-init.yaml:282: benchmark placeholder API_KEY
Values redacted; none are live credentials.

### Semgrep (p/secrets, p/rust on src/)
Only findings: "Detected 'unsafe' usage" on libc::kill/waitpid/signal/raise in signal handling (known, intentional, audited in prior runs). No secrets.

### config-audit.py
Only LOW findings: plugin hooks.json presence (environment, not repo).

### Skipped
Bandit (no Python), TruffleHog/CodeQL/mcp-scan/skillspector (not run; scope-limited autonomous run).
