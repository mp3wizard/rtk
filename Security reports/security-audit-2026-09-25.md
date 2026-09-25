# Security Audit — 2026-09-25

## Summary
- Issues found: 0 | Auto-fixed: 0 | Unresolved: 0
- Status: PASSED

Merged 9 upstream commits (origin/develop). Scope: Cargo.lock (213 packages) + src/.

## Fixed Issues
None.

## Raw Scanner Output
- osv-scanner (Cargo.lock, 213 packages): No issues found
- cargo-audit: not installed (skipped; osv-scanner covers RustSec advisories)
- gitleaks (git history): 38 hits — all previously triaged false positives (test fixtures in tests/guard_integration_test.rs, aws_cmd.rs examples, scripts/benchmark/cloud-init.yaml, prior audit reports quoting examples). 0 verified secrets.
- semgrep p/rust (src/): 31 informational audit-rule hits (unsafe-usage x14 signal/libc FFI, temp-dir x12, current-exe x2, args x3). No ERROR/WARNING-severity findings; all are expected patterns for a CLI (signal handlers, temp files, std::env::args).
