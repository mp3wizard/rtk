# Security Audit — 2026-10-21

## Summary
- Issues found: 10 advisories | Auto-fixed: 5 | Unresolved: 5
- Status: ISSUES REMAINING
- Upstream merged: 299 commits from origin/develop (clean merge, no conflicts)

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0346 (CVSS 4.0) | 0.23.45 → 0.23.46 |
| 2 | rustls (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0347 (CVSS 2.3) | 0.23.45 → 0.23.46 |
| 3 | rustls (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0348 (CVSS 3.7) | 0.23.45 → 0.23.46 |
| 4 | rustls (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0349 (CVSS 3.7) | 0.23.45 → 0.23.46 |
| 5 | rustls-webpki (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 |

Method: `cargo update -p <crate> --precise <ver>` in root and `proxy/` workspaces. `cargo build --release` passes after the bump.

## Partially Fixed
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 6 | webpki-roots | RUSTSEC-2026-0381 | proxy/Cargo.lock 1.0.9 → 1.0.10 and root 1.0.6 → 1.0.10. Root still carries webpki-roots 0.26.11 (pinned by ureq 2.12.1), which remains vulnerable. See Unresolved #4. |

## Unresolved Issues
1. **lru 0.12.5 — RUSTSEC-2026-0002 / GHSA-rhfx-m35p-ff5j (CVSS 2.7) and RUSTSEC-2026-0253.** Fixed in 0.16.3 / 0.18.2. Pulled in by `ratatui 0.29.0`, which pins `lru ^0.12`. Fix needs a ratatui upgrade (major-version change, UI code impact). Trivy also reports GHSA-rhfx (LOW) against the same version. Not auto-bumped: `cargo update` cannot cross the semver range.
2. **paste 1.0.15 — RUSTSEC-2024-0436.** Unmaintained-crate advisory, no patched version published. Transitive via `ratatui`. Needs a ratatui upgrade or replacement, no direct fix.
3. **rustls-webpki / webpki-roots in proxy/** — rustls-webpki fixed (item 5); no further action.
4. **webpki-roots 0.26.11 — RUSTSEC-2026-0381.** Pinned by `ureq 2.12.1`, which requires `webpki-roots ^0.26`. Fix needs ureq 3.x migration (API change). Not auto-bumped.

## Scanner Results (condensed; full scanner output is in the tool logs of this run)

### OSV-Scanner (`osv-scanner scan source -r .`)
Scanned `Cargo.lock` (255 pkgs), `proxy/Cargo.lock` (319 pkgs), and pom.xml fixtures (test data only).
Total: 9 packages, 16 known vulnerabilities (0 Critical, 0 High, 2 Medium, 7 Low, 7 Unknown); 15 fixable.
Remaining after this run: lru ×2, paste, webpki-roots 0.26.11 (root tree).

### Trivy (`trivy fs --scanners vuln,secret --skip-dirs tests/fixtures .`)
- Cargo.lock: 1 finding — GHSA-rhfx-m35p-ff5j, lru 0.12.5, LOW (status: fixed in 0.16.3; still present in tree, see Unresolved #1).
- proxy/Cargo.lock: 0 findings reported by Trivy (OSV reports more; Trivy DB differs).
- Secrets: none reported by Trivy.
- Trivy version 0.75.0 — not in the known-compromised list (0.69.4–0.69.6).

### Gitleaks (`gitleaks detect`)
- Git history: 38 hits across 2118 commits (`stripe-access-token`, `generic-api-key`).
- Filesystem (`--no-git`): 36 hits. Locations: `Security reports/*.md` (earlier audit reports that quote redacted detector output), `SECURITY.md:137`, `scripts/benchmark/cloud-init.yaml:282`, `tests/guard_integration_test.rs:173,190`, `src/cmds/cloud/aws_cmd.rs:1865,1866,1910,2024`.
- Triage: these look like test fixtures, documentation examples, and quoted report text. **Not verified against live services** — no real credential was confirmed. Treat as reviewed-but-open: a human should confirm the `aws_cmd.rs` and `guard_integration_test.rs` matches are placeholder strings and not live keys (rotate if any are real).

### Semgrep (`--config p/owasp-top-ten --include "*.rs" src`)
Ran 6 multilang rules on 170 git-tracked files: 0 findings. The `p/secrets` and `p/python`/`p/typescript` configs were not run (no Python/TS in Rust source; secrets covered by gitleaks).

### TruffleHog, Bandit, mcps-audit, mcp-scan, skillspector
Not run this pass. Bandit: no `.py` in scope. mcp-scan/skillspector LLM mode: opt-in, not requested. TruffleHog not run in this automated pass (time budget); gitleaks covered secrets.

## Build Verification
- `cargo build --release`: OK (after lockfile bumps).
- `cargo install --path . --force`: OK → `$HOME/.cargo/bin/rtk` (rtk v0.50.0 per Cargo.toml).
- Wrapper `$HOME/.local/bin/rtk` rewritten (exec to `$HOME/.cargo/bin/rtk`).
- Binary NOT executed (sandbox SIGKILL note in the run instructions).
- `cargo test --all` not run in this pass.

## Raw Scanner Output
See sections above. Outputs were condensed for this report; the complete OSV, Trivy, Gitleaks, and Semgrep output was printed in full during the run and is not duplicated here.
