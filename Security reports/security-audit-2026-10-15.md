# Security Audit — 2026-10-15

## Summary
- Issues found: 8 (OSV-Scanner, 4 packages) | Auto-fixed: 5 (anyhow 1, rustls 4) | Unresolved: 3 (lru 2, paste 1)
- Status: ISSUES REMAINING

Merged: fast-forward to `fork/develop` (e5633016, base for HEAD), then `origin/develop` merged cleanly (117 upstream commits, no conflicts). Scan target: this worktree, Cargo.lock after merge.

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | anyhow (Cargo.lock) | RUSTSEC-2026-0190 | 1.0.102 → 1.0.103 via `cargo update -p anyhow --precise 1.0.103` |
| 2 | rustls (Cargo.lock, via ureq) | RUSTSEC-2026-0346 (CVSS 4.0), -0347 (2.3), -0348 (3.7), -0349 (3.7) | 0.23.45 → 0.23.46 via `cargo update -p rustls --precise 0.23.46` |

## Unresolved Issues
| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| 3 | lru (via ratatui 0.29) | RUSTSEC-2026-0002 (CVSS 2.7), GHSA-rhfx-m35p-ff5j, RUSTSEC-2026-0253 | Fix needs lru ≥ 0.16.3 / 0.18.2. `ratatui 0.29` pins `lru ^0.12`, so `cargo update --precise` fails. Fix needs a ratatui major/minor upgrade, which is a breaking change. Not done automatically. Low severity (unsound `IterMut`, TUI-only path). |
| 4 | paste 1.0.15 (via ratatui) | RUSTSEC-2024-0436 | Unmaintained advisory, no fixed version exists. Same ratatui upgrade question. |

## Triaged, no action
- Gitleaks: 38 leaks across 2079 commits. All are test fixtures (aws_cmd.rs, guard_integration_test.rs, benchmark cloud-init.yaml) and quoted text in prior reports. False positives. Same set as 2026-10-08.
- Trivy (vuln + secret): Cargo.lock rustls GHSA-2mjx-qc3c-rqvc is fixed by item 2. Remaining LOW lru GHSA-rhfx-m35p-ff5j is item 3. Secrets: none. `proxy/Cargo.lock` and the pom.xml fixtures are clean.
- Semgrep p/rust + p/secrets: blocking heuristic hits in `unsafe` libc signal calls, `std::env::args()` / `args_os()`, and `std::env::current_exe()` (src/llm_proxy.rs and the main CLI). These are intentional CLI patterns, not auto-fixable without behavior change. Informational, same classification as previous audits.
- Bandit, CodeQL, mcp-scan, skillspector: not run (no Python source; CodeQL not configured for this run; mcp-scan is opt-in and requires consent).

## Build
- `cargo install --path . --force`: exit 0, `rtk v0.50.0` replaced in `~/.cargo/bin/rtk`. Log: `$TMPDIR/rtk-install2.log` (0 errors).
- No Rust source changed in this run, so `cargo fmt`/`clippy`/`test` were not re-run. The only change is `Cargo.lock`.

## Raw Scanner Output
### OSV-Scanner (`osv-scanner scan -L Cargo.lock`), pre-fix
```
Total 4 packages affected by 8 known vulnerabilities (0 Critical, 0 High, 1 Medium, 4 Low, 3 Unknown) from 1 ecosystem.
7 vulnerabilities can be fixed.
RUSTSEC-2026-0190  anyhow 1.0.102 → 1.0.103
RUSTSEC-2026-0002  lru 0.12.5 → 0.16.3     (CVSS 2.7)
GHSA-rhfx-m35p-ff5j lru 0.12.5 → 0.16.3
RUSTSEC-2026-0253  lru 0.12.5 → 0.18.2
RUSTSEC-2024-0436  paste 1.0.15 → (no fix)
RUSTSEC-2026-0346  rustls 0.23.45 → 0.23.46 (CVSS 4.0)
RUSTSEC-2026-0347  rustls 0.23.45 → 0.23.46 (CVSS 2.3)
RUSTSEC-2026-0348  rustls 0.23.45 → 0.23.46 (CVSS 3.7)
RUSTSEC-2026-0349  rustls 0.23.45 → 0.23.46 (CVSS 3.7)
```
### OSV-Scanner, post-fix
```
Total 2 packages affected by 3 known vulnerabilities (0 Critical, 0 High, 0 Medium, 1 Low, 2 Unknown) from 1 ecosystem.
RUSTSEC-2026-0002 / GHSA-rhfx-m35p-ff5j (lru), RUSTSEC-2026-0253 (lru), RUSTSEC-2024-0436 (paste)
```
### Trivy (`trivy fs --scanners vuln,secret`)
```
Cargo.lock   cargo  Vulnerabilities: 1 (LOW: 1)   Secrets: -
proxy/Cargo.lock  cargo  0
pom.xml fixtures  0
GHSA-rhfx-m35p-ff5j  lru  LOW  fixed 0.16.3  (installed 0.12.5)
```
### Gitleaks (`gitleaks detect --source . --no-banner --redact`)
```
2079 commits scanned. scanned ~18.5 MB in 1s. leaks found: 38
```
### Semgrep (`--config p/rust --config p/secrets`)
Blocking heuristic hits: `rust.lang.security.unsafe-usage` (libc signal/kill/raise sites), `rust.lang.security.args-os.args-os`, `rust.lang.security.current-exe.current-exe`. Full output was truncated in the run. Triaged above as intentional CLI patterns.
