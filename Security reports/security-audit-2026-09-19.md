# Security Audit — 2026-09-19

## Summary
- Issues found: 1 dependency advisory + 74 triaged non-issues (38 gitleaks + 36 semgrep) | Auto-fixed: 1 | Unresolved: 0 real vulnerabilities (2 finding classes triaged below)
- Status: ISSUES FIXED

Scope: merged 72 upstream commits from `origin/develop` (worktree HEAD `496c94ba`), scanned the full worktree: Rust sources, `Cargo.toml`, `Cargo.lock`, workflows, scripts.

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | `rustls` (transitive via `ureq` 2.12.1) | RUSTSEC-2026-0285 (CVSS 5.3, Medium) | `cargo update -p rustls`: 0.23.37 → 0.23.45 (also pulled `rustls-webpki` 0.103.13 → 0.103.15). Only `Cargo.lock` changed. OSV-Scanner re-run afterwards: see raw output. |

## Unresolved Issues

### 1. Gitleaks — 38 hits, all false positives
Same count and class as prior audits (synthetic secret-shaped fixture in `tests/guard_integration_test.rs` and example strings in historical `Security reports/*.md`). TruffleHog found 0 verified / 0 unverified secrets over the full history (19872 chunks), confirming no live secret.

### 2. Semgrep — 36 hits: GitHub Actions mutable-tag references (CI-hardening, deferred)
`yaml.github-actions.security.github-actions-mutable-action-tag` across `.github/workflows/*.yml`. **Reason unresolved:** a correct fix needs each action's release tag resolved to a verified 40-char commit SHA; guessing would break CI or pin the wrong commit. Same deferred item as every prior audit; needs a manual pass.

### Bandit — 9 Low/High-confidence hits, no fix needed
B404/B603/B607 on list-form `subprocess.run(["tar", ...])` in `scripts/benchmark-sessions/` (no `shell=True`, no interpolation) — already the secure pattern.

## Scanner Notes
- Bundled scripts of the security-scanner skill (`config-audit.py`, `skill-audit.sh`, `mcp-exfil-scan.sh`, `apts-audit.sh`) were **not run**: their SHA256SUMS integrity check reported MISMATCH (skill instructs not to run them). Claude-config/skill/MCP-exfil audit is therefore incomplete this run. mcp-scan / skillspector LLM mode are opt-in and were not used (autonomous run). cargo-audit is not installed; OSV-Scanner + Trivy cover Cargo.lock.
- Trivy reported 0 vulnerabilities for `Cargo.lock` while OSV-Scanner flagged RUSTSEC-2026-0285 — advisory DB lag in Trivy.
- Merge conflict in `src/cmds/dotnet/dotnet_cmd.rs` resolved by taking upstream's version (per task policy); the local side had used `e.decode()` where upstream uses `e.unescape()`. Upstream's `unescape()` does not exist in the fork's pinned `quick-xml` 0.41 (upstream is on 0.37), so `cargo install` initially failed with E0599; fixed by keeping the fork's `e.decode()` call (now in upstream's let-chain form).

## Build & Verification
- `cargo install --path . --force`: OK (rtk v0.48.0, installed to `~/.cargo/bin`, PATH wrapper rewritten at `~/.local/bin/rtk`).
- `cargo fmt --check`: clean | `cargo clippy --all-targets`: 0 warnings | `cargo test --all`: all suites pass (3671 unit tests, 0 failed).

## Raw Scanner Output

### Pre-flight
```
OK  bandit 1.9.4 | semgrep 1.177.0 | trivy 0.74.0 | trufflehog 3.97.5 | gitleaks 8.30.1 | osv-scanner 2.6.0
MISSING  cargo-audit
WARN  bundled-script checksum MISMATCH (bundled scripts skipped)
```

### Gitleaks
```
9:09AM INF 1813 commits scanned.
9:09AM INF scanned ~12452860 bytes (12.45 MB) in 736ms
9:09AM WRN leaks found: 38
exit=1
```

### TruffleHog
```
2026-09-19T09:09:20+07:00	info-0	trufflehog	finished scanning	{"chunks": 19872, "bytes": 13975210, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "946.618333ms", "trufflehog_version": "3.97.5", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
```

### Trivy (fs)
```
Report Summary

┌───────────────────────────────────────────────────────────┬───────┬─────────────────┬─────────┐
│                          Target                           │ Type  │ Vulnerabilities │ Secrets │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ Cargo.lock                                                │ cargo │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-fail-skeleton/pom.xml         │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-skeleton/child-a/pom.xml      │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-skeleton/child-b/pom.xml      │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multi-module-skeleton/pom.xml              │  pom  │        0        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ tests/fixtures/multifail-skeleton/pom.xml                 │  pom  │        0        │    -    │
└───────────────────────────────────────────────────────────┴───────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)

exit=0

```

### OSV-Scanner — before fix
```
Total 1 package affected by 1 known vulnerability (0 Critical, 0 High, 1 Medium, 0 Low, 0 Unknown) from 1 ecosystem.
1 vulnerability can be fixed.

+-----------------------------------+------+-----------+---------+---------+---------------+------------+
| OSV URL                           | CVSS | ECOSYSTEM | PACKAGE | VERSION | FIXED VERSION | SOURCE     |
+-----------------------------------+------+-----------+---------+---------+---------------+------------+
| https://osv.dev/RUSTSEC-2026-0285 | 5.3  | crates.io | rustls  | 0.23.37 | 0.23.45       | Cargo.lock |
+-----------------------------------+------+-----------+---------+---------+---------------+------------+
exit=1

```

### OSV-Scanner — after fix
```
End status: 123 dirs visited, 769 inodes visited, 8 Extract calls, 21.167041ms elapsed, 21.168ms wall time
Warning: enricher transitivedependency/pomxml may be risky when run on untrusted artifacts. Please ensure you trust the source code and artifacts.

No issues found
```

### Semgrep
```
p/owasp-top-ten: Ran 292 rules on 152 files: 36 findings (all github-actions-mutable-action-tag)
p/secrets:       Ran 45 rules on 426 files: 0 findings
```

### Bandit
```
Run metrics:
	Total issues (by severity):
		Undefined: 0
		Low: 9
		Medium: 0
		High: 0
	Total issues (by confidence):
		Undefined: 0
		Low: 0
		Medium: 0
		High: 9
Files skipped (0):
exit=1
```
