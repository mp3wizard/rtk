# Security Audit — 2026-10-22

## Summary
- Issues found: 4 | Auto-fixed: 1 | Unresolved: 3 (2 CVEs, 1 scanner-low group)
- Status: ISSUES REMAINING

Scope: upstream `origin/develop` merged (299 new upstream commits). Full scan run on the worktree: Rust sources, `Cargo.toml`, `Cargo.lock`, `proxy/Cargo.lock`, Python/TS/Ruby files (git-tracked), skill/MCP/config artifacts.

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls | RUSTSEC-2026-0346, -0347, -0348, -0349 (CVSS 4.0 / 2.3 / 3.7 / 3.7) | `cargo update -p rustls --precise 0.23.46`: `0.23.45` → `0.23.46` in root `Cargo.lock` and `proxy/Cargo.lock` (transitive via `ureq` 2.x) |
| 2 | rustls-webpki | RUSTSEC-2026-0382 | `cargo update -p rustls-webpki --precise 0.103.16`: `0.103.15` → `0.103.16` in both lockfiles |
| 3 | webpki-roots | RUSTSEC-2026-0381 | root `Cargo.lock`: `1.0.6` → `1.0.10` (transitive via `webpki-roots` 0.26 / `ureq`); `proxy/Cargo.lock`: `1.0.9` → `1.0.10` |

Lockfile-only changes; `Cargo.toml` unchanged.

## Unresolved Issues
- **lru 0.12.5** — RUSTSEC-2026-0002 (GHSA-rhfx-m35p-ff5j, CVSS 2.7) and RUSTSEC-2026-0253. Fix is `0.16.3` / `0.18.2`, but `ratatui` 0.29.0 pins `lru ^0.12`, so `cargo update --precise` fails. Needs a `ratatui` major bump (TUI code change). Not attempted in this automated run.
- **paste 1.0.15** — RUSTSEC-2024-0436 (unmaintained, no patched version). No safe fix; needs replacement of the proc-macro dependency chain or accept the advisory.
- **Bandit B404/B603/B607 (Low, 9 hits)** — `scripts/benchmark-sessions/lib/runner.py:5,28`: `subprocess.run(["tar", ...])` with list args, no `shell=True`. Accepted as low risk, not changed; partial-path `tar` is a deliberate PATH dependency.

## Scanner Results (summary)
| Tool | Result |
|------|--------|
| Gitleaks (history + tree) | 38 hits, all false positives: `FAKE`/`1234567890` placeholders in `tests/guard_integration_test.rs`, `scripts/benchmark/cloud-init.yaml`, `src/cmds/cloud/aws_cmd.rs`, `SECURITY.md`, and prior `Security reports/*` quoting them. No real credentials. |
| TruffleHog (git) | 0 verified. 3 unverified (JDBC `localhost` URL in `src/filters/liquibase.toml:73` and two others). Run with `--no-verification` (see Notes). |
| Bandit | 9 Low, 0 Medium/High (see Unresolved). |
| Semgrep OWASP (266 rules, 12 git-tracked files in scanned languages) | 0 findings |
| Semgrep secrets (599 files) | 0 findings; 1 file over the 0.3 MB cap skipped |
| Trivy fs | 21 `stripe-secret-token` "errors" all in `gitleaks.sarif` (the scanner's own output file, which quotes the fixture strings); 1 note = lru GHSA above. |
| OSV-Scanner (root + proxy lockfiles) | 15 vulns / 8 pkgs before fixes; rustls/webpki/webpki-roots fixed (above); lru + paste remain. |
| config-audit | LOW only (hooks configuration notices in plugin files). Output truncated in this run; re-run for full detail. |
| skill-audit | Ran per SKILL.md. "Credential access: 1" reported in two skill files. Full text saved to the scanner reports dir; **not triaged** in this run. |
| mcp-exfil-scan | RISK 0/100, VERDICT CLEAN |
| skillspector (--no-llm) | Completed (run finished later, exit 0). 840 results: 478 error, 319 warning, 43 note. Mostly heuristic pattern matches on docs, `Security reports/*`, test files, and `src/discover/*` (rule text). Error-level categories: SC9 (3 hook `.sh` files flagged as concealed executable content; checked, plain UTF-8 bash, no hidden chars, judged false positive), PE3 credential-access and TM1 tool-parameter-abuse matches on command/skill docs (keyword matches, not reviewed line by line), AS1 agent-config-dir mentions in docs. **No real triage done yet**; treat as open coverage. |

## Raw Scanner Output
Full logs: APTS audit log `/tmp/css-scan-20261022T021423Z.jsonl`; skill/MCP reports under the security-scanner plugin `reports/` dir (`test-20261022_*.txt`, `mcp-exfil-20261022_091519.txt`). Condensed here; not fully verbatim.

Gitleaks: `leaks found: 38`, rules `generic-api-key` (17), `stripe-access-token` (21).
OSV: `Total 8 packages affected by 15 known vulnerabilities (0 Critical, 0 High, 2 Medium, 7 Low, 6 Unknown)`; `14 vulnerabilities can be fixed`.
Bandit: `Total issues: Low 9, Medium 0, High 0`; `Total lines of code: 467`.
Semgrep: `Ran 266 rules on 12 files: 0 findings.` / `Ran 45 rules on 599 files: 0 findings.`
mcp-exfil: `RISK SCORE: 0/100`, `VERDICT: CLEAN`.

## Notes / Deviations
- Scanner ran from the worktree (`claude/xenodochial-saha-0a20c0`), not the main checkout path in the task file.
- TruffleHog ran with `--no-verification` so discovered strings were not sent to vendor APIs.
- Semgrep wrapper: `$SG` string-expansion failed under zsh in the first attempt (logged as "command not found"); re-run with array args, results above are from the re-run.
- skillspector first reported as hung. It was still running and finished later. Earlier report wording was wrong and is corrected here.
- Build: `cargo install --path . --force` succeeded (exit 0) after the lockfile bumps; `rtk` 0.50.0 installed to `~/.cargo/bin/rtk`; `~/.local/bin/rtk` wrapper rewritten. Binary not executed (sandbox rule); verified by file metadata only.
- Not run this pass: `cargo fmt`/`clippy`/`cargo test` (not part of the sync steps).
- Commit and push NOT performed: the user's global rule requires confirmation before `git commit` and `git push`, and no one is available to confirm during this automated run.
