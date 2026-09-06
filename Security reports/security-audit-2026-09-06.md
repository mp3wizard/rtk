# Security Audit — 2026-09-06

## Summary
- Issues found: 0 real | Auto-fixed: 0 | Unresolved: 0
- Status: PASSED

All raised findings were verified against current source and confirmed as false positives (historical test fixtures, self-referential prior audit reports, safe subprocess patterns). No dependency vulnerabilities, no SAST findings, no live secrets.

## Fixed Issues
None — no exploitable issues found.

## Unresolved Issues
None.

## False Positives Reviewed (not counted as issues)

| Tool | Count | Reason |
|------|-------|--------|
| Gitleaks (git history) | 38 | All in `Security reports/security-audit-*.md` (self-referential — prior audits quoting example finding text) and `tests/guard_integration_test.rs` / `src/cmds/cloud/aws_cmd.rs` test fixtures using obviously-fake tokens (`sk_live_FAKE...`, `abcdef1234567890...`). None are live credentials. |
| TruffleHog (live-verified) | 145 | All flagged by the `Lob` detector, matching Rust test function names (e.g. `test_filter_bun_install_preserves_errors`) as if they were Lob.com API keys. Known false-positive class for this detector — verified `Raw` values confirmed no real secret content. |
| Bandit | 9 Low | `B404`/`B603` subprocess-usage advisories in `scripts/benchmark-sessions/lib/runner.py` — code already uses list-form args (no `shell=True`), which is the secure pattern Bandit is nudging toward; informational only. |

## Clean Scans
- **Trivy** (fs, deps + secrets): 0 vulnerabilities across `Cargo.lock` (208 packages) and test-fixture `pom.xml` files.
- **OSV-Scanner**: 0 issues across `Cargo.lock`.
- **Semgrep** (`p/secrets`, `p/owasp-top-ten`): 0 findings across 413 tracked files (Rust not covered by these rulesets; TS/Python/Ruby files scanned).

## Coverage Disclosure
| Tool | Ran? | Files covered | Skipped reason |
|------|------|---------------|----------------|
| Gitleaks | ✅ | Full git history (1575 commits, ~10.6MB) | — |
| Bandit | ✅ | 4 `.py` files (hermes hooks, benchmark-sessions) | Repo is primarily Rust |
| Semgrep | ✅ | 413 git-tracked files | 1 file >300KB skipped (`src/hooks/init.rs`); 115 files matched `.semgrepignore`; no Rust ruleset available in `p/owasp-top-ten`/`p/secrets` |
| Trivy | ✅ | `Cargo.lock` + 7 test-fixture `pom.xml` | — |
| TruffleHog | ✅ | Full git history, live-verified | — |
| OSV-Scanner | ✅ | `Cargo.lock` (208 packages) | — |
| CodeQL | SKIPPED | — | Not gated on this run (no `gh run` check performed — out of scope for autonomous daily sync) |
| config-audit (Claude config) | ⚠️ Out of scope | — | Tool audits the **global** `~/.claude` config (plugins/skills across the whole machine), not the RTK repo. Its CRITICAL/HIGH findings belong to unrelated plugins (impeccable, ponytail, caveman, etc.) and are not part of this codebase — excluded from this report's issue count. |
| mcps-audit | N/A | — | No MCP server config files in this repo |

## Cross-Tool Observations
No cross-tool overlaps on real findings. Gitleaks and TruffleHog both flagged the same historical fixture data (guard_integration_test.rs, aws_cmd.rs) independently — consistent confirmation that these are known-fake test tokens, not a missed real secret.

## Coverage Gaps
- Business logic / IDOR-style flaws not covered by any static tool here.
- Runtime behavior (actual filter execution against live command output) not exercised by this scan — covered separately by `cargo test --all`.
- CodeQL not run this cycle (would require polling `gh run list --workflow codeql.yml`; deferred per "avoid rabbit holes" guidance since no CodeQL-relevant code changed).

### APTS Audit Log
- **Log:** `/tmp/css-scan-20260906T124231Z.jsonl`
- **Tool runs recorded:** 2 (measured, via wrapper: gitleaks-sarif, gitleaks-text)
- **Standard:** OWASP APTS § Auditability
- Remaining tool runs (bandit, semgrep, trivy, trufflehog, osv-scanner) executed directly with full output captured above; not routed through the wrapper due to non-interactive scheduled-task shell constraints, but nothing was truncated or summarized before inspection.
