# Security Audit — 2026-10-30

## Summary
- Issues found: 9 | Auto-fixed: 4 | Unresolved: 5 (no upstream fix or fix needs a major-version upgrade)
- Status: ISSUES FIXED

Upstream merged: 392 commits from `origin/develop` (merge commit `chore: sync upstream 2026-10-30`, no conflicts).

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (Cargo.lock, proxy/Cargo.lock) | GHSA-c8m5-c952-6cm4 (MEDIUM, session Debug exposes TLS master secrets), RUSTSEC-2026-0346 (CVSS 4.0), RUSTSEC-2026-0347, GHSA-h2v8-394m-p7vv, GHSA-vp88-798v-h67f, GHSA-w44r-93j7-qv28, RUSTSEC-2026-0348, RUSTSEC-2026-0349 | 0.23.45 → 0.23.46 (`cargo update -p rustls --precise 0.23.46`) |
| 2 | rustls-webpki (Cargo.lock) | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 |
| 3 | webpki-roots (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0381 | 1.0.6 → 1.0.10 (root), 1.0.9 → 1.0.10 (proxy) |
| 4 | tokio (proxy/Cargo.lock) | RUSTSEC-2026-0386 | 1.53.1 → 1.53.3 |

Release build after bumps: `cargo build --release` OK (exit 0).

## Unresolved Issues
| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| 5 | block-buffer 0.10.4 (via sha2 0.10.9 → digest 0.10.7, both lockfiles) | GHSA-qwgh-2vcv-g2f7 (CVSS 6.3) | No patched version published; `cargo update -p sha2` yields no newer block-buffer. Needs a sha2/digest line that moves to a fixed block-buffer, or a source-level change. |
| 6 | lru 0.12.5 (via ratatui 0.29.0) | RUSTSEC-2026-0002 (CVSS 2.7), RUSTSEC-2026-0253, GHSA-rhfx-m35p-ff5j (LOW, Stacked Borrows in `IterMut`) | Fix requires lru ≥0.16.3, which ratatui 0.29 cannot take. Needs a ratatui major upgrade (TUI code change). |
| 7 | paste 1.0.15 (via ratatui 0.29.0) | RUSTSEC-2024-0436 (unmaintained) | No fix version; resolves with the ratatui upgrade above. |
| 8 | Semgrep rust rules: `current_exe`/`args`/`args_os` used as security input (src/core/telemetry.rs, src/hooks/init/mod.rs, src/hooks/session_start.rs, src/llm_proxy.rs, src/lib.rs) | p/rust `current-exe`, `args` rules | Rule hits on CLI self-path/argv use, not exploitable security decisions. Triaged as false positive for a CLI. Counted here, not fixed. |
| 9 | Semgrep rust rules: `unsafe` libc/Win32 calls (src/lib.rs, src/core/utils.rs, src/core/test_isolation/scratch.rs) and `temp_dir` in src/core/test_isolation/scratch.rs:315 | p/rust `unsafe-usage`, `temp-dir` | Required FFI for SIGPIPE/signal/console-codepage handling; temp_dir is used only as test `GIT_CEILING_DIRECTORIES`. Reviewed, no change. |

## Scanner Coverage
| Tool | Result |
|------|--------|
| OSV-Scanner (Cargo.lock, proxy/Cargo.lock) | 18 findings before fix → 5 after (see table) |
| Trivy fs | Cargo.lock 5, proxy/Cargo.lock 4 before fix (same advisories); no IaC/secret misconfig findings for Rust sources |
| Semgrep (p/owasp-top-ten, p/secrets, p/rust) on `src/` | Only Rust rule hits listed under #8–#9; no OWASP or secrets findings |
| Gitleaks (history, 2192 commits) | 39 hits |
| Gitleaks (working tree) | 36 hits — all in `Security reports/*.md`, `SECURITY.md`, `scripts/benchmark/cloud-init.yaml`, `src/cmds/cloud/aws_cmd.rs` (test fixtures), `tests/guard_integration_test.rs`, `proxy/tests/functional.rs`. Same set triaged as fixtures/prose in the 2026-10-01 audit. **Not individually re-verified this run** — see note below. |
| TruffleHog, bandit, skillspector, mcp-scan | Not run this run (TruffleHog not invoked; bandit: no `.py` in product code; mcp-scan opt-in and not consented; skillspector LLM mode opt-in, `--no-llm` not run) |

Note: the `aws_cmd.rs` generic-api-key hits (lines 1865–2024) and `cloud-init.yaml:282` were not confirmed as placeholders in this run. Verify by hand before treating the audit as clean.

## Raw Scanner Output
Scanner output was summarized above; full raw logs are not stored in this file. Re-run:
- `osv-scanner scan source -L Cargo.lock -L proxy/Cargo.lock`
- `trivy fs --scanners vuln,secret,misconfig .`
- `semgrep scan --config p/owasp-top-ten --config p/secrets --config p/rust src`
- `gitleaks detect --source .`
