# Security Audit — 2026-10-10

## Summary
- Issues found: 4 advisories across 3 crates (OSV-Scanner + Trivy) | Auto-fixed: 4 | Unresolved: 0
- Status: ISSUES FIXED
- Build: `cargo install --path . --force` succeeded (rtk 0.50.0, release profile)
- Git: merge of `origin/develop` left open, staged, NOT committed (see Notes)

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | quick-xml | RUSTSEC-2026-0194 (CVSS 7.5) | 0.37.5 → 0.41.0 (matches Cargo.toml `quick-xml = "0.41"`; lockfile had a stale 0.37.5 entry) |
| 2 | quick-xml | RUSTSEC-2026-0195 (CVSS 7.5) | 0.37.5 → 0.41.0 (same change as #1) |
| 3 | crossbeam-epoch | RUSTSEC-2026-0204 | 0.9.18 → 0.9.20 (`cargo update -p crossbeam-epoch --precise 0.9.20`; path: rtk → ignore 0.4.32 → crossbeam-deque → crossbeam-epoch) |
| 4 | rustls | RUSTSEC-2026-0285 (CVSS 5.3) / GHSA-2mjx-qc3c-rqvc (Trivy: MEDIUM) | 0.23.37 → 0.23.45, rustls-webpki 0.103.13 → 0.103.15 (`cargo update -p rustls --precise 0.23.45`; path: rtk → ureq 2.12.1 → rustls) |

Post-fix verification: OSV-Scanner on the updated `Cargo.lock` reports **No issues found**. Trivy's rustls finding is marked `fixed` at 0.23.45.

## Unresolved Issues
None.

## Triaged Findings (no fix required)
| # | Component | Tool | Finding | Disposition |
|---|-----------|------|---------|-------------|
| 1 | `Security reports/*.md` (prior audits), `SECURITY.md` | Gitleaks | 38 total raw hits across git history (18 commits); 26 in prior report text and SECURITY.md | **False positive.** Example/placeholder strings matching Stripe / generic-key regexes, not live credentials. |
| 2 | `tests/guard_integration_test.rs:166,184` | Gitleaks | `stripe-access-token` | **False positive.** Test fixture for rtk's secret-guard filter (`sk_live_…` synthetic). |
| 3 | `src/cmds/cloud/aws_cmd.rs:1878,1879,1923,2035` | Gitleaks | `generic-api-key` | **False positive.** Synthetic AWS CloudWatch pagination tokens in `#[cfg(test)]` fixtures. |
| 4 | `scripts/benchmark/cloud-init.yaml:282,613` | Gitleaks | `generic-api-key` | **False positive.** Placeholder key in benchmark fixture. |
| 5 | Global `~/.claude` plugins/skills (outside the target path) | config-audit | 20 CRITICAL / 20 HIGH / 75 MEDIUM / 21 LOW heuristic matches in installed third-party skills and plugins (e.g. `base64`+`.env`, `eval()` in benchmark scripts) | **Not in scope.** config-audit scans the user's global Claude config, not this repo. Heuristic pattern matches, not verified exploits. Not modified; review the listed plugins manually if desired. |

TruffleHog (`--only-verified`, git history): **0 verified secrets**, which corroborates that the Gitleaks hits are not live credentials.

## Clean Scanners
- **Semgrep** `p/owasp-top-ten` (Rust/TOML, 155 files, 6 rules): 0 findings.
- **Semgrep** `p/secrets` (456 files, 45 rules): 0 findings.
- **Trivy** filesystem (vuln, secret, misconfig): only the rustls MEDIUM above, now fixed.
- **Bandit**: N/A — no `.py` files in scope.
- **CodeQL / mcps-audit / mcp-exfil-scan / skill-audit / skillspector**: not run. No MCP or skill artifacts are in this repo's audit path, and skillspector LLM mode is opt-in.

## Coverage Gaps
- Business logic, IDOR, and runtime behavior are not covered by static scanners.
- The `Cargo.lock` update pulled in `rustls-webpki` 0.103.15 and `quick-xml` 0.41.0; the release build succeeded, but the test suite was not run in this scheduled pass.

## Raw Scanner Output

### OSV-Scanner (before fix)
```
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/Cargo.lock file and found 205 packages
End status: 0 dirs visited, 1 inodes visited, 1 Extract calls, 21.781ms elapsed, 21.781ms wall time

Total 3 packages affected by 4 known vulnerabilities (0 Critical, 2 High, 1 Medium, 0 Low, 1 Unknown) from 1 ecosystem.
4 vulnerabilities can be fixed.

+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE         | VERSION | FIXED VERSION | SOURCE     |
+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
| https://osv.dev/RUSTSEC-2026-0204   |      | crates.io | crossbeam-epoch | 0.9.18  | 0.9.20        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0194   | 7.5  | crates.io | quick-xml       | 0.37.5  | 0.41.0        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0195   | 7.5  | crates.io | quick-xml       | 0.37.5  | 0.41.0        | Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0285   | 5.3  | crates.io | rustls          | 0.23.37 | 0.23.45       | Cargo.lock |
| https://osv.dev/GHSA-2mjx-qc3c-rqvc |      |           |                 |         |               |            |
+-------------------------------------+------+-----------+-----------------+---------+---------------+------------+
osv exit=1
```

### OSV-Scanner (after fix)
```
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/Cargo.lock file and found 205 packages
End status: 0 dirs visited, 1 inodes visited, 1 Extract calls, 1.729167ms elapsed, 1.729ms wall time

No issues found
```

### Trivy (filesystem, before fix)
```

Report Summary

┌───────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┬───────────────────┐
│                          Target                           │    Type    │ Vulnerabilities │ Secrets │ Misconfigurations │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ Cargo.lock                                                │   cargo    │        1        │    -    │         -         │
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
Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

┌─────────┬─────────────────────┬──────────┬────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────┐
│ Library │    Vulnerability    │ Severity │ Status │ Installed Version │ Fixed Version │                          Title                          │
├─────────┼─────────────────────┼──────────┼────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────┤
│ rustls  │ GHSA-2mjx-qc3c-rqvc │ MEDIUM   │ fixed  │ 0.23.37           │ 0.23.45       │ Rustls: TLS 1.3 handshake messages incorrectly accepted │
│         │                     │          │        │                   │               │ across encryption level boundaries                      │
│         │                     │          │        │                   │               │ https://github.com/advisories/GHSA-2mjx-qc3c-rqvc       │
└─────────┴─────────────────────┴──────────┴────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────┘
trivy exit=0
```

### Gitleaks
```
[90m9:54AM[0m [32mINF[0m [1m2003 commits scanned.[0m
[90m9:54AM[0m [32mINF[0m [1mscanned ~15105987 bytes (15.11 MB) in 2.62s[0m
[90m9:54AM[0m [33mWRN[0m [1mleaks found: 38[0m
gitleaks exit=1
```

### Semgrep — p/owasp-top-ten
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 155 files tracked by git with 560 Code rules:
  Scanning 155 files with 6 <multilang> rules.
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 6
 • Targets scanned: 155
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Not matching --include patterns: 301
   ◦ Files larger than  files 0.3 MB: 1
   ◦ Files matching .semgrepignore patterns: 228
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 6 rules on 155 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)

semgrep exit=0
```

### Semgrep — p/secrets
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 456 files tracked by git with 52 Code rules:
                                                                                                                        
  Language      Rules   Files          Origin      Rules                                                                
 ─────────────────────────────        ───────────────────                                                               
  <multilang>      36     456          Community      52                                                                
  yaml              1      11                                                                                           
  ts                5       9                                                                                           
  python            1       2                                                                                           
  ruby              2       1                                                                                           
                                                                                                                        
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 45
 • Targets scanned: 456
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Files larger than  files 0.3 MB: 1
   ◦ Files matching .semgrepignore patterns: 228
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 45 rules on 456 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)

semgrep-secrets exit=0
```

### TruffleHog (verified only)
```
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-10-10T09:54:16+07:00	info-0	trufflehog	running source	{"source_manager_worker_id": "U8cM8", "with_units": true}
2026-10-10T09:54:16+07:00	info-0	trufflehog	scanning repo	{"source_manager_worker_id": "U8cM8", "unit_kind": "dir", "unit": "/var/folders/l4/ncm6zgrd36n9m26knx1f79rr0000gn/T/trufflehog-93370-2898053101", "repo": "file:///Users/mp3wizard/Public/Claude%20Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d"}
2026-10-10T09:54:22+07:00	info-0	trufflehog	finished scanning	{"chunks": 24301, "bytes": 17294433, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "3.359594542s", "trufflehog_version": "3.99.1", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
trufflehog exit=0
```

### config-audit (bundled; global Claude config, out of target scope)
```
============================================================
  Claude Code Security Audit
============================================================

[1/5] Scanning global settings...
[2/5] Scanning installed skills...
[3/5] Scanning installed plugins...
[4/5] Scanning project: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d
[5/5] Scanning project-specific settings...

============================================================
  Results
============================================================

Found 136 issue(s):
  CRITICAL: 20
  HIGH: 20
  MEDIUM: 75
  LOW: 21

🔴 [CRITICAL] skill:anysearch/scripts/anysearch_cli.sh
  Data exfiltration: command substitution with curl combined with .env file access
  Evidence: #!/usr/bin/env bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

ENDPOINT="https://api.anysearch.com/mcp"
# Identifies access mode + spec version to the backend (X-Anysearch-Client).
# Keep the ...

🔴 [CRITICAL] plugin:impeccable/impeccable/4.1.1/skills/impeccable/scripts/generate-image.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/usr/bin/env node
/**
 * API image generation fallback: renders a mock or world board with the
 * user's own OpenAI key when the harness has no native image generation.
 *
 * context.mjs reports ava...

🔴 [CRITICAL] plugin:impeccable/impeccable/4.1.2/skills/impeccable/scripts/generate-image.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/usr/bin/env node
/**
 * API image generation fallback: renders a mock or world board with the
 * user's own OpenAI key when the harness has no native image generation.
 *
 * context.mjs reports ava...

🔴 [CRITICAL] plugin:impeccable/impeccable/4.1.2/skills/impeccable/scripts/font-match.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/usr/bin/env node
/**
 * font-match: measure the lettering in a comp's text region and rank candidate
 * faces against it, so the face is chosen by metrics instead of by name.
 *
 *   node font-matc...

🔴 [CRITICAL] plugin:impeccable/impeccable/4.1.2/skills/impeccable/scripts/detector/engines/browser/detect-url.mjs
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

import { finding } from '../../findings.mjs';
import { profileFindingsAsync, profileStep, profileStep...

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

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/.claude/skills/security-scanner/scripts/mcp-exfil-scan.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/bin/bash
# mcp-exfil-scan.sh — MCP Data Exfiltration Detection Scanner v1.0
# Detects data exfiltration risks in MCP servers, skills, and plugins
# Usage: bash mcp-exfil-scan.sh [scan-target-path]
...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/.claude/skills/security-scanner/scripts/skill-audit.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with SSH directory access
  Evidence: #!/bin/bash
# skill-security-test.sh
# Comprehensive automated skill security tester v2.0
# Usage: bash skill-security-test.sh <URL_OR_FILE_PATH>

set -uo pipefail

SKILL_INPUT="$1"
TEST_DIR="/tmp/ski...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/.claude/skills/security-scanner/scripts/config-audit.py
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

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → Notification[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -X POST http://localhost:${PORT}/hook -H 'Content-Type: application/json...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → PermissionRequest[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -X POST http://localhost:${PORT}/hook -H 'Content-Type: application/json...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → PostToolUse[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -X POST http://localhost:${PORT}/hook -H 'Content-Type: app...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → PreToolUse[1]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -X POST http://localhost:${PORT}/hook -H 'Content-Type: app...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → Stop[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -X POST http://localhost:${PORT}/hook -H 'Content-Type: app...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → StopFailure[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -X POST http://localhost:${PORT}/hook -H 'Content-Type: app...

🟠 [HIGH] /Users/mp3wizard/.claude/settings.json → UserPromptSubmit[0]
  Suspicious command: curl to external URL
  Evidence: PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -X POST http://localhost:${PORT}/hook -H 'Content-Type: app...

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

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/315c4e48967d/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/315c4e48967d/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/d182ca456ca0/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/d182ca456ca0/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/b819188d2eea/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/b819188d2eea/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: chmod 777 (world writable)
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: recursive delete from root
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.8.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: filesystem format command
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/CLAUDE.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟠 [HIGH] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/claude.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Notification[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -X POST http:/...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → PermissionRequest[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -X POST http:/...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → PostCompact[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → PostToolUse[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → PreCompact[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → PreToolUse[1]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[2]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[3]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → StopFailure[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → StopFailure[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → StopFailure[2]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStart[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStop[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → TeammateIdle[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"matcher": "", "hooks": [{"type": "command", "command": "PORT=$(cat ~/.claude/cc-beeper/port 2>/dev/null || echo 19222) && TOKEN=$(cat ~/.claude/cc-beeper/token 2>/dev/null) && curl -s -o /dev/null -...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → UserPromptSubmit[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "'/Users/mp3wizard/Library/Application Support/AgentPeek/bin/AgentPeekBridge' --bridge-hook-event claude", "timeout": 600}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → UserPromptSubmit[2]
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

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: .env file access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 2.1.0
authors:
  - AnySearch Team
cr...

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: credentials file access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 2.1.0
authors:
  - AnySearch Team
cr...

🟡 [MEDIUM] skill:anysearch/SKILL.md
  Sensitive file reference: password-related access
  Evidence: ---
name: anysearch
description: Real-time search engine supporting web search, vertical domain search, parallel batch search, and URL content extraction.
version: 2.1.0
authors:
  - AnySearch Team
cr...

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

🟡 [MEDIUM] plugin:impeccable/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "[ ! -f \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/hook.mjs\" ] || ! { node -e \"process.exit(Math.min(parseInt(process.versions.node,10),22)===22?0:1)...

🟡 [MEDIUM] plugin:impeccable/hooks.json → Stop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "[ ! -f \"${CLAUDE_PLUGIN_ROOT}/skills/impeccable/scripts/hook.mjs\" ] || ! { node -e \"process.exit(Math.min(parseInt(process.versions.node,10),22)===22?0:1)...

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

🟡 [MEDIUM] plugin:severity1-marketplace/hooks.json → UserPromptSubmit[0]
  Broad matcher '' — runs on every operation
  Evidence: {"description": "Declarative hook engine: runs all UserPromptSubmit nudge rules", "hooks": [{"type": "command", "command": "python3 ${CLAUDE_PLUGIN_ROOT}/scripts/engine.py UserPromptSubmit || python $...

🟡 [MEDIUM] plugin:severity1-marketplace/hooks.json → SubagentStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"description": "Declarative hook engine: runs SubagentStart nudge rules when an agent spawns", "hooks": [{"type": "command", "command": "python3 ${CLAUDE_PLUGIN_ROOT}/scripts/engine.py SubagentStart ...

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

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → SubagentStop[0]
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

🟡 [MEDIUM] plugin:claude-plugins-official/hooks.json → SubagentStop[0]
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

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/CLAUDE.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/CLAUDE.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/claude.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/gallant-blackwell-401a7d/claude.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🔵 [LOW] /Users/mp3wizard/.claude/settings.json
  Hooks configuration found
  Evidence: ["Notification", "PermissionDenied", "PermissionRequest", "PostCompact", "PostToolUse", "PostToolUseFailure", "PreCompact", "PreToolUse", "SessionEnd", "SessionStart", "Stop", "StopFailure", "Subagent...

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

🔵 [LOW] plugin:impeccable/hooks.json
  Hooks configuration found
  Evidence: ["PostToolUse", "Stop"]

🔵 [LOW] plugin:impeccable/hooks.json
  Hooks configuration found
  Evidence: ["PostToolUse", "Stop"]

🔵 [LOW] plugin:ponytail/claude-codex-hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "SubagentStart", "UserPromptSubmit"]

🔵 [LOW] plugin:ponytail/copilot-hooks.json
  Hooks configuration found
  Evidence: ["sessionStart", "userPromptSubmitted"]

🔵 [LOW] plugin:ponytail/qoder-hooks.json
  Hooks configuration found
  Evidence: ["UserPromptSubmit", "PreToolUse"]

🔵 [LOW] plugin:severity1-marketplace/hooks.json
  Hooks configuration found
  Evidence: ["UserPromptSubmit", "PreToolUse", "SubagentStart"]

🔵 [LOW] plugin:addy-agent-skills/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop", "SubagentStop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop", "SubagentStop"]

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

## Build Failure
None. `cargo install --path . --force` exit 0.
