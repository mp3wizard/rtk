# Security Audit — 2026-10-31

## Summary
- Issues found: 7 (grouped advisories/findings) | Auto-fixed: 4 | Unresolved: 3
- Status: ISSUES FIXED

Scope: full codebase after merging 409 upstream commits from `origin/develop`. Tools run: gitleaks, bandit, trivy, osv-scanner, trufflehog. Semgrep, skill/MCP audit scripts, and skillspector were not run in this pass (see Coverage).

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (Cargo.lock, proxy/Cargo.lock) | GHSA-c8m5-c952-6cm4 (MEDIUM), GHSA-h2v8-394m-p7vv, GHSA-vp88-798v-h67f, GHSA-w44r-93j7-qv28, RUSTSEC-2026-0346/0347/0348/0349 | 0.23.45 → 0.23.46 (`cargo update -p rustls --precise`) |
| 2 | rustls-webpki (Cargo.lock) | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 |
| 3 | webpki-roots (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0381 | 1.0.6 → 1.0.10 (root); 1.0.9 → 1.0.10 (proxy) |
| 4 | tokio (proxy/Cargo.lock) | RUSTSEC-2026-0386 | 1.53.1 → 1.53.3 |

## Unresolved Issues
| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| U1 | lru 0.12.5 (Cargo.lock) | RUSTSEC-2026-0002, RUSTSEC-2026-0253, GHSA-rhfx-m35p-ff5j (LOW) | Pulled by `ratatui 0.29.0`, which pins `lru ^0.12`. Fixed versions (0.16.3 / 0.18.2) need a `ratatui` upgrade (0.30.x available), a major-API change. Not auto-applied. |
| U2 | paste 1.0.15 | RUSTSEC-2024-0436 (unmaintained) | No fixed version exists. Informational. |
| U3 | hooks/hermes/rtk-rewrite/__init__.py:51-53 | Bandit B603/B607 (High, high-confidence) | `subprocess.run(["rtk", "rewrite", ...], shell=False)`. Args are a list, so no shell injection. The bare `rtk` name resolves through PATH. Pinning an absolute path is a design change to the hook, so not auto-applied. |

## Triage — no code change
- **gitleaks (39 hits):** `generic-api-key` / `stripe-access-token` in `src/cmds/cloud/aws_cmd.rs`, `tests/guard_integration_test.rs`, `proxy/tests/functional.rs`, `scripts/benchmark/cloud-init.yaml`, `SECURITY.md`, and prior `Security reports/*.md` (which quote earlier scanner output). Classed as false positives (test fixtures and quoted report text). Not suppressed in config.
- **trufflehog:** 0 verified secrets, 3 unverified (JDBC `jdbc:oracle:thin:@localhost:1523:XE` in `src/filters/liquibase.toml` and a report file). Local dev connection string, not a credential.
- **bandit:** 6 High findings, all B404/B603/B607 in `hooks/hermes/` (see U3). Tests covered by the same file.

## Raw Scanner Output
Full unedited logs are in the session scratchpad, not inlined here:
- `scratchpad/scan/gitleaks.txt`, `gitleaks.sarif`
- `scratchpad/scan/bandit.txt`
- `scratchpad/scan/trivy.sarif` (9 results), `trivy.txt`
- `scratchpad/scan/osv.txt`
- `scratchpad/scan/trufflehog.txt`

## Coverage
- Not run this pass: Semgrep (OWASP/Python/TypeScript/secrets), CodeQL, mcps-audit, config-audit, skill-audit, mcp-exfil-scan, skillspector (opt-in LLM mode not used).
- No Python/Node SAST beyond Bandit.
- Business logic / IDOR / runtime behaviour not covered; the `deep-audit` skill is the follow-up for those.
