# Security Audit — 2026-10-08

## Summary
- Issues found: 4 | Auto-fixed: 4 | Unresolved: 0
- Status: ISSUES FIXED
- Merged 99 upstream commits (origin/develop); Git HEAD at scan: ea13e1ce. Scan target: this worktree. Tools run: OSV-Scanner, Trivy, Gitleaks, Semgrep (p/secrets + p/rust). Not run: Bandit (no meaningful Python), TruffleHog, CodeQL, mcp-* scanners.

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | crossbeam-epoch (Cargo.lock) | RUSTSEC-2026-0204 | 0.9.18 → 0.9.21 via cargo update |
| 2 | quick-xml (Cargo.lock) | RUSTSEC-2026-0194, RUSTSEC-2026-0195 (CVSS 7.5) | 0.37.5 → 0.41.0 (lockfile re-synced to Cargo.toml's "0.41" after conflict resolution kept upstream's stale lock) |
| 3 | rustls (Cargo.lock) | RUSTSEC-2026-0285 / GHSA-2mjx-qc3c-rqvc (CVSS 5.3) | 0.23.37 → 0.23.45 (rustls-webpki 0.103.13 → 0.103.15 pulled in) |

Re-scan with `osv-scanner scan -L Cargo.lock`: No issues found.

## Triaged, no action
- Gitleaks: 38 hits — all fixtures in aws_cmd.rs / guard_integration_test.rs / benchmark cloud-init.yaml and quoted text in prior reports. False positives.
- Semgrep: 15 blocking heuristic hits (unsafe-usage x6, args x2, args-os x1, current-exe x3, temp-dir x3). Intentional CLI patterns (libc signal handling, env::args, unique-suffixed temp files); not auto-fixable without behavior change. Informational.

## Raw Scanner Output
Raw outputs (OSV table before fix):

```
+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE         | VERSION | FIXED VERSION | SOURCE     |
+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
| https://osv.dev/RUSTSEC-2026-0204   |      | crates.io | crossbeam-epoch | 0.9.18  | 0.9.20        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0194   | 7.5  | crates.io | quick-xml       | 0.37.5  | 0.41.0        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0195   | 7.5  | crates.io | quick-xml       | 0.37.5  | 0.41.0        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0285   | 5.3  | crates.io | rustls          | 0.23.37 | 0.23.45       | Cargo.lock |
| https://osv.dev/GHSA-2mjx-qc3c-rqvc |      |           |                 |         |               |            |
+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
```

Trivy (before fix): Cargo.lock — rustls GHSA-2mjx-qc3c-rqvc MEDIUM, fixed 0.23.45. Gitleaks: 1987 commits scanned, 38 leaks (triaged above). Semgrep: 26 findings / 15 blocking rule hits, 457 files.
