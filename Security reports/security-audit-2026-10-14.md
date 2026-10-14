# Security Audit — 2026-10-14

## Summary
- Issues found: 3 advisories (2 crates, both via `ratatui 0.29`) | Auto-fixed: 0 | Unresolved: 3
- Status: ISSUES REMAINING
- Scope: worktree `claude/unruffled-hamilton-c55464`, HEAD `9e9dfbcb` (after upstream merge of `origin/develop`, 169 new upstream commits). Raw tool output kept at `/private/tmp/claude-501/rtk-scan/`.
- Secrets: 0 verified (TruffleHog), 0 live. All gitleaks hits are false positives (see Triaged Findings).
- Code SAST: 0 findings (Semgrep OWASP top-ten + secrets, Rust/TOML).

## Fixed Issues
None. Both dependency advisories sit behind a semver-major bump of `ratatui` (0.29 → 0.30) that changes a TUI code path (`src/analytics/gain_tui.rs`). Not applied in an unattended run; see Unresolved.

## Unresolved Issues

| # | Component | Advisory | Installed → Fixed | Reason |
|---|-----------|----------|-------------------|--------|
| 1 | `lru` (transitive via `ratatui` 0.29.0) | RUSTSEC-2026-0002 / GHSA-rhfx-m35p-ff5j (CVSS 2.7, LOW) — `IterMut` violates Stacked Borrows | 0.12.5 → 0.16.3 | `ratatui 0.29` requires `lru = "^0.12"`, so `cargo update -p lru --precise` cannot apply. Needs `ratatui` 0.30.x (MSRV 1.88, repo MSRV 1.91 OK) which may change the `gain_tui.rs` API. Not built/tested here. |
| 2 | `lru` (same path) | RUSTSEC-2026-0253 | 0.12.5 → 0.18.2 | Same cause as #1. |
| 3 | `paste` (transitive via `ratatui` 0.29.0) | RUSTSEC-2024-0436 (unmaintained) | 1.0.15 → no fix | Unmaintained advisory, no patched version. Clears when `ratatui` drops `paste`. |

**Recommended follow-up:** bump `ratatui` to 0.30.x in `Cargo.toml`, run `cargo build`, `cargo test --all`, and the `gain_tui` smoke path. That clears #1–#3 together (verify `paste` is gone from `cargo tree`).

## Triaged Findings (no fix required)

| # | Component | Tool | Finding | Disposition |
|---|-----------|------|---------|-------------|
| 1 | `src/cmds/cloud/aws_cmd.rs` (≈1865, 1866, 1910, 2024) | Gitleaks `generic-api-key` | Test JSON with fake ARNs (`123`, `role-1`, `fn-1`) and placeholder keys | False positive — test fixture |
| 2 | `tests/guard_integration_test.rs` (166–190) | Gitleaks `stripe-access-token` | Test fixture string `STRIPE_KEY=sk_live_…` used as git-log content | False positive — fixture, not a live key |
| 3 | `SECURITY.md` (137) | Gitleaks `stripe-access-token` | Example pattern in docs | False positive — documentation |
| 4 | `scripts/benchmark/cloud-init.yaml` (282) | Gitleaks `generic-api-key` | `postgres://user:pass@localhost:5432/...` placeholder | False positive — local placeholder |
| 5 | `Security reports/*.md` (prior audits, 2026-04 → 2026-07) | Gitleaks | Prior reports quote example secrets | False positive — report text |
| 6 | Gitleaks (history) | 38 raw leaks, 36 in working tree | Same classes as rows 1–5 | All triaged as false positive |
| 7 | Skill audit: `.claude/skills/security-guardian/SKILL.md` | skill-audit `CRITICAL` (`rm -rf` string) | Security training skill quotes attack payloads as examples | False positive — documentation |
| 8 | Skill audit: `.claude/skills/ship/SKILL.md` | skill-audit `CRITICAL` (score 90/100) | Score from Bash+Write+Edit tool access and a credential-pattern hit; 0 dangerous patterns, 0 injection patterns | Capability scoring, not a vulnerability |
| 9 | Skill audit: `.claude/skills/performance/SKILL.md` | skill-audit `HIGH` | Tool-breadth scoring | Not a vulnerability |
| 10 | Config audit | config-audit `CRITICAL` ×24 | All 20 shown are in the user's plugin cache (`~/.claude/plugins/cache/...`: ponytail, caveman, security-scanner scripts). Matches are `eval`/`base64`/`curl` in tool code. | Outside repo scope; heuristic. Not in this repo. |
| 11 | skillspector | Score 100/100 on flagged locations | Ran `--no-llm`. Most static analyzers reported `degraded` (no LLM). Flagged locations are mostly test fixtures, docs and `src/` string literals. | Low confidence. Coverage is partial. |

## Raw Scanner Output

Full raw output is saved to `/private/tmp/claude-501/rtk-scan/` (too large to inline; not edited):
- `semgrep-owasp.txt` — 7 rules, 338 files, **0 findings**
- `semgrep-secrets.txt` — 45 rules, 597 files, **0 findings**
- `gitleaks.json` / `gitleaks-tree.json` — 38 / 36 hits, triaged above
- `trufflehog-verified.txt` — **0 verified, 0 unverified**
- `config-audit.txt`, `skill-audit.txt`, `mcp-exfil.txt` (**CLEAN, risk 0/100**), `skillspector.txt`

Excerpts:

Trivy (`trivy fs --scanners vuln,secret,misconfig`):
```
Cargo.lock (cargo)
Total: 1 (UNKNOWN: 0, LOW: 1, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
│ lru │ GHSA-rhfx-m35p-ff5j │ LOW │ fixed │ 0.12.5 │ 0.16.3 │ `IterMut` violates Stacked Borrows ...
```

OSV-Scanner (`osv-scanner scan source -r`):
```
Total 2 packages affected by 3 known vulnerabilities (0 Critical, 0 High, 0 Medium, 1 Low, 2 Unknown) from 1 ecosystem.
| RUSTSEC-2026-0002 | 2.7 | lru   | 0.12.5 | 0.16.3 |
| RUSTSEC-2026-0253 |     | lru   | 0.12.5 | 0.18.2 |
| RUSTSEC-2024-0436 |     | paste | 1.0.15 | --     |
```

Dependency path (`cargo tree -i lru@0.12.5`):
```
lru v0.12.5 └── ratatui v0.29.0 └── rtk v0.50.0
```

## Tool Coverage
| Tool | Ran | Notes |
|------|-----|-------|
| Gitleaks | Yes | 38 raw, all false positive |
| TruffleHog | Yes | 0 verified, 0 unverified |
| Semgrep | Yes | p/owasp-top-ten (*.rs, *.toml) + p/secrets: 0 findings. First run failed: the `$SG` variable did not word-split under zsh. Re-run with an array and the pipx binary. |
| Trivy | Yes | 0.75.0. Not in the compromised list. |
| OSV-Scanner | Yes | 2.6.0 |
| Bandit | Skipped | No `.py` files in scope (Rust repo) |
| CodeQL | Skipped | Not run in this session |
| mcps-audit | Skipped | No MCP config files in repo |
| mcp-scan | Skipped | Opt-in, needs user consent (sends data to invariantlabs.ai) |
| Jev triage | Skipped | Opt-in, off by default |
| config-audit / skill-audit / mcp-exfil | Yes | See triage |
| skillspector | Yes, `--no-llm` | Local only. LLM mode not used (opt-in). |
| APTS audit log | Partial | Log at `/tmp/css-scan-20261014T021505Z.jsonl`. `finalize` errored at `apts-audit.sh:66` (arithmetic on a multi-line count). Tools ran via direct invocation, not the `run` wrapper, so records are `measured:false`. |

## Build Failure
None recorded at this stage. Build is in STEP 5.
