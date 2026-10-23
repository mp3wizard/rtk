# Security Audit — 2026-10-23

## Summary
- Issues found: 9 (vuln advisories in lockfiles) | Auto-fixed: 6 | Unresolved: 3
- Status: ISSUES REMAINING
- Upstream merged: 299 commits (origin/develop → worktree branch claude/relaxed-hellman-293f95, merge commit c9daed51)
- Target: worktree relaxed-hellman-293f95, git HEAD c9daed51 (pre-fix base)

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (via ureq 2.12.1) | RUSTSEC-2026-0346 (CVSS 4.0), RUSTSEC-2026-0347 (2.3), RUSTSEC-2026-0348 (3.7), RUSTSEC-2026-0349 (3.7) | 0.23.45 → 0.23.46 in Cargo.lock and proxy/Cargo.lock |
| 2 | rustls-webpki | RUSTSEC-2026-0382 | 0.103.15 → 0.103.16 in both lockfiles |
| 3 | webpki-roots | RUSTSEC-2026-0381 | 1.0.6 → 1.0.10 (root); 1.0.9 → 1.0.10 (proxy) |

Lockfile-only change (cargo update -p ... --precise). Cargo.toml unchanged.

## Unresolved Issues
| # | Component | Advisory | Reason |
|---|-----------|----------|--------|
| 1 | lru 0.12.5 (via ratatui 0.29.0) | RUSTSEC-2026-0002 / GHSA-rhfx-m35p-ff5j (CVSS 2.7) | Fix needs lru 0.16.3; ratatui 0.29 pins lru ^0.12. Requires ratatui upgrade — not auto-applied. |
| 2 | lru 0.12.5 (via ratatui 0.29.0) | RUSTSEC-2026-0253 | Fix needs lru 0.18.2; same blocker as #1. |
| 3 | paste 1.0.15 (via ratatui 0.29.0) | RUSTSEC-2024-0436 | Unmaintained advisory; no fixed version. Clears with ratatui upgrade. |

## Triaged Findings (no fix required, needs human review)
| # | Tool | Finding | Disposition |
|---|------|---------|-------------|
| 1 | gitleaks (2137 commits) | 38 raw hits; rules stripe-access-token / generic-api-key; in Security reports/*.md, SECURITY.md, tests/guard_integration_test.rs, src/cmds/cloud/aws_cmd.rs, scripts/benchmark/cloud-init.yaml | Same pattern as prior triage (2026-10-01). Secret values NOT reproduced here. Not verified live. Confirm test/example strings only; rotate if any is real. |
| 2 | gitleaks (working tree, --no-git) | 36 hits in the same files | Same as #1 — current-tree occurrences of the same strings. |
| 3 | TruffleHog (git history) | 3 unverified results, detector JDBC | 0 verified. Not live-verified. Review file/commit manually. |
| 4 | Bandit | 9 issues, all Low severity (B404/B603/B607 subprocess use) | Expected for hook/benchmark subprocess code. No Medium/High. |
| 5 | skill-audit heuristics | ship 90/100 (chmod, bash complexity); performance privilege-escalation flag (sudo dtrace in docs); security-guardian CRITICAL (destructive-command examples in attack docs); rtk-tdd 35/100 | Heuristic hits on documentation/example text in the repo's own skills. Likely false positive; not changed. |

## Scanner Coverage
| Tool | Result |
|------|--------|
| Gitleaks | ran; see Triaged #1–2 |
| Bandit | ran; 9 Low |
| Semgrep (owasp-top-ten, secrets, python) | ran; 0 findings. OWASP config include set does not cover Rust (.rs) — SAST gap for Rust. |
| Trivy fs | ran; 1 LOW |
| TruffleHog (git) | ran; 0 verified, 3 unverified |
| OSV-Scanner | ran before fix: 15 entries; after fix: 3 (see Unresolved) |
| config-audit | ran; LOW informational only |
| skill-audit | ran over 12 SKILL.md; see Triaged #5 |
| mcp-exfil-scan | ran; 0/100 CLEAN |
| CodeQL | skipped (not run in this scheduled pass) |
| mcp-scan, skillspector LLM mode | OPT-IN, not run (no consent in scheduled run) |
| skillspector --no-llm | not run |

## Build Status
- cargo install --path . --force: success (release, ~2m). Installed to ~/.cargo/bin/rtk (12M), version 0.50.0.
- Wrapper ~/.local/bin/rtk rewritten (exec → ~/.cargo/bin/rtk). Binary not executed (sandbox SIGKILL rule).
- Note: install replaced prior rtk built from worktree unruffled-hamilton-c55464.

## Raw Scanner Output
Secret values redacted/omitted. Gitleaks and TruffleHog summarized above (rule, file, line only). Other tool output follows verbatim (ANSI stripped).

### Gitleaks (summary, values omitted)
Log lines: 3; leaks found: 38 (history scan), see Triaged #1.

### Bandit
```
Run started:2026-10-23 02:21:45.040486+00:00

Test results:
>> Issue: [B404:blacklist] Consider possible security implications associated with the subprocess module.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/blacklists/blacklist_imports.html#b404-import-subprocess
   Location: ./hooks/hermes/rtk-rewrite/__init__.py:8:0
7	import shutil
8	import subprocess
9	import sys

--------------------------------------------------
>> Issue: [B607:start_process_with_partial_path] Starting a process with a partial executable path
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b607_start_process_with_partial_path.html
   Location: ./hooks/hermes/rtk-rewrite/__init__.py:51:21
50	        try:
51	            result = subprocess.run(
52	                ["rtk", "rewrite", command],
53	                shell=False,
54	                timeout=2,
55	                capture_output=True,
56	                text=True,
57	            )
58	        except subprocess.TimeoutExpired:

--------------------------------------------------
>> Issue: [B603:subprocess_without_shell_equals_true] subprocess call - check for execution of untrusted input.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b603_subprocess_without_shell_equals_true.html
   Location: ./hooks/hermes/rtk-rewrite/__init__.py:53:21
52	                ["rtk", "rewrite", command],
53	                shell=False,
54	                timeout=2,
55	                capture_output=True,
56	                text=True,
57	            )
58	        except subprocess.TimeoutExpired:
59	            _warn("rtk rewrite timed out")
60	            return

--------------------------------------------------
>> Issue: [B404:blacklist] Consider possible security implications associated with the subprocess module.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/blacklists/blacklist_imports.html#b404-import-subprocess
   Location: ./hooks/hermes/tests/test_rtk_rewrite_plugin.py:6:0
5	import stat
6	import subprocess
7	import sys

--------------------------------------------------
>> Issue: [B607:start_process_with_partial_path] Starting a process with a partial executable path
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b607_start_process_with_partial_path.html
   Location: ./hooks/hermes/tests/test_rtk_rewrite_plugin.py:318:21
317	
318	            result = subprocess.run(
319	                ["cargo", "run", "--quiet", "--", "init", "--agent", "hermes"],
320	                cwd=repo_root,
321	                env=env,
322	                capture_output=True,
323	                text=True,
324	                timeout=300,
325	            )
326	

--------------------------------------------------
>> Issue: [B603:subprocess_without_shell_equals_true] subprocess call - check for execution of untrusted input.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b603_subprocess_without_shell_equals_true.html
   Location: ./hooks/hermes/tests/test_rtk_rewrite_plugin.py:318:21
317	
318	            result = subprocess.run(
319	                ["cargo", "run", "--quiet", "--", "init", "--agent", "hermes"],
320	                cwd=repo_root,
321	                env=env,
322	                capture_output=True,
323	                text=True,
324	                timeout=300,
325	            )
326	

--------------------------------------------------
>> Issue: [B404:blacklist] Consider possible security implications associated with the subprocess module.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/blacklists/blacklist_imports.html#b404-import-subprocess
   Location: ./scripts/benchmark-sessions/lib/runner.py:5:0
4	import os
5	import subprocess
6	import tempfile

--------------------------------------------------
>> Issue: [B607:start_process_with_partial_path] Starting a process with a partial executable path
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b607_start_process_with_partial_path.html
   Location: ./scripts/benchmark-sessions/lib/runner.py:28:8
27	    try:
28	        subprocess.run(
29	            ["tar", "czf", tarball, "-C", str(source_dir), "."],
30	            check=True,
31	        )
32	    except Exception:

--------------------------------------------------
>> Issue: [B603:subprocess_without_shell_equals_true] subprocess call - check for execution of untrusted input.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   More Info: https://bandit.readthedocs.io/en/1.9.4/plugins/b603_subprocess_without_shell_equals_true.html
   Location: ./scripts/benchmark-sessions/lib/runner.py:28:8
27	    try:
28	        subprocess.run(
29	            ["tar", "czf", tarball, "-C", str(source_dir), "."],
30	            check=True,
31	        )
32	    except Exception:

--------------------------------------------------

Code scanned:
	Total lines of code: 467
	Total lines skipped (#nosec): 0
	Total potential issues skipped due to specifically being disabled (e.g., #nosec BXXX): 0

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
```

### Semgrep — OWASP
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 12 files tracked by git with 560 Code rules:
                                                                                                                        
  Language      Rules   Files          Origin      Rules                                                                
 ─────────────────────────────        ───────────────────                                                               
  <multilang>       5      12          Community     560                                                                
  ts               71       9                                                                                           
  python          147       2                                                                                           
  ruby             43       1                                                                                           
                                                                                                                        
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 266
 • Targets scanned: 12
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Not matching --include patterns: 586
   ◦ Files matching .semgrepignore patterns: 284
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 266 rules on 12 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### Semgrep — secrets
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
   ◦ Files larger than  files 0.3 MB: 1
   ◦ Files matching .semgrepignore patterns: 284
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 45 rules on 597 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### Semgrep — python
```
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 2 files tracked by git with 151 Code rules:
  Scanning 2 files with 151 python rules.
                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 151
 • Targets scanned: 2
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Not matching --include patterns: 596
   ◦ Files matching .semgrepignore patterns: 284
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 151 rules on 2 files: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)
```

### Trivy
```
2026-10-23T09:17:33+07:00	INFO	[vuln] Vulnerability scanning is enabled
2026-10-23T09:17:33+07:00	INFO	[secret] Secret scanning is enabled
2026-10-23T09:17:33+07:00	INFO	[secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2026-10-23T09:17:33+07:00	INFO	[secret] Please see https://trivy.dev/docs/v0.75/guide/scanner/secret#recommendation for faster secret detection
2026-10-23T09:17:33+07:00	INFO	Number of language-specific files	num=9
2026-10-23T09:17:33+07:00	INFO	[cargo] Detecting vulnerabilities...
2026-10-23T09:17:33+07:00	INFO	[pom] Detecting vulnerabilities...

Report Summary

┌───────────────────────────────────────────────────────────┬───────┬─────────────────┬─────────┐
│                          Target                           │ Type  │ Vulnerabilities │ Secrets │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ Cargo.lock                                                │ cargo │        1        │    -    │
├───────────────────────────────────────────────────────────┼───────┼─────────────────┼─────────┤
│ proxy/Cargo.lock                                          │ cargo │        0        │    -    │
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


Cargo.lock (cargo)
==================
Total: 1 (UNKNOWN: 0, LOW: 1, MEDIUM: 0, HIGH: 0, CRITICAL: 0)

┌─────────┬─────────────────────┬──────────┬────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────────┐
│ Library │    Vulnerability    │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                            │
├─────────┼─────────────────────┼──────────┼────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ lru     │ GHSA-rhfx-m35p-ff5j │ LOW      │ fixed  │ 0.12.5            │ 0.16.3        │ `IterMut` violates Stacked Borrows by invalidating internal │
│         │                     │          │        │                   │               │ pointer                                                     │
│         │                     │          │        │                   │               │ https://github.com/advisories/GHSA-rhfx-m35p-ff5j           │
└─────────┴─────────────────────┴──────────┴────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────────┘
```

### OSV-Scanner (before fix)
```
Scanning dir .
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/proxy/Cargo.lock file and found 319 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multifail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/child-b/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/child-a/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/Cargo.lock file and found 255 packages
End status: 154 dirs visited, 1039 inodes visited, 9 Extract calls, 32.792166ms elapsed, 32.793ms wall time
Warning: enricher transitivedependency/pomxml may be risky when run on untrusted artifacts. Please ensure you trust the source code and artifacts.

Total 8 packages affected by 15 known vulnerabilities (0 Critical, 0 High, 2 Medium, 7 Low, 6 Unknown) from 1 ecosystem.
14 vulnerabilities can be fixed.

+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE       | VERSION  | FIXED VERSION | SOURCE           |
+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
| https://osv.dev/RUSTSEC-2026-0002   | 2.7  | crates.io | lru           | 0.12.5   | 0.16.3        | Cargo.lock       |
| https://osv.dev/GHSA-rhfx-m35p-ff5j |      |           |               |          |               |                  |
| https://osv.dev/RUSTSEC-2026-0253   |      | crates.io | lru           | 0.12.5   | 0.18.2        | Cargo.lock       |
| https://osv.dev/RUSTSEC-2024-0436   |      | crates.io | paste         | 1.0.15   | --            | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0346   | 4.0  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0347   | 2.3  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0348   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0349   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0382   |      | crates.io | rustls-webpki | 0.103.15 | 0.103.16      | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0381   |      | crates.io | webpki-roots  | 1.0.6    | 1.0.10        | Cargo.lock       |
| https://osv.dev/RUSTSEC-2026-0346   | 4.0  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0347   | 2.3  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0348   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0349   | 3.7  | crates.io | rustls        | 0.23.45  | 0.23.46       | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0382   |      | crates.io | rustls-webpki | 0.103.15 | 0.103.16      | proxy/Cargo.lock |
| https://osv.dev/RUSTSEC-2026-0381   |      | crates.io | webpki-roots  | 1.0.9    | 1.0.10        | proxy/Cargo.lock |
+-------------------------------------+------+-----------+---------------+----------+---------------+------------------+
```

### OSV-Scanner (after fix)
```
Scanning dir .
Starting filesystem walk for root: /
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/proxy/Cargo.lock file and found 319 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/child-b/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-fail-skeleton/child-a/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multifail-skeleton/pom.xml file and found 1 package
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/child-b/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/tests/fixtures/multi-module-skeleton/child-a/pom.xml file and found 0 packages
Scanned /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95/Cargo.lock file and found 255 packages
End status: 154 dirs visited, 1039 inodes visited, 9 Extract calls, 32.505416ms elapsed, 32.506ms wall time
Warning: enricher transitivedependency/pomxml may be risky when run on untrusted artifacts. Please ensure you trust the source code and artifacts.

Total 2 packages affected by 3 known vulnerabilities (0 Critical, 0 High, 0 Medium, 1 Low, 2 Unknown) from 1 ecosystem.
2 vulnerabilities can be fixed.

+-------------------------------------+------+-----------+---------+---------+---------------+------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE | VERSION | FIXED VERSION | SOURCE     |
+-------------------------------------+------+-----------+---------+---------+---------------+------------+
| https://osv.dev/RUSTSEC-2026-0002   | 2.7  | crates.io | lru     | 0.12.5  | 0.16.3        | Cargo.lock |
| https://osv.dev/GHSA-rhfx-m35p-ff5j |      |           |         |         |               |            |
| https://osv.dev/RUSTSEC-2026-0253   |      | crates.io | lru     | 0.12.5  | 0.18.2        | Cargo.lock |
| https://osv.dev/RUSTSEC-2024-0436   |      | crates.io | paste   | 1.0.15  | --            | Cargo.lock |
+-------------------------------------+------+-----------+---------+---------+---------------+------------+
```

### config-audit
```
============================================================
  Claude Code Security Audit
============================================================

[1/5] Scanning global settings...
[2/5] Scanning installed skills...
[3/5] Scanning installed plugins...
[4/5] Scanning project: .
[5/5] Scanning project-specific settings...

============================================================
  Results
============================================================

Found 178 issue(s):
  CRITICAL: 24
  HIGH: 48
  MEDIUM: 78
  LOW: 28

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

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/.claude/skills/security-scanner/scripts/mcp-exfil-scan.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/bin/bash
# mcp-exfil-scan.sh — MCP Data Exfiltration Detection Scanner v1.0
# Detects data exfiltration risks in MCP servers, skills, and plugins
# Usage: bash mcp-exfil-scan.sh [scan-target-path]
...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/.claude/skills/security-scanner/scripts/skill-audit.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with SSH directory access
  Evidence: #!/bin/bash
# skill-security-test.sh
# Comprehensive automated skill security tester v2.0
# Usage: bash skill-security-test.sh <URL_OR_FILE_PATH>

set -uo pipefail

SKILL_INPUT="$1"
TEST_DIR="/tmp/ski...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/.claude/skills/security-scanner/scripts/config-audit.py
  Data exfiltration: ncat connection combined with SSH directory access
  Evidence: #!/usr/bin/env python3
"""
Claude Code Security Audit Scanner

Scans Claude Code configurations, skills, MCP servers, and project files
for potentially malicious commands, hooks, and data exfiltration...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/.claude/skills/security-scanner/scripts/mcp-exfil-scan.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with .env file access
  Evidence: #!/bin/bash
# mcp-exfil-scan.sh — MCP Data Exfiltration Detection Scanner v1.0
# Detects data exfiltration risks in MCP servers, skills, and plugins
# Usage: bash mcp-exfil-scan.sh [scan-target-path]
...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/.claude/skills/security-scanner/scripts/skill-audit.sh
  Data exfiltration: base64 encoding (possible data obfuscation) combined with SSH directory access
  Evidence: #!/bin/bash
# skill-security-test.sh
# Comprehensive automated skill security tester v2.0
# Usage: bash skill-security-test.sh <URL_OR_FILE_PATH>

set -uo pipefail

SKILL_INPUT="$1"
TEST_DIR="/tmp/ski...

🔴 [CRITICAL] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/.claude/skills/security-scanner/scripts/config-audit.py
  Data exfiltration: ncat connection combined with SSH directory access
  Evidence: #!/usr/bin/env python3
"""
Claude Code Security Audit Scanner

Scans Claude Code configurations, skills, MCP servers, and project files
for potentially malicious commands, hooks, and data exfiltration...

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

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8d392c816966/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8d392c816966/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/d1adba47161a/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/d1adba47161a/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/a0b04d9ddcca/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/a0b04d9ddcca/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/641d925db5a6/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/641d925db5a6/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/f359d43b4060/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/f359d43b4060/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

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

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/a9fd21643fc1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/a9fd21643fc1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/4a5d5524630a/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/4a5d5524630a/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/ea10b213686c/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/ea10b213686c/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8bd00f08f0b0/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8bd00f08f0b0/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8f814f49002f/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/8f814f49002f/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/af59ee3a1051/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/af59ee3a1051/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/51b66d2fd2e1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/51b66d2fd2e1/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/465f09344424/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/465f09344424/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/b8e53f1c05df/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/b8e53f1c05df/skills/hook-development/examples/validate-bash.sh
  Dangerous command: dd raw disk operation
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/c2bdb427efe8/skills/hook-development/examples/validate-bash.sh
  Dangerous command: filesystem format command
  Evidence: #!/bin/bash
# Example PreToolUse hook for validating Bash commands
# This script demonstrates bash command validation patterns

set -euo pipefail

# Read input from stdin
input=$(cat)

# Extract comma...

🟠 [HIGH] plugin:claude-plugins-official/plugin-dev/c2bdb427efe8/skills/hook-development/examples/validate-bash.sh
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

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: chmod 777 (world writable)
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: recursive delete from root
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.10.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: filesystem format command
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: chmod 777 (world writable)
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: recursive delete from root
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

🟠 [HIGH] plugin:claude-code-security-plugins/claude-code-security-plugins/1.9.0/hooks/bash-guard-pretooluse.sh
  Dangerous command: filesystem format command
  Evidence: #!/usr/bin/env bash
# bash-guard-pretooluse.sh — PreToolUse hook (matcher: Bash).
#
# DENIES only a small, deliberately narrow set of CATASTROPHIC, near-zero-
# false-positive commands (root/home wipe...

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

🟠 [HIGH] CLAUDE.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟠 [HIGH] claude.md
  Suspicious command: curl to external URL
  Evidence: # CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy th...

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Notification[2]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" notification"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SessionEnd[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" session-end"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[1]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → Stop[2]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "\"/Applications/Setapp/BlackFlare.app/Contents/MacOS/blackflare-hook\" stop"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → StopFailure[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

🟡 [MEDIUM] /Users/mp3wizard/.claude/settings.json → SubagentStop[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "/bin/sh -c '[ -x \"$HOME/.vibe-island/bin/vibe-island-bridge\" ] && \"$HOME/.vibe-island/bin/vibe-island-bridge\" --source claude; exit 0'"}]}

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

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

🟡 [MEDIUM] plugin:claude-code-security-plugins/hooks.json → SessionStart[0]
  Broad matcher '' — runs on every operation
  Evidence: {"hooks": [{"type": "command", "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/preflight-sessionstart.sh\"", "timeout": 30}]}

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

🟡 [MEDIUM] CLAUDE.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] CLAUDE.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🟡 [MEDIUM] claude.md
  Suspicious instruction: instruction to skip verification
  Evidence: **Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

🟡 [MEDIUM] claude.md
  Suspicious instruction: trust-all instruction
  Evidence: - Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)

🔵 [LOW] /Users/mp3wizard/.claude/settings.json
  Hooks configuration found
  Evidence: ["Notification", "PermissionRequest", "PostCompact", "PostToolUse", "PreCompact", "PreToolUse", "SessionEnd", "SessionStart", "Stop", "StopFailure", "SubagentStart", "SubagentStop", "TeammateIdle", "U...

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
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop", "SubagentStop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop", "SubagentStop"]

🔵 [LOW] plugin:claude-plugins-official/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "UserPromptSubmit", "PostToolUse", "Stop"]

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

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

🔵 [LOW] plugin:claude-code-security-plugins/hooks.json
  Hooks configuration found
  Evidence: ["SessionStart", "PreToolUse"]

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

### mcp-exfil-scan
```
============================================================
  MCP Data Exfiltration Scanner v1.0
============================================================
Scan target: /Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/relaxed-hellman-293f95
Report: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/mcp-exfil-20261023_091742.txt

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

Full report saved to: /Users/mp3wizard/.claude/plugins/cache/claude-code-security-plugins/claude-code-security-plugins/1.11.0/.claude/skills/security-scanner/scripts/../reports/mcp-exfil-20261023_091742.txt
```
