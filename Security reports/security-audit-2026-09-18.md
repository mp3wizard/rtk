# Security Audit — 2026-09-18

## Summary
- Issues found: 3 | Auto-fixed: 1 | Unresolved: 2
- Status: ISSUES FIXED

## Scope Record
- Scan target: `/Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/amazing-raman-3115e8`
- Git HEAD: `26878b597607898dfee14fe33a7ce558dfdfb7c3` (post-merge, pre-audit-commit)
- Include: full codebase — Rust sources, Cargo.toml, Cargo.lock, Python helper scripts, docs
- Exclude: `.git/`, `target/`, `.gitignore`-honored paths

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | `rustls` (transitive, via `ureq`) | RUSTSEC-2026-0285 (CVSS 5.3, Medium) | `cargo update -p rustls`: 0.23.37 → 0.23.45 (pulled `rustls-webpki` 0.103.13 → 0.103.15 as a dependent bump). Verified with `cargo build`, `cargo fmt --all`, `cargo clippy --all-targets` (clean), `cargo test --all` (3809 passed, 8 ignored). |

## Unresolved Issues
| # | Component | Reason |
|---|-----------|--------|
| 2 | Trivy filesystem scan | Vulnerability DB download failed twice (`mirror.gcr.io/aquasec/trivy-db:2`, connection reset by peer) — transient network issue in this sandbox, not a code finding. OSV-Scanner ran successfully instead and covers the same dependency-vulnerability surface (Cargo.lock), so coverage is not fully lost. Retry on next scheduled run. |
| 3 | `mcp-exfil-scan.sh` (bundled skill script) | Bundled-script integrity check (`SHA256SUMS`) **FAILED** for this script — skipped per skill safety rule rather than running unverified code. Not part of the Rust-codebase scope this task targets (it audits MCP tool configs, not source code); reinstall the `claude-code-security-plugins` skill from a trusted release to restore it. |

## Non-Issues (reviewed, no action needed)
- **Gitleaks**: 38 hits across git history, all in test fixtures and prior audit reports (`tests/guard_integration_test.rs`, `src/cmds/cloud/aws_cmd.rs` unit tests, `scripts/benchmark/cloud-init.yaml`, `SECURITY.md`, past `Security reports/*.md`). Every hit inspected traces to an explicit placeholder value (a `stripe-access-token`-shaped string with a literal `FAKE` marker in it) used to test rtk's own secret-redaction filters — not a real credential. Consistent with every prior audit report in this repo.
- **TruffleHog** (git history, live-verification mode): 0 verified secrets, 0 unverified secrets, 19701 chunks scanned. Confirms the gitleaks hits are not real/active credentials.
- **Semgrep** (`p/owasp-top-ten`, `p/secrets`, `p/python`): 0 findings across all three rulesets.
- **Bandit** (Python, `hooks/hermes/` + `scripts/benchmark-sessions/`): 9 Low-severity/High-confidence findings, all `B404`/`B603`/`B607` (subprocess module usage, `shell=False`). Every call site uses a hardcoded trusted argv (`rtk`, `cargo`, `tar`) with `shell=False` — standard bandit noise for legitimate subprocess use, not exploitable. No fix applied.

## Coverage Disclosure (APTS § Reporting)
| Tool | Ran? | Version | Files covered | Skipped reason |
|------|------|---------|---------------|----------------|
| Gitleaks | OK | 8.30.1 | full git history (1795 commits) + working tree | — |
| TruffleHog | OK | 3.97.5 | full git history (19701 chunks) | — |
| Semgrep | OK | 1.177.0 | tracked files matching rule includes; 2 files >300KB skipped (`src/cmds/git/diff_cmd.rs`, `src/hooks/init.rs`) | oversized-file limit (`--max-target-bytes 300000`) |
| Bandit | OK | 1.9.4 | 3 `.py` files (`hooks/hermes/`, `scripts/benchmark-sessions/`) | — |
| OSV-Scanner | OK | 2.6.0 | Cargo.lock (213 packages) + Maven test fixtures | — |
| Trivy | SKIPPED | 0.74.0 | none | vulnerability DB download failed (network), retried once, both failed |
| mcps-audit | N/A | — | — | no MCP config files (`.mcp*.json`, `*.skill`) in target scope |
| mcp-exfil-scan | SKIPPED | — | — | bundled script failed integrity checksum |
| CodeQL | N/A | — | — | out of scope for this task (requires GH Actions workflow run, not invoked) |
| skill-security-auditor / skillspector | N/A | — | — | out of scope — task explicitly scopes to Rust source, Cargo.toml, Cargo.lock |

## Cross-Tool Observations
Gitleaks and TruffleHog agree: the 38 gitleaks pattern-matches are not real secrets (TruffleHog's live-verification found 0 verified/unverified). No cross-tool corroboration of an actual vulnerability beyond the single OSV-Scanner rustls finding, which had no corresponding Trivy run to cross-check (Trivy skipped).

## Coverage Gaps
- Business logic / IDOR / runtime behavior: not covered by static tools.
- Trivy dependency + IaC + secrets scan: not completed this run (network failure) — OSV-Scanner substitutes for the Cargo.lock portion.
- MCP/skill-specific audits (config-audit, skill-audit, mcp-exfil-scan, skillspector): out of scope per task instructions (scope = "every Rust source file, Cargo.toml, Cargo.lock"); mcp-exfil-scan additionally blocked by a failed integrity check regardless.

## Merge Conflict Note (STEP 2)
One conflict during the upstream merge: `src/cmds/dotnet/dotnet_cmd.rs` — both sides changed the same `quick_xml` text-decoding call as part of independent edition-2024/let-chain refactors (HEAD: `.decode()` in an `if let { if ... }`; upstream: `.unescape()` in a let-chain). Resolved by keeping upstream's let-chain structure but with `.decode()`, since this repo pins `quick-xml = "0.41"` and `BytesText::unescape()` does not exist in that version (`cargo build` caught this immediately — fixed and reverified). No fork-specific logic was lost.

## Raw Scanner Output

### Gitleaks (git history + working tree)
```
1795 commits scanned.
scanned ~12336760 bytes (12.34 MB) in 2.08s
leaks found: 38
```
38/38 findings reviewed — see Non-Issues above. Full SARIF retained at the scan scratch dir for this run (not committed — contains file:line references to test fixtures only, no real secrets).

### TruffleHog (git, live-verification)
```
2026-09-18T10:55:43+07:00 info-0 trufflehog finished scanning
{"chunks": 19701, "bytes": 13850596, "verified_secrets": 0, "unverified_secrets": 0,
 "scan_duration": "2.544757459s", "trufflehog_version": "3.97.5"}
```

### Semgrep — p/owasp-top-ten
```
Ran 266 rules on 12 files: 0 findings.
```

### Semgrep — p/secrets
```
Ran 45 rules on 426 files: 0 findings.
```

### Semgrep — p/python
```
Ran 151 rules on 2 files: 0 findings.
```

### Bandit
```
Run metrics:
    Total issues (by severity):   Undefined: 0  Low: 9  Medium: 0  High: 0
    Total issues (by confidence): Undefined: 0  Low: 0  Medium: 0  High: 9
```
All 9: `B404` (subprocess import) / `B607` (partial executable path) / `B603` (subprocess without shell=True) on `rtk`, `cargo`, `tar` invocations in `hooks/hermes/rtk-rewrite/__init__.py`, `hooks/hermes/tests/test_rtk_rewrite_plugin.py`, `scripts/benchmark-sessions/lib/runner.py`. All `shell=False`, hardcoded trusted argv — no injection surface.

### OSV-Scanner
```
Total 1 package affected by 1 known vulnerability (0 Critical, 0 High, 1 Medium, 0 Low, 0 Unknown) from 1 ecosystem.
1 vulnerability can be fixed.

+-----------------------------------+------+-----------+---------+---------+---------------+
| OSV URL                           | CVSS | ECOSYSTEM | PACKAGE | VERSION | FIXED VERSION |
+-----------------------------------+------+-----------+---------+---------+---------------+
| https://osv.dev/RUSTSEC-2026-0285 | 5.3  | crates.io | rustls  | 0.23.37 | 0.23.45       |
+-----------------------------------+------+-----------+---------+---------+---------------+
```
→ Fixed (see Fixed Issues #1).

### Trivy
```
FATAL Fatal error  run error: init error: DB error: failed to download vulnerability DB:
OCI artifact error: ... read tcp ...: read: connection reset by peer
```
Failed identically on retry — network issue in this sandbox, not a code finding. Skipped per Coverage Disclosure.

### APTS Audit Log
- **Log:** `/tmp/css-scan-20260918T030548Z.jsonl`
- **Tool runs recorded:** 13 (measured: 13, asserted: 0)
- **Standard:** OWASP APTS § Auditability
