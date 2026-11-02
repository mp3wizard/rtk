# Security Audit — 2026-11-02

## Summary
- Issues found: 7 | Auto-fixed: 4 | Unresolved: 3
- Status: ISSUES REMAINING

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (Cargo.lock, proxy/Cargo.lock) | GHSA-w44r-93j7-qv28 | 0.23.45 → 0.23.46 |
| 2 | rustls-webpki (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 |
| 3 | webpki-roots (Cargo.lock, proxy/Cargo.lock) | RUSTSEC-2026-0381 | 1.0.6 / 1.0.9 → 1.0.10 |
| 4 | tokio (proxy/Cargo.lock) | RUSTSEC-2026-0386 | 1.53.1 → 1.53.3 |

## Unresolved Issues
1. **lru 0.12.5** (Cargo.lock, MEDIUM) — OSV-listed advisories RUSTSEC-2026-0002 and RUSTSEC-2026-0253 (fixed in 0.16.3 / 0.18.2). Pinned by `ratatui 0.29.0` (requires `^0.12`). Cannot bump without a ratatui upgrade — needs a human decision.
2. **paste 1.0.15** (Cargo.lock, LOW) — RUSTSEC-2024-0436, unmaintained crate, no fixed version. Transitive via `ratatui 0.29.0` (proc-macro).
3. **gitleaks — 39 hits in git history** (no fix applied). 21 `stripe-access-token` + 18 `generic-api-key`. Locations: `src/cmds/cloud/aws_cmd.rs` (5), `tests/guard_integration_test.rs` (2), `proxy/tests/functional.rs` (1), `scripts/benchmark/cloud-init.yaml` (2), `SECURITY.md` (1), and prior `Security reports/*.md` files (quoted pattern text). Current-tree scan finds no realistic-length `sk_live_` string; TruffleHog verified 0 secrets (4 unverified, JDBC localhost strings). Judged historical/false positive. Rewriting history is out of scope for this run.
   - Not counted as an issue: skill-audit reports CRITICAL 100/100 for `.claude/skills/security-guardian/SKILL.md`. Matches are example attack strings inside a security-guidance document. False positive.

## Raw Scanner Output
Tool outputs below are ANSI-stripped. Secret-looking strings were redacted with [REDACTED].

### gitleaks (history + tree)
```
9:14AM INF 2258 commits scanned.
9:14AM INF scanned ~19765035 bytes (19.77 MB) in 1.14s
9:14AM WRN leaks found: 39
```

### semgrep — OWASP
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 338 files tracked by git with 560 Code rules:
  Scanning 338 files with 7 <multilang> rules.
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 7
 • Targets scanned: 338
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Not matching --include patterns: 259
   ◦ Files larger than  files 0.3 MB: 3
   ◦ Files matching .semgrepignore patterns: 351
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 7 rules on 338 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### semgrep — secrets
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
   ◦ Files matching .semgrepignore patterns: 351
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 45 rules on 597 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### trivy fs (vuln, secret, misconfig)
```
2026-11-02T09:14:27+07:00	INFO	[vulndb] Need to update DB
2026-11-02T09:14:27+07:00	INFO	[vulndb] Downloading vulnerability DB...
2026-11-02T09:14:27+07:00	INFO	[vulndb] Downloading artifact...	repo="mirror.gcr.io/aquasec/trivy-db:2"
223.24 KiB / 131.42 MiB [>___________________________________________________________] 0.17% ? p/s ?1.50 MiB / 131.42 MiB [>_____________________________________________________________] 1.14% ? p/s ?3.87 MiB / 131.42 MiB [->____________________________________________________________] 2.95% ? p/s ?5.66 MiB / 131.42 MiB [-->______________________________________________] 4.30% 9.06 MiB p/s ETA 13s9.87 MiB / 131.42 MiB [--->_____________________________________________] 7.51% 9.06 MiB p/s ETA 13s10.91 MiB / 131.42 MiB [--->____________________________________________] 8.30% 9.06 MiB p/s ETA 13s14.64 MiB / 131.42 MiB [----->_________________________________________] 11.14% 9.44 MiB p/s ETA 12s17.77 MiB / 131.42 MiB [------>________________________________________] 13.52% 9.44 MiB p/s ETA 12s19.36 MiB / 131.42 MiB [------>________________________________________] 14.73% 9.44 MiB p/s ETA 11s23.66 MiB / 131.42 MiB [-------->______________________________________] 18.01% 9.80 MiB p/s ETA 10s25.88 MiB / 131.42 MiB [--------->_____________________________________] 19.69% 9.80 MiB p/s ETA 10s28.82 MiB / 131.42 MiB [---------->____________________________________] 21.93% 9.80 MiB p/s ETA 10s31.76 MiB / 131.42 MiB [----------->___________________________________] 24.16% 10.04 MiB p/s ETA 9s32.69 MiB / 131.42 MiB [----------->___________________________________] 24.88% 10.04 MiB p/s ETA 9s35.51 MiB / 131.42 MiB [------------>__________________________________] 27.02% 10.04 MiB p/s ETA 9s37.76 MiB / 131.42 MiB [------------->_________________________________] 28.73% 10.04 MiB p/s ETA 9s38.98 MiB / 131.42 MiB [------------->_________________________________] 29.66% 10.04 MiB p/s ETA 9s41.76 MiB / 131.42 MiB [-------------->________________________________] 31.77% 10.04 MiB p/s ETA 8s43.10 MiB / 131.42 MiB [--------------->________________________________] 32.80% 9.97 MiB p/s ETA 8s44.79 MiB / 131.42 MiB [---------------->_______________________________] 34.08% 9.97 MiB p/s ETA 8s46.85 MiB / 131.42 MiB [----------------->______________________________] 35.65% 9.97 MiB p/s ETA 8s47.29 MiB / 131.42 MiB [----------------->______________________________] 35.98% 9.77 MiB p/s ETA 8s49.04 MiB / 131.42 MiB [----------------->______________________________] 37.31% 9.77 MiB p/s ETA 8s50.13 MiB / 131.42 MiB [------------------>_____________________________] 38.15% 9.77 MiB p/s ETA 8s50.88 MiB / 131.42 MiB [------------------>_____________________________] 38.72% 9.53 MiB p/s ETA 8s52.23 MiB / 131.42 MiB [------------------->____________________________] 39.74% 9.53 MiB p/s ETA 8s52.48 MiB / 131.42 MiB [------------------->____________________________] 39.93% 9.53 MiB p/s ETA 8s54.85 MiB / 131.42 MiB [-------------------->___________________________] 41.74% 9.34 MiB p/s ETA 8s57.57 MiB / 131.42 MiB [--------------------->__________________________] 43.81% 9.34 MiB p/s ETA 7s58.88 MiB / 131.42 MiB [--------------------->__________________________] 44.80% 9.34 MiB p/s ETA 7s61.44 MiB / 131.42 MiB [---------------------->_________________________] 46.75% 9.45 MiB p/s ETA 7s63.07 MiB / 131.42 MiB [----------------------->________________________] 47.99% 9.45 MiB p/s ETA 7s64.41 MiB / 131.42 MiB [----------------------->________________________] 49.01% 9.45 MiB p/s ETA 7s66.35 MiB / 131.42 MiB [------------------------>_______________________] 50.49% 9.37 MiB p/s ETA 6s67.01 MiB / 131.42 MiB [------------------------>_______________________] 50.99% 9.37 MiB p/s ETA 6s69.04 MiB / 131.42 MiB [------------------------->______________________] 52.53% 9.37 MiB p/s ETA 6s70.51 MiB / 131.42 MiB [------------------------->______________________] 53.65% 9.21 MiB p/s ETA 6s71.35 MiB / 131.42 MiB [-------------------------->_____________________] 54.29% 9.21 MiB p/s ETA 6s72.63 MiB / 131.42 MiB [-------------------------->_____________________] 55.27% 9.21 MiB p/s ETA 6s73.41 MiB / 131.42 MiB [-------------------------->_____________________] 55.86% 8.93 MiB p/s ETA 6s74.54 MiB / 131.42 MiB [--------------------------->____________________] 56.72% 8.93 MiB p/s ETA 6s76.60 MiB / 131.42 MiB [--------------------------->____________________] 58.29% 8.93 MiB p/s ETA 6s77.04 MiB / 131.42 MiB [---------------------------->___________________] 58.62% 8.74 MiB p/s ETA 6s80.07 MiB / 131.42 MiB [----------------------------->__________________] 60.93% 8.74 MiB p/s ETA 5s82.32 MiB / 131.42 MiB [------------------------------>_________________] 62.64% 8.74 MiB p/s ETA 5s84.44 MiB / 131.42 MiB [------------------------------>_________________] 64.25% 8.97 MiB p/s ETA 5s86.82 MiB / 131.42 MiB [------------------------------->________________] 66.06% 8.97 MiB p/s ETA 4s87.66 MiB / 131.42 MiB [-------------------------------->_______________] 66.70% 8.97 MiB p/s ETA 4s89.48 MiB / 131.42 MiB [-------------------------------->_______________] 68.08% 8.93 MiB p/s ETA 4s91.01 MiB / 131.42 MiB [--------------------------------->______________] 69.25% 8.93 MiB p/s ETA 4s91.73 MiB / 131.42 MiB [--------------------------------->______________] 69.80% 8.93 MiB p/s ETA 4s93.23 MiB / 131.42 MiB [---------------------------------->_____________] 70.94% 8.76 MiB p/s ETA 4s94.26 MiB / 131.42 MiB [---------------------------------->_____________] 71.72% 8.76 MiB p/s ETA 4s95.54 MiB / 131.42 MiB [---------------------------------->_____________] 72.70% 8.76 MiB p/s ETA 4s97.19 MiB / 131.42 MiB [----------------------------------->____________] 73.96% 8.62 MiB p/s ETA 3s98.85 MiB / 131.42 MiB [------------------------------------>___________] 75.22% 8.62 MiB p/s ETA 3s100.73 MiB / 131.42 MiB [------------------------------------>__________] 76.64% 8.62 MiB p/s ETA 3s102.60 MiB / 131.42 MiB [------------------------------------>__________] 78.07% 8.65 MiB p/s ETA 3s103.91 MiB / 131.42 MiB [------------------------------------->_________] 79.07% 8.65 MiB p/s ETA 3s105.44 MiB / 131.42 MiB [------------------------------------->_________] 80.23% 8.65 MiB p/s ETA 3s106.19 MiB / 131.42 MiB [------------------------------------->_________] 80.80% 8.48 MiB p/s ETA 2s107.73 MiB / 131.42 MiB [-------------------------------------->________] 81.97% 8.48 MiB p/s ETA 2s109.23 MiB / 131.42 MiB [--------------------------------------->_______] 83.11% 8.48 MiB p/s ETA 2s109.85 MiB / 131.42 MiB [--------------------------------------->_______] 83.59% 8.32 MiB p/s ETA 2s111.13 MiB / 131.42 MiB [--------------------------------------->_______] 84.56% 8.32 MiB p/s ETA 2s112.01 MiB / 131.42 MiB [---------------------------------------->______] 85.23% 8.32 MiB p/s ETA 2s113.01 MiB / 131.42 MiB [---------------------------------------->______] 85.99% 8.13 MiB p/s ETA 2s114.26 MiB / 131.42 MiB [---------------------------------------->______] 86.94% 8.13 MiB p/s ETA 2s114.66 MiB / 131.42 MiB [----------------------------------------->_____] 87.25% 8.13 MiB p/s ETA 2s116.48 MiB / 131.42 MiB [----------------------------------------->_____] 88.63% 7.97 MiB p/s ETA 1s118.54 MiB / 131.42 MiB [------------------------------------------>____] 90.20% 7.97 MiB p/s ETA 1s121.04 MiB / 131.42 MiB [------------------------------------------->___] 92.10% 7.97 MiB p/s ETA 1s123.91 MiB / 131.42 MiB [-------------------------------------------->__] 94.29% 8.26 MiB p/s ETA 0s125.69 MiB / 131.42 MiB [-------------------------------------------->__] 95.64% 8.26 MiB p/s ETA 0s126.07 MiB / 131.42 MiB [--------------------------------------------->_] 95.93% 8.26 MiB p/s ETA 0s129.76 MiB / 131.42 MiB [---------------------------------------------->] 98.73% 8.35 MiB p/s ETA 0s130.85 MiB / 131.42 MiB [---------------------------------------------->] 99.57% 8.35 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 8.35 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.99 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.99 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.99 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.48 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.48 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.48 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.00 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.00 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [--------------------------------------------->] 100.00% 7.00 MiB p/s ETA 0s131.42 MiB / 131.42 MiB [-------------------------------------------------] 100.00% 7.56 MiB p/s 18s2026-11-02T09:14:46+07:00	INFO	[vulndb] Artifact successfully downloaded	repo="mirror.gcr.io/aquasec/trivy-db:2"
2026-11-02T09:14:46+07:00	INFO	[vuln] Vulnerability scanning is enabled
2026-11-02T09:14:46+07:00	INFO	[misconfig] Misconfiguration scanning is enabled
2026-11-02T09:14:46+07:00	INFO	[checks-client] Using existing checks from cache	path="/Users/mp3wizard/Library/Caches/trivy/policy/content"
2026-11-02T09:14:47+07:00	INFO	[secret] Secret scanning is enabled
2026-11-02T09:14:47+07:00	INFO	[secret] If your scanning is slow, please try '--scanners vuln,misconfig' to disable secret scanning
2026-11-02T09:14:47+07:00	INFO	[secret] Please see https://trivy.dev/docs/v0.76/guide/scanner/secret#recommendation for faster secret detection
2026-11-02T09:14:47+07:00	INFO	Number of language-specific files	num=9
2026-11-02T09:14:47+07:00	INFO	[cargo] Detecting vulnerabilities...
2026-11-02T09:14:47+07:00	INFO	[pom] Detecting vulnerabilities...
2026-11-02T09:14:47+07:00	INFO	Detected config files	num=1

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

### osv-scanner
```
Scanning dir /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/proxy/Cargo.lock file and found 319 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-fail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multifail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-skeleton/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-skeleton/child-b/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/tests/fixtures/multi-module-skeleton/child-a/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9/Cargo.lock file and found 248 packages
End status: 155 dirs visited, 1109 inodes visited, 9 Extract calls, 31.743958ms elapsed, 31.744ms wall time
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

### trufflehog (git)
```
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-11-02T09:14:53+07:00	info-0	trufflehog	running source	{"source_manager_worker_id": "ijIXL", "job_id": 1, "source_id": 1, "source_name": "trufflehog - git", "source_type": "SOURCE_TYPE_GIT", "with_units": true}
Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: jdbc:[REDACTED]
Commit: fc9ce3dcde5a7dbb52759a9953d4fb3bec553eac
Email: mp3wizard <mp3wizard@gmail.com>
File: Security reports/security-audit-2026-10-31.md
Line: 26
Repository: file:///Users/mp3wizard/Public/Claude%20Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9
Repository_local_path: /var/folders/l4/ncm6zgrd36n9m26knx1f79rr0000gn/T/trufflehog-95192-4284271306
Timestamp: 2026-10-31 03:34:03 +0000

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
(🔍 Using cached verification)
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: jdbc:[REDACTED]
Commit: 9053d33f874d3d92493948ee52d22f9aa0eb0503
Email: mp3wizard <mp3wizard@gmail.com>
File: Security reports/security-audit-2026-10-11.md
Line: 26
Repository: file:///Users/mp3wizard/Public/Claude%20Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9
Repository_local_path: /var/folders/l4/ncm6zgrd36n9m26knx1f79rr0000gn/T/trufflehog-95192-4284271306
Timestamp: 2026-10-11 15:24:43 +0000

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: jdbc:[REDACTED]
Commit: 630f0a2800e311d7aee80a9edca22f9f50315f83
Email: s2005 <s2005@users.noreply.github.com>
File: src/filters/liquibase.toml
Line: 78
Repository: file:///Users/mp3wizard/Public/Claude%20Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9
Repository_local_path: /var/folders/l4/ncm6zgrd36n9m26knx1f79rr0000gn/T/trufflehog-95192-4284271306
Timestamp: 2026-04-05 13:28:02 +0000

Found unverified result 🐷🔑❓
Verification issue: unsupported subprotocol: oracle
Detector Type: JDBC
Decoder Type: PLAIN
Raw result: jdbc:[REDACTED]
Commit: 630f0a2800e311d7aee80a9edca22f9f50315f83
Email: s2005 <s2005@users.noreply.github.com>
File: src/filters/liquibase.toml
Line: 73
Repository: file:///Users/mp3wizard/Public/Claude%20Proxy/rtk/.claude/worktrees/blissful-archimedes-5f4fc9
Repository_local_path: /var/folders/l4/ncm6zgrd36n9m26knx1f79rr0000gn/T/trufflehog-95192-4284271306
Timestamp: 2026-04-05 13:28:02 +0000

2026-11-02T09:14:53+07:00	info-0	trufflehog	finished scanning	{"chunks": 27389, "bytes": 21714231, "verified_secrets": 0, "unverified_secrets": 4, "scan_duration": "1.411170833s", "trufflehog_version": "3.99.7", "verification_caching": {"Hits":1,"Misses":4,"HitsWasted":0,"AttemptsSaved":1,"VerificationTimeSpentMS":707}}
```

### skill-audit (skill-audit.sh over SKILL.md)
```
Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/security-guardian/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/pr-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/issue-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/design-patterns/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091454.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/rtk-triage/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/repo-recap/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/ship/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/performance/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091455.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/code-simplifier/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/rtk-tdd/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/tdd-rust/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

Skill Security Tester v2.0
==========================
Testing: ./.claude/skills/pr-review/SKILL.md
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/test-20261102_091456.txt
```

## APTS Audit Log

### APTS Audit Log
- **Log:** `/tmp/css-scan-20261102T021421Z.jsonl`
- **Tool runs recorded:** 7 (measured: 7, asserted: 0)
- **Standard:** OWASP APTS § Auditability
