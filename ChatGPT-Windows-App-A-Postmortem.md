# Windows Store app repeatedly becomes unlaunchable after successful recovery and after two clean reinstall methods

> This issue concerns the **Codex/ChatGPT Windows desktop application distributed through Microsoft Store**, package `OpenAI.Codex`. It is not a Codex CLI installation problem.

## Document status — updated 2026-08-20

This document preserves the original critical investigation of Store build `26.721.11231.0`. That build's complete startup/bootstrap failure is historical evidence, not the currently installed package state.

The package observed running on 2026-08-20 is `OpenAI.Codex_26.814.5517.0`. It launches, but later builds have continued to exhibit crashes, freezes, duplicated prompts, failed steering, Store update failures, abnormal idle resource use, and broken local-task archiving.

Current follow-up:

- [Codex Windows task archiving fails and completed tasks accumulate in the sidebar](./Codex-Windows-Task-Archive-Failure-20260819.md)

Public tracking for the archive defect is now consolidated in [openai/codex#39492](https://github.com/openai/codex/issues/39492). Phil's independent report, [#39638](https://github.com/openai/codex/issues/39638), added the recurring-automation accumulation and false-success impact; that evidence was transferred to `#39492` before `#39638` was closed as a duplicate. Related reports [#39600](https://github.com/openai/codex/issues/39600), [#39239](https://github.com/openai/codex/issues/39239), and [#39130](https://github.com/openai/codex/issues/39130) provide strong reproduction evidence implicating inconsistent handling of Windows extended-length `\\?\` rollout paths. OpenAI has not yet publicly confirmed that as the official root cause.

## Public tracker and related reports

This report is intended for the official public Codex issue tracker:

- [OpenAI Codex issues](https://github.com/openai/codex/issues)
- [Create a new issue](https://github.com/openai/codex/issues/new)

Related Windows desktop reports showing that startup and reliability failures are not isolated:

- [#25489 — Codex Windows app will not launch after a clean reinstall](https://github.com/openai/codex/issues/25489)
- [#19352 — Windows Codex app blank/startup runtime failure](https://github.com/openai/codex/issues/19352)
- [#19437 — Windows app blank/spinner at startup](https://github.com/openai/codex/issues/19437)
- [#19770 — Windows desktop app white screen on startup](https://github.com/openai/codex/issues/19770)
- [#26624 — Windows desktop project chats missing while history remains available elsewhere](https://github.com/openai/codex/issues/26624)
- [#33483 — Windows app instability after Microsoft Store update](https://github.com/openai/codex/issues/33483)
- [#13993 — Request for a standalone Windows installer](https://github.com/openai/codex/issues/13993)

OpenAI’s product announcement confirms that the Codex app became available on Windows:

- [Introducing the Codex app](https://openai.com/index/introducing-the-codex-app/)

Repository follow-up incident:

- [Codex Windows task archiving fails and completed tasks accumulate in the sidebar](./Codex-Windows-Task-Archive-Failure-20260819.md) — reproduced on package `26.814.5517.0`; a further Codex crash occurred while the report was being prepared.

## Severity

**Critical / blocker**

The application repeatedly becomes completely unlaunchable, reports no actionable error, can leave no useful current Windows crash record, survives state isolation, survives forced uninstall/reinstall, and now also survives a separate reinstall performed directly through the Microsoft Store UI.

The user already spent approximately two days preserving and reconstructing work after the first failure. The replacement build initially worked, allowed recovery, and then became unlaunchable again.

## Original affected package

```text
Package name: OpenAI.Codex
Store product ID: 9PLM9XGG6VKS
Package version: 26.721.11231.0
Package status reported by Windows: Ok
Architecture: x64
AUMID: OpenAI.Codex_2p2nqsd0c76g0!App
Entry point: Windows.FullTrustApplication
Executable: app\ChatGPT.exe
```

Installed package path:

```text
C:\Program Files\WindowsApps\OpenAI.Codex_26.721.11231.0_x64__2p2nqsd0c76g0\app\ChatGPT.exe
```

Executable metadata previously verified:

```text
Length: 4,139,008 bytes
ProductVersion: 150.0.7871.128
FileVersion: 150.0.7871.128
```

## Operating system

```text
Windows 10
Build 19045.7548
x64
```

## User-facing failure

Launching the Store app produces the following behavior:

1. Launch begins.
2. A black window or black bar appears briefly.
3. The black area disappears.
4. No usable application window remains.
5. No ChatGPT, Codex, or OpenAI process remains running.
6. Windows displays:

```text
The application is exiting and cannot service this request
```

The error dialog identifies the packaged executable:

```text
C:\Program Files\WindowsApps\OpenAI.Codex_26.721.11231.0_x64__2p2nqsd0c76g0\app\ChatGPT.exe
```

## Reproduction after two independent reinstall paths

### Reinstall path 1: forced uninstall/reinstall through `winget`

Uninstall executed in Administrator PowerShell:

```powershell
winget uninstall --id 9PLM9XGG6VKS --source msstore --exact --accept-source-agreements
```

Result:

```text
Successfully uninstalled
```

Reinstall executed from a regular, non-elevated PowerShell session:

```powershell
winget install --id 9PLM9XGG6VKS --source msstore --exact --accept-source-agreements --accept-package-agreements
```

Result:

```text
Successfully installed
```

Observed after reinstall:

- same package build,
- package reports `Status: Ok`,
- brief black startup area,
- immediate exit,
- same “cannot service this request” dialog.

### Reinstall path 2: direct Microsoft Store UI reinstall

The app was then uninstalled and reinstalled through the Microsoft Store application itself rather than through `winget`.

Observed after direct Store reinstall:

- Store installation completed,
- launch began,
- black startup area appeared briefly,
- application exited,
- no usable process remained,
- the exact same error dialog appeared,
- the package remained build `26.721.11231.0`.

This second reproduction rules out `winget` as the cause. Both installation paths retrieve or register a package that fails in the same way.

## Earlier failing build

The earlier package was:

```text
26.721.4979.0
```

That build produced AppModel/lifecycle failures including:

```text
AppModel Runtime event 208
0x8000001A
```

The newer `26.721.11231.0` build initially launched and appeared to fix the problem. It allowed recovery of the user’s projects and sidebar state. It then became unlaunchable again.

## State-isolation tests already completed

The failure continued after temporarily moving aside each of the following, one at a time:

```text
%USERPROFILE%\.codex
%LOCALAPPDATA%\OpenAI\Codex
%LOCALAPPDATA%\Packages\OpenAI.Codex_2p2nqsd0c76g0
```

Each folder was restored afterward.

Observed in every test:

- black startup area,
- no main window,
- no surviving app process,
- same launch failure.

The restored Codex state, OpenAI runtime directory, and AppX user-data container are therefore not sufficient explanations for the immediate startup failure.

## Package and registration checks

The package reports:

```text
Name: OpenAI.Codex
Version: 26.721.11231.0
Status: Ok
SignatureKind: Store
```

The executable exists.

The manifest identifies:

```text
Application ID: App
Executable: app/ChatGPT.exe
EntryPoint: Windows.FullTrustApplication
```

The registered AUMID is:

```text
OpenAI.Codex_2p2nqsd0c76g0!App
```

Launching through the AUMID reproduces the failure:

```powershell
explorer.exe shell:AppsFolder\OpenAI.Codex_2p2nqsd0c76g0!App
```

## Windows integrity checks

The operating system and storage were checked extensively.

### DISM

```text
DISM /Online /Cleanup-Image /CheckHealth
No component store corruption detected.

DISM /Online /Cleanup-Image /ScanHealth
No component store corruption detected.
```

### SFC

After repair of an unrelated Default-profile OneDrive shortcut:

```text
Windows Resource Protection did not find any integrity violations.
```

### CHKDSK

```text
Windows has scanned the file system and found no problems.
0 KB in bad sectors.
```

### WindowsApps root

The root owner was restored to:

```text
NT SERVICE\TrustedInstaller
```

No explicit `Everyone` ACL remained on the WindowsApps root.

## Windows diagnostics checked

The failed launch produced no useful current event in the relevant time window from:

```text
Application
Microsoft-Windows-AppModel-Runtime/Admin
Microsoft-Windows-TWinUI/Operational
Microsoft-Windows-AppXDeploymentServer/Operational
Microsoft-Windows-Shell-Core/Operational
Microsoft-Windows-CodeIntegrity/Operational
Microsoft-Windows-Windows Defender/Operational
Microsoft-Windows-AppLocker/Packaged app-Deployment
Microsoft-Windows-AppLocker/Packaged app-Execution
```

No current Windows Error Reporting record was created for the silent launch failure.

After launch, this returned no matching process:

```powershell
Get-Process |
Where-Object { $_.ProcessName -match 'ChatGPT|Codex|OpenAI' }
```

## Sandbox/runtime defect observed

The Codex sandbox log repeatedly reports:

```text
hide users: failed to hide current user profile dir (C:\Users\Default):
SetFileAttributesW failed for C:\Users\Default: 5 (Access is denied.)
```

It also reports:

```text
junction: failed to create C:\Users\Default\.codex\.sandbox\cwd:
Access is denied. (os error 5)
```

The actual signed-in profile environment and registry mapping point to the real user profile, not `C:\Users\Default`.

These warnings were also present during a period when Codex was functioning, so they are not proven to be the direct startup cause. They nevertheless show an incorrect internal profile resolution and should be investigated.

## Recovery burden and continuity risk

The initial failure forced preservation and reconstruction work involving:

```text
More than 262,000 files
Approximately 12.5 GB
148 exported conversations verified
98 older conversations retrieved
3 archived conversations retrieved
261 preserved threads merged
24 workspace roots recovered
22 project-order entries recovered
10 pinned threads recovered
SQLite integrity verified
```

The post-recovery backup completed with zero mismatches and zero failed files.

The recovered application then failed again.

This is a major continuity risk for a developer application whose UI depends on local indexing of cloud-backed threads, project assignments, workspace roots, ordering, and pins.

## Public evidence

The sanitized evidence package is available here:

- [Download the public evidence ZIP](./evidence/ChatGPT-Codex-Public-Evidence-SANITIZED-20260730.zip)
- [View the repository](https://github.com/phillipnhenry/chatgpt-windows-app-postmortem)

The ZIP contains sanitized logs, launch traces, screenshots, and supporting evidence. Raw logs should not be posted publicly because they can contain Windows usernames, local paths, project names, account SIDs, environment variables, and thread identifiers.
## Why Store distribution should be suspended temporarily

The Store package should be temporarily delisted or distribution-paused because:

1. The application repeatedly becomes completely unlaunchable.
2. Windows reports the package as healthy while it is unusable.
3. A forced Store uninstall/reinstall does not repair it.
4. A second reinstall through the Microsoft Store UI does not repair it.
5. The failure produces almost no actionable diagnostics.
6. A replacement build initially worked and then failed again.
7. The app’s local indexing model creates serious continuity and apparent-data-loss risk.
8. Recovery requires expertise and effort far beyond what a Store customer should need.
9. Similar Windows startup and reliability failures have already been reported publicly in the same tracker.
10. Microsoft Store remains the primary distribution path, while a standalone installer has separately been requested.

## Requested engineering action

Please provide:

1. A reproducible internal bug ID.
2. Confirmation of the affected build range.
3. A startup diagnostic mode that writes logs before UI initialization.
4. A deterministic crash report or WER record for early startup exits.
5. An explanation for the incorrect `C:\Users\Default` sandbox resolution.
6. A safe and supported local-state recovery procedure.
7. A fixed package version.
8. Sustained testing of clean install, upgrade, reinstall, reset, interrupted update, large workspace state, and recovered sidebar state.
9. A temporary Store distribution pause until a verified package is available.

## Requested support response

Please do not close this with generic advice to reset, reinstall, run SFC, run DISM, use the browser, or use the CLI.

Those actions have already been completed or do not address the defective Windows package.

A useful response should identify:

```text
Engineering bug ID
Affected versions
Root cause
Fixed version
Safe migration/recovery steps
Store replacement or delisting decision
```

## Final statement

A user should not have to spend nearly two days preserving and reconstructing professional work, successfully restore the application, and then watch the replacement Store build become unlaunchable again.

The direct Microsoft Store reinstall has now reproduced the exact same failure as the forced `winget` reinstall. This is compelling evidence that the defect is in the distributed application package, its startup/runtime behavior, or its interaction with supported Windows environments—not merely in one installation command or one recoverable cache directory.

**Requested resolution: temporarily remove or suspend the Microsoft Store package until OpenAI ships a verified Windows build with reliable startup, migration, diagnostics, and recovery behavior.**


---

## Major update: clean-profile test identifies the sandbox setup helper

A separate Windows account named `ChatGPTTest` was used to isolate the failure from the primary Windows profile and recovered Codex data.

The package was registered successfully under that account. The app progressed farther than it did under the primary account:

1. ChatGPT displayed a loading spinner.
2. A tray icon appeared.
3. The app requested one-time permission to work on the computer.
4. Authorization from the primary account was required.
5. The first-run sandbox setup helper launched.
6. The helper failed with the same native Windows error.

The clean-profile first-run screen displayed:

```text
Finish Windows setup

ChatGPT needs a one-time permission to work on your computer
```

The failing executable was:

```text
C:\Program Files\WindowsApps\OpenAI.Codex_26.721.11231.0_x64__2p2nqsd0c76g0\app\resources\codex-windows-sandbox-setup.exe
```

The dialog reported:

```text
The application is exiting and cannot service this request
```

This clean-profile test rules out the following as sufficient explanations:

```text
The primary Windows profile
The recovered .codex data
The primary account's Electron profile
The primary account's OpenAI runtime folder
The primary account's AppX user-data container
The restored chat/sidebar/project state
```

The defect is machine-wide or inherent to Store build `26.721.11231.0` and its bootstrap/sandbox setup path.

---

## Major update: fresh Procmon trace proves pre-runtime bootstrap exit

A new Process Monitor trace was started from zero and captured only the current launch sequence.

Two separate `ChatGPT.exe` launches showed the same behavior.

### Launch attempt 1

```text
Process start: 5:50:42.2525421 PM
Process exit:  5:50:42.5338560 PM
Elapsed:       approximately 0.281 seconds
```

Loaded images:

```text
ChatGPT.exe
ntdll.dll
```

CPU use:

```text
User time:   0.0000000 seconds
Kernel time: 0.0000000 seconds
```

Exit:

```text
Exit Status: -1073741823
Hex:         0xC0000001
Meaning:     STATUS_UNSUCCESSFUL
```

### Launch attempt 2

```text
Process start: 5:50:51.6888074 PM
Process exit:  5:50:51.9515156 PM
Elapsed:       approximately 0.263 seconds
```

Loaded images:

```text
ChatGPT.exe
ntdll.dll
```

CPU use:

```text
User time:   0.0000000 seconds
Kernel time: 0.0000000 seconds
```

Exit:

```text
Exit Status: -1073741823
Hex:         0xC0000001
Meaning:     STATUS_UNSUCCESSFUL
```

This is not normal Electron or Chromium startup followed by a crash. The process exits before loading the normal application runtime and before creating the usual Electron profile folders.

ProcDump also confirmed:

```text
Process Exit
Exit Code 0xc0000001
Dump count not reached
```

No exception was available to catch, and the process disappeared before a termination dump could be written.

---

## Dependency inspection

`dumpbin /DEPENDENTS` for `ChatGPT.exe` reported:

```text
chrome_elf.dll
VERSION.dll
KERNEL32.dll
ntdll.dll
```

The bundled `chrome_elf.dll` exists and itself depends only on standard Windows libraries.

`dumpbin /DEPENDENTS` for `codex-windows-sandbox-setup.exe` reported only Windows system libraries:

```text
KERNEL32.dll
advapi32.dll
oleaut32.dll
bcryptprimitives.dll
secur32.dll
bcrypt.dll
crypt32.dll
ole32.dll
user32.dll
ntdll.dll
ws2_32.dll
api-ms-win-core-synch-l1-2-0.dll
fwpuclnt.dll
netapi32.dll
```

The helper's PE headers report:

```text
Machine: x64
Operating system version: 6.00
Subsystem version: 6.00
```

There is no obvious missing third-party DLL or declared Windows-version incompatibility.

---

## CPU compatibility context

The system CPU is:

```text
AMD A8-5500 APU with Radeon HD Graphics
```

Supported:

```text
SSE
SSE2
SSE3
SSSE3
SSE4.1
SSE4.2
AVX
FMA
F16C
CMPXCHG16B
x86-64-v2
```

Not supported:

```text
AVX2
BMI2
MOVBE
x86-64-v3
```

This is a legitimate compatibility concern for a newly rebuilt Chromium binary, but it is not proven to be the direct cause because the observed failure is `0xC0000001`, not an illegal-instruction exception, and this same Store build previously ran temporarily on this machine.

---

## Suspected recurrence trigger: multiple simultaneous chats/tasks

The last two recurrences happened while two or three chats/tasks were active at the same time.

The PC remained responsive and system performance was monitored.

This does not prove causation, but OpenAI should specifically test:

```text
Two simultaneous chats
Three simultaneous chats
Concurrent tool execution
Multiple active workspace roots
Forced close with several active tasks
Restart after interrupted concurrent tasks
Sandbox setup after multi-task activity
```

---

## Updated conclusion

The latest evidence changes the diagnosis materially.

The clean-profile test proves the failure is not confined to the primary Windows account or recovered user state.

The fresh Procmon trace proves that `ChatGPT.exe` exits before normal application initialization.

The clean-profile first-run text and error dialog show that `codex-windows-sandbox-setup.exe` fails during the permission/bootstrap stage.

Every uninstall and reinstall now returns the same Store version:

```text
26.721.11231.0
```

The earlier successful recovery happened only because the Store replaced the older broken build `26.721.4979.0` with a genuinely newer build. Repeating the uninstall procedure now only reinstalls the same failing package.

**Updated requested resolution: suspend Store distribution of build `26.721.11231.0` until OpenAI ships a verified replacement with reliable bootstrap, sandbox setup, multi-task handling, diagnostics, and recovery behavior.**

---

## Follow-up incident: task archive failure and continuing instability

On 2026-08-19, package `26.814.5517.0` failed to archive completed local Codex tasks. The thread store returned Windows `os error 2`, and completed automation runs remained visible in the sidebar. Nineteen runs from one daily automation had accumulated when cleanup was attempted.

Codex Desktop also crashed again while the follow-up report was being prepared. That crash is recorded as an additional reliability observation, not as proof that the archive defect caused it.

Immediately after the crash, Microsoft Store showed `Update available` for Codex and then transitioned to `Installing` without a separate user confirmation. The observation is recorded without asserting which component initiated the Store transaction.

See the standalone sanitized report:

- [`Codex-Windows-Task-Archive-Failure-20260819.md`](./Codex-Windows-Task-Archive-Failure-20260819.md)

## Current requested resolution — 2026-08-20

The requested resolution is no longer limited to suspending superseded build `26.721.11231.0`. OpenAI should:

1. Repair local-task archiving and the missing-file/thread-store failure.
2. Provide supported task-store diagnostics, repair/reindex, and recovery procedures.
3. Ensure archive and other state-changing operations return structured failures that callers cannot misread as success.
4. Test recurring automation cleanup so completed run tasks do not accumulate in the active sidebar.
5. Investigate continuing crashes, freezes, duplicated prompts, failed steering, and abnormal idle resource consumption across later Store builds.
6. Make Store update initiation, progress, failure, rollback, and installed-version identity explicit and diagnosable.
7. Regression-test startup, sandbox setup, updates, reinstalls, concurrent tasks, task-state recovery, and sidebar restoration on supported Windows 10 and Windows 11 environments.
