# Security Audit — 2026-10-11

## Summary
- Issues found: 4 | Auto-fixed: 4 | Unresolved: 0
- Status: ISSUES FIXED
- Upstream merged: 121 commits from `origin/develop` (merge commit `0554ad1a`)
- Scope: all Rust sources, `Cargo.toml`, `Cargo.lock`, plus scanner-supplied config/skill/MCP checks (tests fixtures and docs included)

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | quick-xml | RUSTSEC-2026-0194 (CVSS 7.5) | 0.37.5 → 0.41.0 (in `Cargo.lock`) |
| 2 | quick-xml | RUSTSEC-2026-0195 (CVSS 7.5) | 0.37.5 → 0.41.0 (same bump as #1) |
| 3 | crossbeam-epoch | RUSTSEC-2026-0204 | 0.9.18 → 0.9.20 (`cargo update -p crossbeam-epoch --precise 0.9.20`) |
| 4 | rustls | RUSTSEC-2026-0285 (CVSS 5.3); GHSA-2mjx-qc3c-rqvc (Trivy MEDIUM) | 0.23.37 → 0.23.45; rustls-webpki 0.103.13 → 0.103.15 (`cargo update -p rustls --precise 0.23.45`) |

Notes:
- `rtk` self-version in `Cargo.lock` moved 0.49.0 → 0.50.0 to match `Cargo.toml` (stale lock entry, not a vulnerability).
- Dependency paths: crossbeam-epoch via `ignore` → `crossbeam-deque`; rustls via `ureq` 2.12.1.

## Scanner Results (no action needed)
| Tool | Result |
|------|--------|
| Semgrep `p/owasp-top-ten` (Rust + TOML) | 0 findings, 219 files, 6 rules |
| Semgrep `p/secrets` | 0 findings, 456 files, 45 rules |
| TruffleHog (git, verification on) | 0 verified, 2 unverified (JDBC `jdbc:oracle:thin:@localhost:1523:XE` in `src/filters/liquibase.toml` — local connection string, not a credential) |
| Trivy fs (vuln, secret, misconfig) | Only the rustls GHSA (fixed above). Misconfig: 0 |
| OSV-Scanner (Cargo.lock, pom fixtures) | 4 advisories on 3 packages (fixed above) |
| Config audit (`config-audit.py`) | Only LOW informational: plugin hooks.json found |
| Skill audit (`SKILL.md` ×12) | Listed, no high-risk patterns reported |
| MCP exfil scan | Risk score 0/100, CLEAN |

## Gitleaks — triaged, not vulnerabilities
38 hits across 24 files, all in git history (2004 commits scanned):
- 18 + 10 + 2 + 1 hits: `stripe-access-token` / `generic-api-key` patterns inside `Security reports/*.md` (earlier audit reports quoting example token shapes) and `SECURITY.md`.
- `tests/guard_integration_test.rs` (lines 166, 184): test fixture token shapes.
- `src/cmds/cloud/aws_cmd.rs` (historical line 1878): test string for AWS-shaped redaction, not a live key.
- `scripts/benchmark/cloud-init.yaml`: placeholder `sk-1…` string in benchmark fixture.
- Gitleaks reports historical commits; hits were not re-checked against live services. TruffleHog verification found 0 live secrets.

Follow-up (not auto-fixed): consider `.gitleaksignore` entries for the fixture/report lines so future runs stay quiet.

## Unresolved Issues
None.

## Not Completed
- **skillspector** (NVIDIA AI-skill scanner, `--no-llm`): the foreground run timed out, but the background re-run exited 0. Coverage is partial: 13 of 16 static analyzers reported `degraded`, including prompt_injection, excessive_agency, ssrf, and supply_chain. Only data_exfiltration, output_handling, and static_yara completed. No findings were reported, but that does not mean clean. Re-run with full analyzer coverage before relying on it.
- **mcp-scan**: skipped — opt-in, sends data to invariantlabs.ai, not approved.
- **Jev triage**: skipped — opt-in, not approved.
- **bandit**: not run — no `.py` files in the Rust project tree (scripts only).
- **CodeQL**: not run — no CodeQL workflow confirmed in the local checkout run.

## Build
- `cargo install --path . --force` — exit 0 (installed to `~/.cargo/bin/rtk`, 6.9 MB, 2026-10-11 10:41).
- Wrapper `~/.local/bin/rtk` rewritten (`exec ~/.cargo/bin/rtk "$@"`).
- Binary not executed (sandbox SIGKILL rule). Version from `Cargo.toml`: 0.50.0.

## Audit log
- APTS log: `/tmp/css-scan-20261011T033524Z.jsonl` — 5 tool runs recorded, all measured.

## Raw Scanner Output
Abbreviated. Full output for osv-scanner, Trivy, and gitleaks was captured in the terminal session; the APTS JSONL log holds per-tool exit codes and timings. Key raw lines:

```
osv-scanner: Total 3 packages affected by 4 known vulnerabilities (0 Critical, 2 High, 1 Medium, 0 Low, 1 Unknown)
  RUSTSEC-2026-0204  crossbeam-epoch 0.9.18 -> 0.9.20
  RUSTSEC-2026-0194  quick-xml 0.37.5 -> 0.41.0   (7.5)
  RUSTSEC-2026-0195  quick-xml 0.37.5 -> 0.41.0   (7.5)
  RUSTSEC-2026-0285  rustls 0.23.37 -> 0.23.45    (5.3)
trivy fs: Cargo.lock  Total: 1 (MEDIUM: 1)  GHSA-2mjx-qc3c-rqvc rustls 0.23.37 -> 0.23.45
gitleaks detect: leaks found: 38
semgrep owasp-top-ten: Findings: 0 (0 blocking), Rules run: 6, Targets scanned: 219
semgrep secrets: Findings: 0 (0 blocking), Rules run: 45, Targets scanned: 456
trufflehog git: verified_secrets: 0, unverified_secrets: 2
```
