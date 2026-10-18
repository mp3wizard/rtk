# Security Audit — 2026-09-21

## Summary
- Issues found: 1 | Auto-fixed: 1 | Unresolved: 0
- Status: ISSUES FIXED
- Scope: `/Users/mp3wizard/Public/Claude Proxy/rtk/.claude/worktrees/amazing-raman-3115e8`, git HEAD `16e9e1e9` (merge of 87 upstream commits from `origin/develop`)
- Tools run: OSV-Scanner 2.6.0, Trivy 0.74.0, Gitleaks 8.30.1, TruffleHog 3.97.5, Semgrep 1.177.0 (p/rust + p/secrets). Skipped: Bandit (4 stray `.py` files, no Python project), CodeQL (no workflow), mcp-scan / skillspector LLM mode (opt-in, non-interactive run).

## Fixed Issues
| # | Component | Advisory | Change |
|---|-----------|----------|--------|
| 1 | rustls (transitive via ureq 2.12.1) | RUSTSEC-2026-0285 (CVSS 5.3, Medium) | `Cargo.lock` `rustls` 0.23.37 → 0.23.45; `rustls-webpki` 0.103.13 → 0.103.15 pulled in as a required companion. Rescan: OSV "No issues found". |

## Merge Regression Fixed (not a vulnerability)
Conflict in `src/cmds/dotnet/dotnet_cmd.rs` was resolved by accepting upstream's version (per task rule). That reverted the 2026-07-07 quick-xml 0.41 migration (RUSTSEC-2026-0194/0195), breaking the build (`BytesText::unescape` removed). Re-applied `e.unescape()` → `e.decode()` at `src/cmds/dotnet/dotnet_cmd.rs:647`. Verified: `cargo clippy --all-targets` clean, `cargo test --all` 3882 passed / 8 ignored. The quick-xml 0.41 pin in `Cargo.toml` is kept; upstream still uses 0.37.

## Triaged, no action
- **Gitleaks (38 hits, history scan):** all test fixtures or prior audit reports quoting fake tokens (`src/cmds/cloud/aws_cmd.rs` tests, `tests/guard_integration_test.rs` fake Stripe keys, `scripts/benchmark/cloud-init.yaml`, `Security reports/*.md`, `SECURITY.md`). False positives, same set as previous audits.
- **TruffleHog:** 0 verified, 0 unverified secrets.
- **Semgrep (31 findings, informational):** 5 `unsafe-usage` (libc signal/kill in the child-process wrapper), 6 `temp-dir`, 2 `current-exe`, 2 `args`, 1 `args-os` — all expected for a CLI proxy, no injection or data-flow findings.
- **Trivy:** 0 vulnerabilities, 0 secrets, 0 misconfigurations.

## Raw Scanner Output
### OSV-Scanner (before fix)
```
Scanning dir .
Starting filesystem walk for root: /
End status: 123 dirs visited, 770 inodes visited, 8 Extract calls, 31.95875ms elapsed, 31.959ms wall time
Warning: enricher transitivedependency/pomxml may be risky when run on untrusted artifacts. Please ensure you trust the source code and artifacts.

Total 1 package affected by 1 known vulnerability (0 Critical, 0 High, 1 Medium, 0 Low, 0 Unknown) from 1 ecosystem.
1 vulnerability can be fixed.

+-----------------------------------+------+-----------+---------+---------+---------------+------------+
| OSV URL                           | CVSS | ECOSYSTEM | PACKAGE | VERSION | FIXED VERSION | SOURCE     |
+-----------------------------------+------+-----------+---------+---------+---------------+------------+
| https://osv.dev/RUSTSEC-2026-0285 | 5.3  | crates.io | rustls  | 0.23.37 | 0.23.45       | Cargo.lock |
+-----------------------------------+------+-----------+---------+---------+---------------+------------+
osv exit 1
```
### Trivy
```
2026-09-21T09:12:29+07:00	INFO	[secret] If your scanning is slow, please try '--scanners vuln,misconfig' to disable secret scanning
2026-09-21T09:12:29+07:00	INFO	[secret] Please see https://trivy.dev/docs/v0.74/guide/scanner/secret#recommendation for faster secret detection
2026-09-21T09:12:31+07:00	INFO	Number of language-specific files	num=8
2026-09-21T09:12:31+07:00	INFO	[cargo] Detecting vulnerabilities...
2026-09-21T09:12:31+07:00	INFO	[pom] Detecting vulnerabilities...
2026-09-21T09:12:31+07:00	INFO	Detected config files	num=1

Report Summary

┌───────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┬───────────────────┐
│                          Target                           │    Type    │ Vulnerabilities │ Secrets │ Misconfigurations │
├───────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┼───────────────────┤
│ Cargo.lock                                                │   cargo    │        0        │    -    │         -         │
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

trivy exit 0
```
### Gitleaks (summary; secrets redacted)
```
9:12AM INF 1829 commits scanned.
9:12AM INF scanned ~12681924 bytes (12.68 MB) in 3.72s
9:12AM WRN leaks found: 38
gitleaks exit 1
```
### TruffleHog
```
2026-09-21T09:12:32+07:00	info-0	trufflehog	finished scanning	{"chunks": 20091, "bytes": 14244084, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "2.275316375s", "trufflehog_version": "3.97.5", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
trufflehog exit 0
```
### Semgrep
```
                    
                    
┌──────────────────┐
│ 31 Code Findings │
└──────────────────┘
                                 
    src/cmds/dotnet/dotnet_cmd.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          272┆ std::env::temp_dir().join(format!(
            ⋮┆----------------------------------------
          280┆ std::env::temp_dir().join(format!("rtk_dotnet_testresults_{}", unique_temp_suffix()))
            ⋮┆----------------------------------------
          312┆ std::env::temp_dir().join(format!("rtk_dotnet_format_{}.json", unique_temp_suffix()))
                           
    src/cmds/git/git_cmd.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          6564┆ let tmp = std::env::temp_dir().join("rtk_test_not_a_repo");
                               
    src/cmds/system/find_cmd.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          792┆ let tmp = std::env::temp_dir().to_string_lossy().into_owned();
                          
    src/core/args_utils.rs
     ❱ rust.lang.security.args.args
          ❰❰ Blocking ❱❱
          args should not be used for security operations. From the docs: "The first element is traditionally
          the path of the executable, but it can be set to arbitrary text, and might not even exist. This    
          means this property should not be relied upon for security purposes."                              
          Details: https://sg.run/RADN                                                                       
                                                                                                             
           10┆ let raw_args: Vec<String> = std::env::args().collect();
                         
    src/core/retriever.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          181┆ let base = std::env::temp_dir().join(format!("rtk-test-store-{}", std::process::id()));
                      
    src/core/stream.rs
     ❱ rust.lang.security.unsafe-usage.unsafe-usage
          ❰❰ Blocking ❱❱
          Detected 'unsafe' usage, please audit for secure usage
          Details: https://sg.run/lqgo                          
                                                                
          291┆ unsafe {
          292┆     libc::signal(sig, libc::SIG_DFL);
          293┆     libc::raise(sig);
          294┆ }
            ⋮┆----------------------------------------
          298┆ unsafe {
          299┆     libc::kill(pid as libc::pid_t, sig);
          300┆ }
            ⋮┆----------------------------------------
          310┆ unsafe {
          311┆     libc::kill(pid as libc::pid_t, libc::SIGKILL);
          312┆ }
            ⋮┆----------------------------------------
          320┆ unsafe {
          321┆     libc::signal(sig, libc::SIG_DFL);
          322┆     libc::raise(sig);
          323┆ }
            ⋮┆----------------------------------------
          342┆ unsafe {
          343┆     for sig in [libc::SIGINT, libc::SIGTERM] {
          344┆         let previous = libc::signal(sig, relay as *const () as libc::sighandler_t);
          345┆         if previous == libc::SIG_IGN {
          346┆             libc::signal(sig, libc::SIG_IGN);
          347┆         }
          348┆     }
          349┆ }
            ⋮┆----------------------------------------
          369┆ unsafe {
          370┆     for sig in [libc::SIGINT, libc::SIGTERM] {
          371┆         if libc::signal(sig, libc::SIG_DFL) == libc::SIG_IGN {
          372┆             libc::signal(sig, libc::SIG_IGN);
          373┆         }
          374┆     }
          375┆ }
            ⋮┆----------------------------------------
          399┆ unsafe {
          400┆     libc::signal(sig, libc::SIG_DFL);
          401┆     libc::raise(sig);
          402┆ }
                        
    src/core/tee_file.rs
     ❱ rust.lang.security.unsafe-usage.unsafe-usage
          ❰❰ Blocking ❱❱
          Detected 'unsafe' usage, please audit for secure usage
          Details: https://sg.run/lqgo                          
                                                                
          329┆ let previous = unsafe { libc::umask(0o000) };
            ⋮┆----------------------------------------
          334┆ unsafe { libc::umask(previous) };
                         
    src/core/telemetry.rs
     ❱ rust.lang.security.current-exe.current-exe
          ❰❰ Blocking ❱❱
          current_exe should not be used for security operations. From the docs: "The output of this function
          should not be trusted for anything that might have security implications. Basically, if users can  
          run the executable, they can change the output arbitrarily."                                       
          Details: https://sg.run/AW1B                                                                       
                                                                                                             
          530┆ let exe = match std::env::current_exe() {
                        
    src/core/tracking.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          2051┆ let db_path = env::temp_dir().join(format!(
            ⋮┆----------------------------------------
          2077┆ let db_path = env::temp_dir().join(format!(
            ⋮┆----------------------------------------
          2111┆ let custom_path = env::temp_dir().join("rtk_test_custom.db");
            ⋮┆----------------------------------------
          2136┆ env::temp_dir().join(format!("rtk_test_schema_version_{}.db", std::process::id()));
            ⋮┆----------------------------------------
          2174┆ let db_path = std::env::temp_dir().join(format!(
                     
    src/core/utils.rs
     ❱ rust.lang.security.unsafe-usage.unsafe-usage
          ❰❰ Blocking ❱❱
          Detected 'unsafe' usage, please audit for secure usage
          Details: https://sg.run/lqgo                          
                                                                
          875┆ let cp = unsafe {
          876┆     let console = windows_sys::Win32::System::Console::GetConsoleOutputCP();
          877┆     if console != 0 {
          878┆         console
          879┆     } else {
          880┆         windows_sys::Win32::Globalization::GetACP()
          881┆     }
          882┆ };
                            
    src/discover/registry.rs
     ❱ rust.lang.security.current-exe.current-exe
          ❰❰ Blocking ❱❱
          current_exe should not be used for security operations. From the docs: "The output of this function
          should not be trusted for anything that might have security implications. Basically, if users can  
          run the executable, they can change the output arbitrarily."                                       
          Details: https://sg.run/AW1B                                                                       
                                                                                                             
          3915┆ let test_mtime = std::env::current_exe()
                         
    src/hooks/hook_cmd.rs
     ❱ rust.lang.security.temp-dir.temp-dir
          ❰❰ Blocking ❱❱
          temp_dir should not be used for security operations. From the docs: 'The temporary directory may be
          shared among users, or between processes with different privileges; thus, the creation of any files
          or directories in the temporary directory must use a secure method to create a uniquely named file.
          Creating a file or directory with a fixed or predictable name may result in “insecure temporary    
          file” security vulnerabilities.'                                                                   
          Details: https://sg.run/qzEO                                                                       
                                                                                                             
          2386┆ let tmp = std::env::temp_dir().join("rtk-test-audit");
               
    src/main.rs
     ❱ rust.lang.security.args.args
          ❰❰ Blocking ❱❱
          args should not be used for security operations. From the docs: "The first element is traditionally
          the path of the executable, but it can be set to arbitrary text, and might not even exist. This    
          means this property should not be relied upon for security purposes."                              
          Details: https://sg.run/RADN                                                                       
                                                                                                             
          1548┆ let args: Vec<String> = std::env::args().skip(1).collect();
   
     ❱ rust.lang.security.unsafe-usage.unsafe-usage
          ❰❰ Blocking ❱❱
          Detected 'unsafe' usage, please audit for secure usage
          Details: https://sg.run/lqgo                          
                                                                
          1867┆ unsafe {
          1868┆     libc::signal(libc::SIGPIPE, libc::SIG_DFL);
          1869┆ }
   
     ❱ rust.lang.security.args-os.args-os
          ❰❰ Blocking ❱❱
          args_os should not be used for security operations. From the docs: "The first element is         
          traditionally the path of the executable, but it can be set to arbitrary text, and might not even
          exist. This means this property should not be relied upon for security purposes."                
          Details: https://sg.run/G6k6                                                                     
                                                                                                           
          1935┆ let cli = match Cli::try_parse_from(std::env::args_os()) {
   
     ❱ rust.lang.security.unsafe-usage.unsafe-usage
          ❰❰ Blocking ❱❱
          Detected 'unsafe' usage, please audit for secure usage
          Details: https://sg.run/lqgo                          
                                                                
          3061┆ unsafe {
          3062┆     libc::kill(pid as libc::pid_t, libc::SIGTERM);
          3063┆     libc::waitpid(pid as libc::pid_t, std::ptr::null_mut(), 0);
          3064┆ }
            ⋮┆----------------------------------------
          3067┆ unsafe {
          3068┆     libc::signal(sig, libc::SIG_DFL);
          3069┆     libc::raise(sig);
          3070┆ }
            ⋮┆----------------------------------------
          3073┆ unsafe {
          3074┆     libc::signal(
          3075┆         libc::SIGINT,
          3076┆         handle_signal as *const () as libc::sighandler_t,
          3077┆     );
          3078┆     libc::signal(
          3079┆         libc::SIGTERM,
          3080┆         handle_signal as *const () as libc::sighandler_t,
          3081┆     );
          3082┆ }

```
