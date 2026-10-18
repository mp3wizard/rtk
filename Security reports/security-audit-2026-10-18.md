# Security Audit — 2026-10-18

## Summary
- Issues found: 0 confirmed code vulnerabilities | 3 scanner groups unverified (see Unresolved) | Auto-fixed: 0 | Unresolved: 2
- Status: ISSUES REMAINING

Scope: upstream `origin/develop` merged cleanly (233 new upstream commits, 0 conflicts). Branch `claude/angry-dewdney-6d0b3c`, worktree `gallant-blackwell-401a7d`, HEAD `5655d385` after merge.

## Fixed Issues
None. No confirmed vulnerability required a code or dependency change.

## Unresolved Issues

| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| 1 | Dependency SCA (Cargo.lock, 255 packages) | none checked | OSV-Scanner could not reach `api.osv.dev` (DNS lookup failed), so it reported "0 packages affected" with no real database query. Trivy could not download its vulnerability DB (`mirror.gcr.io` unreachable). **No advisory check of Cargo.lock was performed.** Re-run `osv-scanner scan source -r .` and `trivy fs --scanners vuln .` with network access. |
| 2 | Gitleaks / TruffleHog hits (38 + 3) | n/a | Same count and file classes as the 2026-10-01 triage (prior report text, test fixtures, sample configs). Triage was not repeated line by line this run, and TruffleHog ran with `--no-verification`, so no live check was done. Confirm the Stripe-pattern hits in `tests/guard_integration_test.rs` and `SECURITY.md` are fake values. Git history was not rewritten. |

## Triaged Findings (no fix required, pending #2 confirmation)

| Tool | Finding | Disposition |
|------|---------|-------------|
| Gitleaks | 38 hits, rules `stripe-access-token` (21) and `generic-api-key` (17), in 18 historical commits | 22 in `Security reports/*.md` (quoted prior findings). Others: `tests/guard_integration_test.rs` (2 Stripe-pattern), `scripts/benchmark/cloud-init.yaml` (2 `API_KEY=` placeholders), `src/cmds/cloud/aws_cmd.rs` (5 `nextForwardToken`/`API_KEY` JSON keys), `SECURITY.md` (1). Treated as false positive, same as 2026-10-01. |
| TruffleHog | 3 `JDBC` hits: `Security reports/security-audit-2026-10-11.md:26`, `src/filters/liquibase.toml:73,78` | Documentation text and filter sample strings. Unverified (no live check). |
| Semgrep 1.180.0 (`p/secrets`, `p/rust`) | 33 results, all INFO: `unsafe-usage` (15), `current-exe` (9), `temp-dir` (6), `args` (2), `args-os` (1) | Audit-style hints, not vulnerabilities. No `p/owasp-top-ten` run this pass. |
| Bandit (4 `.py` files: `scripts/benchmark-sessions/lib/runner.py`, `hooks/hermes/**`) | Only Low severity: B404, B603, B607 (subprocess use) | Expected for test/hook scripts. No Medium or High issues. |

## Not Run (tool gaps)
- Trivy: DB download blocked by network. No container or IaC scan.
- CodeQL: not triggered (no `gh run` query this pass).
- cargo-audit / cargo-deny: not installed.
- mcp-scan, skillspector LLM mode, Jev triage: opt-in, not run (no user consent this scheduled run).
- config-audit / skill-audit / mcp-exfil-scan: not run, scope kept to the repo.

## Raw Scanner Output (verbatim summaries; full logs in session scratchpad)
- OSV-Scanner: `Total 0 packages affected by 0 known vulnerabilities (0 Critical, 0 High, 0 Medium, 0 Low, 0 Unknown) from 0 ecosystems.` Preceded by `Error during extraction: ... dial tcp: lookup api.osv.dev: no such host`. **Invalid result, see Unresolved #1.**
- Trivy: `FATAL ... failed to download vulnerability DB ... lookup mirror.gcr.io: no such host`. Exit 1, no result.
- Gitleaks: `2078 commits scanned.` `scanned ~17948773 bytes (17.95 MB)`. `leaks found: 38`.
- Semgrep: `results 33`, `errors 0`.
- TruffleHog: `findings 3`, `verified 0` (run with `--no-verification`).
- Bandit: Low-only findings, 0 Medium/High.

## Build
- `cargo install --path . --force`: succeeded, `rtk v0.50.0` (Cargo.toml version) installed to `~/.cargo/bin/rtk` (12M).
- `~/.local/bin/rtk` wrapper rewritten to exec `~/.cargo/bin/rtk`.
- Note: this replaced a binary previously installed from another worktree (`amazing-nobel-706e8f`).
- Binary not executed (sandbox SIGKILL risk per run rules); file metadata only.
