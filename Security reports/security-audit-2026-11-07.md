# Security Audit — 2026-11-07

## Summary
- Issues found: 6 (advisory groups; 20 advisory-to-crate rows in Trivy/OSV) | Auto-fixed: 4 | Unresolved: 2
- Status: ISSUES REMAINING

Scope: `Cargo.lock` and `proxy/Cargo.lock` after upstream sync (origin/develop merged, 543 upstream commits).
Git HEAD at scan: 41928760 (merge commit, pre-fix; lockfile fixes applied afterwards and uncommitted).

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0346, -0347, -0348, -0349; GHSA-c8m5-c952-6cm4, -vp88-798v-h67f, -h2v8-394m-p7vv, -w44r-93j7-qv28 (MEDIUM/LOW, CVSS 2.3–4.0) | 0.23.45 → 0.23.46 (`cargo update --precise`) |
| 2 | rustls-webpki (both lockfiles) | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 |
| 3 | webpki-roots (both lockfiles) | RUSTSEC-2026-0381 | 1.0.6 → 1.0.10 (root); 1.0.9 → 1.0.10 (proxy). The 0.26.11 copy in root was not flagged and is unchanged. |
| 4 | tokio (proxy/Cargo.lock) | RUSTSEC-2026-0386 | 1.53.1 → 1.53.3 |

Verification: `cargo check --all-targets` passes on the root crate after the lockfile updates (205 crates).

## Unresolved Issues
| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| 5 | lru 0.12.5 (Cargo.lock) | RUSTSEC-2026-0002 / GHSA-rhfx-m35p-ff5j (LOW, IterMut Stacked Borrows), RUSTSEC-2026-0253 | Transitive via `ratatui 0.29.0`, which requires `lru ^0.12`. Fixed versions 0.16.3 / 0.18.2 are semver-incompatible; `cargo update --precise 0.16.3` fails. Needs a ratatui upgrade, which is a separate change. |
| 6 | paste 1.0.15 (Cargo.lock) | RUSTSEC-2024-0436 (unmaintained) | Advisory lists no fixed version. Needs a replacement crate or accepted risk. |

## Scanner Observations (not counted as issues)
- **Semgrep** (p/owasp-top-ten, p/secrets): 0 findings across 226 + 45 rules.
- **Gitleaks** (git history, 2324 commits): 39 historical hits. Most are in `Security reports/*` (earlier audit reports quoting patterns), plus test strings and docs in `proxy/tests/functional.rs`, `tests/guard_integration_test.rs`, `scripts/benchmark/cloud-init.yaml`, `src/cmds/cloud/aws_cmd.rs` and `SECURITY.md`. Not triaged line by line; likely false positives. Manual review recommended.
- **TruffleHog** (filesystem): 66 unverified detections, 0 verified. Most are in build output under `target/` (`.rlib`, `.rmeta`, copies of `rtk`). TruffleHog did not honor `.gitignore`, so `target/` was scanned. Source hits are `src/filters/liquibase.toml` and `src/filters/builtin_filters.toml` (JDBC/URI-style detectors on test strings).
- **config-audit** reported 15 CRITICAL and 13 HIGH, all in the global plugin cache under `~/.claude/plugins/` (e.g. the security-scanner's own scripts and ponytail benchmarks), not in this repo. Out of target scope. Not acted on; review manually.
- **skill-audit** overall risk score 5/100; **mcp-exfil-scan** risk score 0/100.
- **skillspector** (--no-llm) started but reported only "Skill discovery was incomplete; continuing with a bounded scan" and no findings. Coverage is partial.
- **APTS audit log**: the finalize step failed (`apts-audit.sh: line 66 syntax error`). Tools were run directly rather than through the `run` wrapper, so the log has no measured records.

## Raw Scanner Output
Secret values in TruffleHog "Raw result" lines are redacted. ANSI codes are stripped.

### Trivy (fs: vuln, secret, misconfig)
```

Report Summary

┌───────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┬───────────────────┐
│                          Target                           │    Type    │ Vulnerabilities │ Secrets │ Misconfigurations │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ Cargo.lock                                                │   cargo    │        5        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ proxy/Cargo.lock                                          │   cargo    │        4        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-fail-skeleton/pom.xml         │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-skeleton/child-a/pom.xml      │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-skeleton/child-b/pom.xml      │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multi-module-skeleton/pom.xml              │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/multifail-skeleton/pom.xml                 │    pom     │        0        │    -    │         -         │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ tests/fixtures/oc_pods.json                               │ kubernetes │        -        │    -    │         0         │
└───────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┴───────────────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


Cargo.lock (cargo)
==================
Total: 5 (UNKNOWN: 0, LOW: 4, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

┌─────────┬─────────────────────┬──────────┬────────┬───────────────────┬───────────────────────┬─────────────────────────────────────────────────────────────┐
│ Library │    Vulnerability    │ Severity │ Status │ Installed Version │     Fixed Version     │                            Title                            │
├─────────┼─────────────────────┼──────────┼────────┼───────────────────┼───────────────────────┼─────────────────────────────────────────────────────────────┤
│ lru     │ GHSA-rhfx-m35p-ff5j │ LOW      │ fixed  │ 0.12.5            │ 0.16.3                │ `IterMut` violates Stacked Borrows by invalidating internal │
│         │                     │          │        │                   │                       │ pointer                                                     │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-rhfx-m35p-ff5j           │
├─────────┼─────────────────────┼──────────┤        ├───────────────────┼───────────────────────┼─────────────────────────────────────────────────────────────┤
│ rustls  │ GHSA-c8m5-c952-6cm4 │ MEDIUM   │        │ 0.23.45           │ 0.23.46, 0.24.0-dev.2 │ Rustls: Session Debug formatting exposes TLS master secrets │
│         │                     │          │        │                   │                       │ and resumption PSKs                                         │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-c8m5-c952-6cm4           │
│         ├─────────────────────┼──────────┤        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-h2v8-394m-p7vv │ LOW      │        │                   │                       │ Rustls: Brotli certificate decompression can allocate an    │
│         │                     │          │        │                   │                       │ attacker-controlled amount of memory up...                  │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-h2v8-394m-p7vv           │
│         ├─────────────────────┤          │        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-vp88-798v-h67f │          │        │                   │                       │ rustls: ECH rejection can expose buffered application data  │
│         │                     │          │        │                   │                       │ to authenticated outer server...                            │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-vp88-798v-h67f           │
│         ├─────────────────────┤          │        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-w44r-93j7-qv28 │          │        │                   │                       │ rustls: An unauthenticated ServerHello poisons the per-name │
│         │                     │          │        │                   │                       │ key-exchange record, so later ClientHellos...               │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-w44r-93j7-qv28           │
└─────────┴─────────────────────┴──────────┴────────┴───────────────────┴───────────────────────┴─────────────────────────────────────────────────────────────┘

proxy/Cargo.lock (cargo)
========================
Total: 4 (UNKNOWN: 0, LOW: 3, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

┌─────────┬─────────────────────┬──────────┬────────┬───────────────────┬───────────────────────┬─────────────────────────────────────────────────────────────┐
│ Library │    Vulnerability    │ Severity │ Status │ Installed Version │     Fixed Version     │                            Title                            │
├─────────┼─────────────────────┼──────────┼────────┼───────────────────┼───────────────────────┼─────────────────────────────────────────────────────────────┤
│ rustls  │ GHSA-c8m5-c952-6cm4 │ MEDIUM   │ fixed  │ 0.23.45           │ 0.23.46, 0.24.0-dev.2 │ Rustls: Session Debug formatting exposes TLS master secrets │
│         │                     │          │        │                   │                       │ and resumption PSKs                                         │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-c8m5-c952-6cm4           │
│         ├─────────────────────┼──────────┤        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-h2v8-394m-p7vv │ LOW      │        │                   │                       │ Rustls: Brotli certificate decompression can allocate an    │
│         │                     │          │        │                   │                       │ attacker-controlled amount of memory up...                  │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-h2v8-394m-p7vv           │
│         ├─────────────────────┤          │        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-vp88-798v-h67f │          │        │                   │                       │ rustls: ECH rejection can expose buffered application data  │
│         │                     │          │        │                   │                       │ to authenticated outer server...                            │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-vp88-798v-h67f           │
│         ├─────────────────────┤          │        │                   │                       ├─────────────────────────────────────────────────────────────┤
│         │ GHSA-w44r-93j7-qv28 │          │        │                   │                       │ rustls: An unauthenticated ServerHello poisons the per-name │
│         │                     │          │        │                   │                       │ key-exchange record, so later ClientHellos...               │
│         │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-w44r-93j7-qv28           │
└─────────┴─────────────────────┴──────────┴────────┴───────────────────┴───────────────────────┴─────────────────────────────────────────────────────────────┘
```

### OSV-Scanner (source, recursive)
```
Scanning dir /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/proxy/Cargo.lock file and found 321 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-fail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multifail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-skeleton/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-skeleton/child-b/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/tests/fixtures/multi-module-skeleton/child-a/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/Cargo.lock file and found 250 packages
End status: 157 dirs visited, 1129 inodes visited, 9 Extract calls, 35.964458ms elapsed, 35.965ms wall time
Warning: enricher transitivedependency/pomxml may be risky when run on untrusted artifacts. Please ensure you trust the source code and artifacts.

Total 9 packages affected by 16 known vulnerabilities (0 Critical, 0 High, 2 Medium, 7 Low, 7 Unknown) from 1 ecosystem.
15 vulnerabilities can be fixed.

+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE       | VERSION  | FIXED VERSION | SOURCE           |
+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
| https://osv.dev/RUSTSEC-2026-0002   | 2.7  | crates.io | lru           | 0.12.5   | 0.16.3        | Cargo.lock       |
| https://osv.dev/GHSA-rhfx-m35p-ff5j |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0253   |      | crates.io | lru           | 0.12.5   | 0.18.2        | Cargo.lock       |
| https://osv.dev/RUSTSEC-2024-0436   |      | crates.io | paste         | 1.0.15   | --            | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0346   | 4.0  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/GHSA-c8m5-c952-6cm4 |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0347   | 2.3  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/GHSA-vp88-798v-h67f |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0348   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/GHSA-h2v8-394m-p7vv |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0349   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/GHSA-w44r-93j7-qv28 |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0382   |      | crates.io | rustls-webpki | 0.103.15 | 0.103.16      | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0381   |      | crates.io | webpki-roots  | 1.0.6    | 1.0.10        | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0346   | 4.0  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/GHSA-c8m5-c952-6cm4 |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0347   | 2.3  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/GHSA-vp88-798v-h67f |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0348   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/GHSA-h2v8-394m-p7vv |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0349   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/GHSA-w44r-93j7-qv28 |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0382   |      | crates.io | rustls-webpki | 0.103.15 | 0.103.16      | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0386   |      | crates.io | tokio         | 1.53.1   | 1.53.3        | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0381   |      | crates.io | webpki-roots  | 1.0.9    | 1.0.10        | proxy/Cargo.lock |
+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
```

### Gitleaks (detect, git history)
```
9:25AM INF 2324 commits scanned.
9:25AM INF scanned ~20537456 bytes (20.54 MB) in 1.25s
9:25AM WRN leaks found: 39
```

### TruffleHog (filesystem)
```
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-11-07T09:25:20+07:00	info-0	trufflehog	running source	{"source_manager_worker_id": "I2XQd", "job_id": 1, "source_id": 1, "source_name": "trufflehog - filesystem", "source_type": "SOURCE_TYPE_FILESYSTEM", "with_units": true}
Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/src/filters/liquibase.toml
Line: 78

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/src/filters/liquibase.toml
Line: 73

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/build/rtk-63c1dcde80506064/out/builtin_filters.toml
Line: 836

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/build/rtk-63c1dcde80506064/out/builtin_filters.toml
Line: 831

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/build/rtk-978cd73afbb21c80/out/builtin_filters.toml
Line: 836

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/build/rtk-978cd73afbb21c80/out/builtin_filters.toml
Line: 831

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: ESCAPED_UNICODE
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rmeta
Line: 10074

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 10108

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rmeta
Line: 19092

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rmeta
Line: 19087

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 10074

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rmeta
Line: 10108

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 19125

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 19120

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rmeta
Line: 19125

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rmeta
Line: 19120

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 19092

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 19087

Found unverified result 🐷🔑❓
Detector Type: BitbucketAppPassword
Decoder Type: UTF16
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librustix-f8e283e8cff64821.rmeta
Line: 10716

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 117351

Found unverified result 🐷🔑❓
Detector Type: URI
Decoder Type: HTML
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rmeta
Line: 807

Found unverified result 🐷🔑❓
Detector Type: URI
Decoder Type: HTML
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rlib
Line: 807

Found unverified result 🐷🔑❓
Detector Type: URI
Decoder Type: HTML
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rmeta
Line: 803

Found unverified result 🐷🔑❓
Detector Type: URI
Decoder Type: HTML
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rlib
Line: 803

Found unverified result 🐷🔑❓
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 117351

Found unverified result 🐷🔑❓
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 128436

Found unverified result 🐷🔑❓
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-7c2d7040f6ace84e.rlib
Line: 117351

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 65472

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 65467

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/libsyn-8f11821302f7eb39.rlib
Line: 265

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 65711

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 65706

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: ESCAPED_UNICODE
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 10074

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 19092

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 19087

Found unverified result 🐷🔑❓
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 128436

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 65472

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 65467

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/librtk-3b36611f03451aed.rlib
Line: 128436

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 70036

Found unverified result 🐷🔑❓
(🔍 Using cached verification)
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 128436

Found unverified result 🐷🔑❓
(🔍 Using cached verification)
Detector Type: NpmToken
Decoder Type: PLAIN
Raw result: [REDACTED]
Rotation_guide: https://howtorotate.com/docs/tutorials/npm/
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 128436

Found unverified result 🐷🔑❓
(🔍 Using cached verification)
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/librtk.rlib
Line: 128436

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 70036

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 69791

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-1648676eacc3b82d
Line: 69791

Found unverified result 🐷🔑❓
(🔍 Using cached verification)
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 70036

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/rtk-79a0aa2a3f04549c
Line: 70036

Found unverified result 🐷🔑❓
Detector Type: Box
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/rtk
Line: 69791

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rmeta
Line: 809

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rmeta
Line: 813

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rlib
Line: 844

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rlib
Line: 809

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rlib
Line: 848

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rlib
Line: 813

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rmeta
Line: 844

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rmeta
Line: 848

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rlib
Line: 848

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-31d8e470f0d0d77e.rmeta
Line: 848

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rmeta
Line: 844

Found unverified result 🐷🔑❓
Verification issue: i/o timeout
Detector Type: FTP
Decoder Type: PLAIN
Raw result: [REDACTED]
File: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/target/release/deps/liburl-031c51242f3b3c6c.rlib
Line: 844

2026-11-07T09:25:49+07:00	info-0	trufflehog	finished scanning	{"chunks": 97493, "bytes": 1234070882, "verified_secrets": 0, "unverified_secrets": 66, "scan_duration": "28.98252775s", "trufflehog_version": "3.99.7", "verification_caching": {"Hits":160,"Misses":94,"HitsWasted":31,"AttemptsSaved":129,"VerificationTimeSpentMS":274181}}
```

### Semgrep — OWASP Top Ten
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 369 files tracked by git with 560 Code rules:
                                                                                                                        
  Language      Rules   Files          Origin      Rules                                                                
 ─────────────────────────────        ───────────────────                                                               
  <multilang>       7     369          Community     560                                                                
  bash              1      20                                                                                           
  ts               71       9                                                                                           
  python          147       2                                                                                           
                                                                                                                        
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 226
 • Targets scanned: 369
 • Parsed lines: ~99.9%
 • Scan skipped: 
   ◦ Not matching --include patterns: 228
   ◦ Files larger than  files 0.3 MB: 3
   ◦ Files matching .semgrepignore patterns: 368
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 226 rules on 369 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### Semgrep — Secrets
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 597 files tracked by git with 52 Code rules:
                                                                                                                        
  Language      Rules   Files          Origin      Rules                                                                
 ─────────────────────────────        ───────────────────                                                               
  <multilang>      36     597          Community      52                                                                
  yaml              1      11                                                                                           
  ts                5       9                                                                                           
  python            1       2                                                                                           
  js                5       1                                                                                           
  ruby              2       1                                                                                           
                                                                                                                        
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 45
 • Targets scanned: 597
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Files larger than  files 0.3 MB: 3
   ◦ Files matching .semgrepignore patterns: 368
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 45 rules on 597 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### config-audit (security-audit)
```
============================================================
  Claude Code Security Audit
============================================================

[1/5] Scanning global settings...
[2/5] Scanning installed skills...
[3/5] Scanning installed plugins...
[4/5] Scanning project: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524
[5/5] Scanning project-specific settings...

============================================================
  Results
============================================================

Found 107 issue(s):
  CRITICAL: 15
  HIGH: 13
  MEDIUM: 61
  LOW: 18

🔴 [CRITICAL] plugin:ponytail/ponytail/4.9.0/benchmarks/correctness.js
  Data exfiltration: eval() execution combined with .env file access
  Evidence: // Functional correctness assertion: runs generated code against lightweight test
// cases per task. Proves "less code" is not "broken code". Spawns python/node
// with the extracted code + appended a...

🔴 [CRITICAL] plugin:ponytail/ponytail/4.9.0/benchmarks/robustness-audit.js
  Data exfiltration: eval() execution combined with .env file access
  Evidence: // Robustness audit (issue #65 follow-up): find where ponytail actually breaks on a
// weak model. 12 tasks with classic edge-case traps. Each has a known-good and a
// known-lazy-wrong reference so t...

🔴 [CRITICAL] plugin:ponytail/ponytail/4.9.0/benchmarks/agentic/tasks.py
  Data exfiltration: ncat connection combined with .env file access
  Evidence: """Tasks for the agentic benchmark.

Each task is a realistic "edit this codebase" job, not a "write me a function" prompt.
The workspace is seeded with a starter file the agent must modify, which (a)...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/mcp-exfil-scan.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/bin/bash
# mcp-exfil-scan.sh — MCP Data Exfiltration Detection Scanner v1.0
# Detects data exfiltration risks in MCP servers, skills, and plugins
# Usage: bash mcp-exfil-scan.sh [scan-target-path]
...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/skill-audit.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with SSH directory access
  Evidence: #!/bin/bash
# skill-security-test.sh
# Comprehensive automated skill security tester v2.0
# Usage: bash skill-security-test.sh <URL_OR_FILE_PATH>

set -uo pipefail

SKILL_INPUT="$1"
TEST_DIR="/tmp/ski...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/config-audit.py
  Data exfiltration: ncat connection combined with SSH directory access
  Evidence: #!/usr/bin/env python3
"""
Claude Code Security Audit Scanner

Scans Claude Code configurations, skills, MCP servers, and project files
for potentially malicious commands, hooks, and data exfiltration...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/scripts/sign-binary-checksums.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/usr/bin/env node
import { createHash, createPrivateKey, createPublicKey, sign, verify } from "node:crypto";
import { readFileSync, writeFileSync } from "node:fs";
import { resolve } from "node:path...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/src/tools/caveman-init.js
  Data exfiltration: curl to external URL combined with .env file access
  Evidence: #!/usr/bin/env node
// caveman init — drop the always-on caveman activation rule into a target
// repo for every IDE agent we support. Idempotent. Safe to re-run.
//
// Usage:
//   node src/tools/cave...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/src/hooks/uninstall.sh
  Data exfiltration: curl to external URL combined with .env file access
  Evidence: #!/bin/bash
# caveman — uninstaller for the SessionStart + UserPromptSubmit hooks
# Removes: hook files in ~/.claude/hooks, settings.json entries, and the flag file
# Usage: bash src/hooks/uninstall.s...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/src/hooks/install.sh
  Data exfiltration: curl to external URL combined with .env file access
  Evidence: #!/bin/bash
# caveman — one-command hook installer for Claude Code
# Installs: SessionStart hook (auto-load rules) + UserPromptSubmit hook (mode tracking)
# Usage: bash src/hooks/install.sh
#   or:  b...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/browse/bin/binary-installer.generated.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import { createHash, createPublicKey, randomBytes, verify } from "node:crypto";
import {
  accessSync,
  chmodSync,
  constants,
  mkdirSync,
  readFileSync,
  renameSync,
  statSync,
  unlinkSync,
} ...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/packages/cli/src/native-hook-fast.ts
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import { execFileSync, spawnSync } from "node:child_process";
import { appendFileSync, chmodSync, lstatSync, mkdirSync, readFileSync, realpathSync, statSync } from "node:fs";
import { homedir } from "...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/packages/shared/binary-installer/installer.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import { createHash, createPublicKey, randomBytes, verify } from "node:crypto";
import {
  accessSync,
  chmodSync,
  constants,
  mkdirSync,
  readFileSync,
  renameSync,
  statSync,
  unlinkSync,
} ...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/shrink/bin/binary-installer.generated.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import { createHash, createPublicKey, randomBytes, verify } from "node:crypto";
import {
  accessSync,
  chmodSync,
  constants,
  mkdirSync,
  readFileSync,
  renameSync,
  statSync,
  unlinkSync,
} ...

🔴 [CRITICAL] plugin:caveman/caveman/2f49f0e1a352/mcp/bin/binary-installer.generated.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import { createHash, createPublicKey, randomBytes, verify } from "node:crypto";
import {
  accessSync,
  chmodSync,
  constants,
  mkdirSync,
  readFileSync,
  renameSync,
  statSync,
  unlinkSync,
} ...

🟠 [HIGH] skill:optimize/SKILL.md
  Suspicious command: netcat connection
  Evidence: ---
name: optimize
description: Diagnoses and fixes UI performance across loading speed, rendering, animations, images, and bundle size. Use when the user mentions slow, laggy, janky, performance, bun...

🟠 [HIGH] skill:qa/SKILL.md
  Suspicious command: curl to external URL
  Evidence: ---
name: qa
description: QA-test a website or web app and return a 1-5 quality score (5 = flawless, 1 = broken) with evidence. Use when the user wants to test, QA, evaluate, score, or "check how good...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/0.1.1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/0.1.1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/58267944f921/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/58267944f921/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/41f619d61b99/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/41f619d61b99/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: chmod 777 (world writable)
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: recursive delete from root
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.11.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: filesystem format command
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/CLAUDE.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟠 [HIGH] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/claude.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Notification[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" notification"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" session-end"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionStart[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" session-start"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" stop"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → StopFailure[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStart[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" subagent-start"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStop[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" subagent-stop"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → TeammateIdle[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json
  Dangerous mode permission prompt is disabled
  Evidence: skipDangerousModePermissionPrompt: true

🟡 [MEDIUM] skill:playwright-cli/SKILL.md
  Sensitive file reference: cookie/browser data access
  Evidence: ---
name: playwright-cli
description: Automate browser interactions, test web pages and work with Playwright tests.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*) Bash(npm:*)
---

# Browser Automat...

🟡 [MEDIUM] skill:playwright-cli/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: playwright-cli
description: Automate browser interactions, test web pages and work with Playwright tests.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*) Bash(npm:*)
---

# Browser Automat...

🟡 [MEDIUM] skill:wayfinder/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: wayfinder
description: Plan a huge chunk of work (more than one agent session can hold) as a shared map of decision tickets on your issue tracker, and resolve them one at a time until the wa...

🟡 [MEDIUM] skill:handoff/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
disable-model-invocation:...

🟡 [MEDIUM] skill:ask-matt/SKILL.md
  Sensitive file reference: .env file access
  Evidence: ---
name: ask-matt
description: Ask which skill or flow fits your situation. A router over the skills in this repo.
disable-model-invocation: true
---

# Ask Matt

You don't remember every skill, so a...

🟡 [MEDIUM] skill:ask-matt/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: ask-matt
description: Ask which skill or flow fits your situation. A router over the skills in this repo.
disable-model-invocation: true
---

# Ask Matt

You don't remember every skill, so a...

🟡 [MEDIUM] skill:wizard/SKILL.md
  Sensitive file reference: .env file access
  Evidence: ---
name: wizard
description: Generate an interactive bash wizard that walks a human through steps only they can perform. Use when provisioning infrastructure, setting up credentials or CI secrets, wa...

🟡 [MEDIUM] skill:wizard/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: wizard
description: Generate an interactive bash wizard that walks a human through steps only they can perform. Use when provisioning infrastructure, setting up credentials or CI secrets, wa...

🟡 [MEDIUM] skill:claude-handoff/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: claude-handoff
description: Hand the current conversation off to a fresh background agent that picks up the work immediately.
argument-hint: "What will the next session be used for?"
disable...

🟡 [MEDIUM] skill:taste/SKILL.md
  Sensitive file reference: cookie/browser data access
  Evidence: ---
name: taste
description: Reverse-engineer the design taste of any website. Given a URL, captures DOM data and a screenshot via real browser, then runs a 4-step analysis to produce taste.md + taste...

🟡 [MEDIUM] skill:taste/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: taste
description: Reverse-engineer the design taste of any website. Given a URL, captures DOM data and a screenshot via real browser, then runs a 4-step analysis to produce taste.md + taste...

🟡 [MEDIUM] skill:ego-browser/SKILL.md
  Sensitive file reference: cookie/browser data access
  Evidence: ---
name: ego-browser
description: When you need a browser, read this Skill by default. Use it to open and operate websites, fill forms, click buttons, take screenshots, extract page data, sign in, an...

🟡 [MEDIUM] skill:vibe/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: vibe
description: Vibe Code Orchestrator (VCO) is a governed runtime entry that freezes requirements, bounds execution, and enforces verification and phase cleanup.
---

# Vibe Governed Runt...

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: .env file access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 3.1.1
authors:
  - AnySearch Team
cr...

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 3.1.1
authors:
  - AnySearch Team
cr...

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 3.1.1
authors:
  - AnySearch Team
cr...

🟡 [MEDIUM] skill:migrate-to-shoehorn/SKILL.md
  Sensitive file reference: cookie/browser data access
  Evidence: ---
name: migrate-to-shoehorn
description: Migrate test files from `as` type assertions to @total-typescript/shoehorn. Use when user mentions shoehorn, wants to replace `as` in tests, or needs partial...

🟡 [MEDIUM] plugin:claude-video/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/check-setup.sh", "timeout": 5}]}

🟡 [MEDIUM] plugin:openai-codex/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/session-lifecycle-hook.mjs\" SessionStart", "timeout": 5}]}

🟡 [MEDIUM] plugin:openai-codex/hooks.json → SessionEnd[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/session-lifecycle-hook.mjs\" SessionEnd", "timeout": 5}]}

🟡 [MEDIUM] plugin:openai-codex/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/stop-review-gate-hook.mjs\"", "timeout": 900}]}

🟡 [MEDIUM] plugin:checklist-design/hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/check-design-review-trigger.mjs\""}]}

🟡 [MEDIUM] plugin:impeccable/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "[ ! -f \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/impeccable\" ] || \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/impeccable\" hook", "timeout": 5...

🟡 [MEDIUM] plugin:impeccable/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "[ ! -f \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/impeccable\" ] || \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/impeccable\" hook", "timeout": 3...

🟡 [MEDIUM] plugin:ponytail/claude-codex-hooks.json → SubagentStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/ponytail-subagent.js\"", "timeout": 5, "statusMessage": "Loading ponytail mode..."}]}

🟡 [MEDIUM] plugin:ponytail/claude-codex-hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/ponytail-mode-tracker.js\"", "timeout": 5, "statusMessage": "Tracking ponytail mode..."}]}

🟡 [MEDIUM] plugin:ponytail/copilot-hooks.json → sessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"type": "command", "bash": "node \"${PLUGIN_ROOT}/hooks/ponytail-activate.js\"", "powershell": "node \"${PLUGIN_ROOT}\\hooks\\ponytail-activate.js\"", "timeoutSec": 5}

🟡 [MEDIUM] plugin:ponytail/copilot-hooks.json → userPromptSubmitted[0]
  Broad matcher '' — runs on every operation
  Evidence: {"type": "command", "bash": "node \"${PLUGIN_ROOT}/hooks/ponytail-mode-tracker.js\"", "powershell": "node \"${PLUGIN_ROOT}\\hooks\\ponytail-mode-tracker.js\"", "timeoutSec": 5}

🟡 [MEDIUM] plugin:ponytail/qoder-hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node PONYTAIL_DIR/hooks/ponytail-mode-tracker.js"}]}

🟡 [MEDIUM] plugin:addy-agent-skills/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/ensure_agent_sdk.py\"", "timeout": 180}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\""}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\"", "asyncRewake": true, "rewakeMessage": "Backgrou...

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/ensure_agent_sdk.py\"", "timeout": 180}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\""}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\"", "asyncRewake": true, "rewakeMessage": "Backgrou...

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/ensure_agent_sdk.py\"", "timeout": 180}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\""}]}

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/sg-python.sh\" \"${CLAUDE_PLUGIN_ROOT}/hooks/security_reminder_hook.py\"", "asyncRewake": true, "rewakeMessage": "Backgrou...

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

🟡 [MEDIUM] plugin:caveman/plugin.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/src/hooks/caveman-activate.js\"", "timeout": 5, "statusMessage": "Loading caveman mode..."}]}

🟡 [MEDIUM] plugin:caveman/plugin.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/src/hooks/caveman-mode-tracker.js\"", "timeout": 5, "statusMessage": "Tracking caveman mode..."}]}

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/CLAUDE.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/CLAUDE.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/claude.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/claude.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🔵 [LOW] /Users/mp3wizard/.claude/settings.json
  Hooks configuration found
  Evidence: ["Notification", "PermissionRequest", "PostCompact", "PostToolUse", "PostToolUseFailure", "PreCompact", "PreToolUse", "SessionEnd", "SessionStart", "Stop", "StopFailure", "SubagentStart", "SubagentSto...

🔵 [LOW] plugin:claude-video/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:openai-codex/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "SessionEnd", "Stop"]

🔵 [LOW] plugin:checklist-design/hooks.json
  Hooks configuration found
  Evidence: ["UserPromptSubmit"]

🔵 [LOW] plugin:engram/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:engram/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:impeccable/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PostToolUse", "Stop"]

🔵 [LOW] plugin:ponytail/claude-codex-hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "SubagentStart", "UserPromptSubmit"]

🔵 [LOW] plugin:ponytail/copilot-hooks.json
  Hooks configuration found
  Evidence: ["sessionStart", "userPromptSubmitted"]

🔵 [LOW] plugin:ponytail/qoder-hooks.json
  Hooks configuration found
  Evidence: ["UserPromptSubmit", "PreToolUse"]

🔵 [LOW] plugin:addy-agent-skills/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop"]

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

🔵 [LOW] plugin:caveman/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:caveman/plugin.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit"]
```

### skill-audit (skill-security-auditor)
```
Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/security-guardian/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (504 lines)

[2/13] Scanning for dangerous patterns...
  [CRITICAL] Destructive file deletion
      Line: 72:    .arg(&cmd) // Attacker can inject: `; rm -rf /`
      Line: 171:            "; rm -rf /",
  [CRITICAL] Format filesystem
      Line: 354:    const BLOCKED: &[&str] = &["rm", "dd", "mkfs"];
  [HIGH] Sensitive system file
      Line: 172:            "| cat /etc/passwd",
  Found 3 dangerous patterns! (+110 risk points)

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  3 file operations detected

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Description: CLI security expert for RTK - command injection, shell escaping, hook security...

[10/13] Checking allowed tools...
  Allowed tools: Read Grep Glob Bash
    Bash access (+25 risk points)

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 6
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~72

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 100/100

VERDICT: CRITICAL RISK
   DO NOT USE without thorough expert review!

RECOMMENDATION: REJECT

Analysis Summary:
   - Lines analyzed: 504
   - Dangerous patterns: 3
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 3
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/pr-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (332 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  1 file operations detected

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: pr-triage
  Description: >...
xargs: unterminated quote
  Author: 

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 7
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~4

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 5/100

VERDICT: LOW RISK
   Generally safe to use with normal precautions

RECOMMENDATION: APPROVE

Analysis Summary:
   - Lines analyzed: 332
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 1
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/issue-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (364 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  1 file operations detected

[6/13] Checking credential access...
  Potential credential access: credentials
  Found 1 credential access patterns (+20 risk points)

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: issue-triage
  Description: >...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 5
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~1

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 25/100

VERDICT: MEDIUM RISK
   Review carefully before use, test in sandbox

RECOMMENDATION: APPROVE WITH CAUTION

Analysis Summary:
   - Lines analyzed: 364
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 1
   - Credential access: 1

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/design-patterns/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (248 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  1 file operations detected

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: design-patterns
  Description: Rust design patterns for RTK. Newtype, Builder, RAII, Trait Objects, State Machine. Applied to CLI f...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 0
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~56

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 5/100

VERDICT: LOW RISK
   Generally safe to use with normal precautions

RECOMMENDATION: APPROVE

Analysis Summary:
   - Lines analyzed: 248
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 1
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092613.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/rtk-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (244 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  1 file operations detected
    10:  - Write

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: rtk-triage
  Description: >...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 4
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~2

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 5/100

VERDICT: LOW RISK
   Generally safe to use with normal precautions

RECOMMENDATION: APPROVE

Analysis Summary:
   - Lines analyzed: 244
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 1
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/repo-recap/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (216 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  Found 1 unique URLs:
    TRUSTED: https://github.com/

[5/13] Checking file operations...
  2 file operations detected

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Description: Generate a comprehensive repo recap (PRs, issues, releases) for sharing with team. Pass en or fr as ...

[10/13] Checking allowed tools...
  Allowed tools: Bash Read Grep
    Bash access (+25 risk points)

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 4
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~2

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 30/100

VERDICT: MEDIUM RISK
   Review carefully before use, test in sandbox

RECOMMENDATION: APPROVE WITH CAUTION

Analysis Summary:
   - Lines analyzed: 216
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 1
   - File operations: 2
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/ship/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (398 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  Found 1 unique URLs:
    TRUSTED: https://github.com/rtk-ai/rtk/releases/download/v0.17.0/rtk-macos-latest

[5/13] Checking file operations...
  2 file operations detected
    3:allowed-tools: Read Write Edit Bash Grep Glob

[6/13] Checking credential access...
  Potential credential access: \.env
  Found 1 credential access patterns (+20 risk points)

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Description: Build, commit, push & version bump workflow - automates the complete release cycle...

[10/13] Checking allowed tools...
  Allowed tools: Read Write Edit Bash Grep Glob
    Bash access (+25 risk points)
    Write access (+20 risk points)
    Edit access (+10 risk points)

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 21
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~2
  High bash complexity (+10 risk points)

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 90/100

VERDICT: CRITICAL RISK
   DO NOT USE without thorough expert review!

RECOMMENDATION: REJECT

Analysis Summary:
   - Lines analyzed: 398
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 1
   - File operations: 2
   - Credential access: 1

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092614.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/performance/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (435 lines)

[2/13] Scanning for dangerous patterns...
  [MEDIUM] Privilege escalation
      Line: 197:sudo dtrace -n 'syscall::open*:entry { @[execname] = count(); }' &
      Line: 199:sudo pkill dtrace
      Line: 424:| **dtrace** | System call tracing (macOS) | `sudo dtrace -n 'syscall::open*:entry'` |
  Found 1 dangerous patterns! (+20 risk points)

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  1 file operations detected

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  CRITICAL: Privilege escalation detected!
    197:sudo dtrace -n 'syscall::open*:entry { @[execname] = count(); }' &
    199:sudo pkill dtrace
    424:| **dtrace** | System call tracing (macOS) | `sudo dtrace -n 'syscall::open*:entry'` |

[9/13] Extracting metadata...
  Description: CLI performance optimization - startup time, memory usage, token savings benchmarking...

[10/13] Checking allowed tools...
  No allowed-tools declared (prompt-only skill, -10 risk points)

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 12
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~30
  High bash complexity (+10 risk points)

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 65/100

VERDICT: HIGH RISK
   Significant security concerns, needs mitigation

RECOMMENDATION: USE WITH EXTREME CAUTION

Analysis Summary:
   - Lines analyzed: 435
   - Dangerous patterns: 1
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 1
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/code-simplifier/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (178 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  2 file operations detected
    14:  - Edit

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: code-simplifier
  Description: Review RTK Rust code for idiomatic simplification. Detects over-engineering, unnecessary allocations...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  [Medium] Silent action instruction
  Found 1 prompt injection patterns!

[12/13] Analyzing code complexity...
  Bash code blocks: 1
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~49

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 25/100

VERDICT: MEDIUM RISK
   Review carefully before use, test in sandbox

RECOMMENDATION: APPROVE WITH CAUTION

Analysis Summary:
   - Lines analyzed: 178
   - Dangerous patterns: 0
   - Prompt injection patterns: 1
   - Network URLs: 0
   - File operations: 2
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/rtk-tdd/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (85 lines)

[2/13] Scanning for dangerous patterns...
  [HIGH] Sensitive system file
      Line: 57:| Validation/security | Boundary inputs -> assert bool | `assert!(!is_valid("../etc/passwd"))` |
  Found 1 dangerous patterns! (+30 risk points)

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  5 file operations detected
    10:  - Write
    11:  - Edit
    22:2. Write only enough test to fail (including compilation failure)
    23:3. Write only enough production code to pass the failing test
    30:1. Write test in #[cfg(test)] mod tests of the SAME file

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: rtk-tdd
  Description: >...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 1
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~9

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 35/100

VERDICT: MEDIUM RISK
   Review carefully before use, test in sandbox

RECOMMENDATION: APPROVE WITH CAUTION

Analysis Summary:
   - Lines analyzed: 85
   - Dangerous patterns: 1
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 5
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092615.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/tdd-rust/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092616.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (289 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  No hardcoded URLs

[5/13] Checking file operations...
  4 file operations detected
    13:  - Write
    14:  - Edit
    27:1. RED   — Write failing test with real fixture
    51:## Step 2: Write the Test (Red)

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Skill name: tdd-rust
  Description: TDD workflow for RTK filter development. Red-Green-Refactor with Rust idioms. Real fixtures, token s...

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 3
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~61

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 5/100

VERDICT: LOW RISK
   Generally safe to use with normal precautions

RECOMMENDATION: APPROVE

Analysis Summary:
   - Lines analyzed: 289
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 0
   - File operations: 4
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092616.txt

Skill Security Tester v2.0
==========================
Testing: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524/.claude/skills/pr-review/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092616.txt

[1/13] Acquiring skill file...
  Copying local file...
  Skill acquired (227 lines)

[2/13] Scanning for dangerous patterns...
  No dangerous patterns found

[3/13] Checking for code obfuscation...
  No obfuscation detected

[4/13] Analyzing network calls...
  Found 1 unique URLs:
    TRUSTED: https://github.com/rtk-ai/rtk/pull/

[5/13] Checking file operations...
  1 file operations detected
    15:  - Write

[6/13] Checking credential access...
  No credential access detected

[7/13] Checking external dependencies...
  No external dependencies

[8/13] Checking privilege requirements...
  No privilege escalation

[9/13] Extracting metadata...
  Description: >...
xargs: unterminated quote
  Author: 

[10/13] Checking allowed tools...
  Allowed tools: 

[11/13] Checking for prompt injection patterns...
  No prompt injection patterns detected

[12/13] Analyzing code complexity...
  Bash code blocks: 5
  Python code blocks: 0
  JS/TS code blocks: 0
  Executable lines: ~1

[13/13] Calculating final risk score...

==============================
    FINAL SECURITY REPORT     
==============================

OVERALL RISK SCORE: 5/100

VERDICT: LOW RISK
   Generally safe to use with normal precautions

RECOMMENDATION: APPROVE

Analysis Summary:
   - Lines analyzed: 227
   - Dangerous patterns: 0
   - Prompt injection patterns: 0
   - Network URLs: 1
   - File operations: 1
   - Credential access: 0

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261107_092616.txt
```

### mcp-exfil-scan
```
============================================================
  MCP Data Exfiltration Scanner v1.0
============================================================
Scan target: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/optimistic-elbakyan-4de524
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/mcp-exfil-20261107_092616.txt

MCP configs found: 0
Skill files found: 24

[1/6] Scanning MCP tool descriptions for exfiltration instructions...
  No tool description poisoning detected

[2/6] Scanning MCP server outbound data flow...
  No suspicious outbound data flow detected

[3/6] Scanning skill-level exfiltration chains...
  No exfiltration chains detected

[4/6] Scanning for encoded/obfuscated exfiltration payloads...
  No encoded/obfuscated exfiltration detected

[5/6] Scanning environment variable leaking via MCP servers...
  No environment variable leaking detected

[6/6] Verifying source trust for MCP servers and skills...
  All sources verified or no concerns

============================================================
  MCP Exfiltration Scan Results
============================================================

No MCP exfiltration risks detected.

RISK SCORE: 0/100
VERDICT: CLEAN

Coverage note: This scanner checks locally-configured MCP servers, skills,
and plugin files. For runtime MCP tool description analysis, use mcp-scan (opt-in).

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/mcp-exfil-20261107_092616.txt
```

### skillspector (--no-llm)
```
Warning: Skill discovery was incomplete; continuing with a bounded scan and 
reporting partial coverage.
```
