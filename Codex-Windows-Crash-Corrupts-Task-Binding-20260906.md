# Windows app crash restores a task with stale repository metadata and without its browser-control capability

## Summary

The Microsoft Store Codex/ChatGPT Windows app crashed and then restored an existing development task in a materially corrupted state. The conversation survived, but the task's operational identity did not: its working directory regressed to an obsolete repository path, its browser-control runtime disappeared, and its archive/sidebar/project state became contradictory.

This is not a cosmetic defect. A development agent that silently resumes with the wrong repository identity and missing tools is unsafe. It invites the user to trust a task that is no longer the task that existed before the crash.

The Windows app is not production-quality in its handling of long-running development work. The underlying Codex agent can be excellent; the Windows host repeatedly turns that work into recovery labor.

## Environment

```text
Package: OpenAI.Codex
Microsoft Store package version: 26.901.5003.0
Store product ID: 9PLM9XGG6VKS
Executable: app\ChatGPT.exe
ChatGPT.exe file/product version: 152.0.7977.64
OS: Windows
```

The installed version was reconfirmed on 2026-09-06 through two independent local surfaces: Microsoft Store reported **Installed version 26.901.5003.0**, and every running `ChatGPT.exe` process resolved beneath `C:\Program Files\WindowsApps\OpenAI.Codex_26.901.5003.0_x64__2p2nqsd0c76g0\app\ChatGPT.exe`. The executable file and product versions were both `152.0.7977.64`. This identity was not inferred from a marketing version or Codex CLI release.

## Failed recovery through the next in-app update

On 2026-09-08, the in-app updater said the application would restart after updating. It never restarted. The user had to return to it manually.

The update did install a newer package, verified from every running process:

```text
Package: OpenAI.Codex
Package version: 26.901.6511.0
Executable: C:\Program Files\WindowsApps\OpenAI.Codex_26.901.6511.0_x64__2p2nqsd0c76g0\app\ChatGPT.exe
ChatGPT.exe file/product version: 152.0.7977.83
```

The post-update result was another failure:

- all chat histories that had previously been visible in the application were absent;
- the Brand Navigation task still lacked browser control; and
- the task first reported that `node_repl` was missing, then performed a capability-specific check and confirmed that `mcp__cua_repl` was also absent from its callable tool surface.

This is not a missing Node installation, npm package, or `PATH` entry. Both names refer to app-provided Computer Use execution interfaces; a resumed task cannot install or synthesize a host tool that the app did not attach. The newer package therefore did not repair the damaged task-to-browser binding. It failed to perform its promised restart and returned the user to an application with no previously visible chat history.

## What happened

Before the crash, the affected task:

- was attached to the correct project and repository;
- had a valid working directory;
- had authenticated browser control through Chrome; and
- could operate the already configured browser extension/native-host path.

After the app crashed and restarted:

1. The conversation and browser tab were still visible.
2. The task received ambient metadata identifying the browser tab.
3. The callable browser-control runtime was no longer attached.
4. The task's recorded working directory regressed to an obsolete, nonexistent repository path.
5. Sidebar visibility, pinning, archive state, and project placement required manual repair.
6. The app reported `Could not unarchive this task` while archive-list and task-list state disagreed.
7. A forced archive/unarchive cycle did not restore browser control.
8. Opening a fresh Chrome window did not restore the task-to-browser attachment.

Chrome was running. The OpenAI extension was installed and enabled. Native-host registration was correct. The browser side was healthy. The failure was in the resumed Codex task binding.

## Expected behavior

After a crash or restart, an existing task must restore—or explicitly refuse to restore—as one coherent identity:

- exact task ID;
- exact project membership;
- exact working directory and repository;
- archive/pin/sidebar state;
- permission profile;
- tool and plugin inventory;
- browser-extension/native-host attachment; and
- any authenticated browser-control session state that is designed to persist.

If a capability cannot be restored, the app must say so clearly and offer a supported repair/reconnect action. It must not display the old task and browser context while silently dropping the callable capability or reverting repository metadata.

## Actual behavior

The app restored a convincing shell of the task while silently mixing current conversation state with stale repository metadata and a reduced capability set.

That is worse than a clean startup failure. A clean failure is obvious. This failure is deceptive: it looks recovered until the task attempts real work, at which point the user discovers that its repository identity or tools are wrong.

## Impact

This defect:

- halted work on an active Discourse component;
- forced manual recovery of task visibility and project placement;
- left the original task unable to perform browser work it could perform before the crash;
- created a risk of commands being aimed at an obsolete repository location;
- consumed substantial time diagnosing Chrome, the extension, native-host registration, task state, and project placement even though the app binding was the broken layer; and
- provided no supported in-task recovery mechanism.

For a professional coding tool, loss of repository identity and tool attachment after a crash is a release-blocking reliability defect, not an edge case.

## The update and recovery process is unacceptable

OpenAI is shipping a Windows development application without a dependable public contract for upgrade, crash recovery, task-store migration, capability restoration, or rollback. The official troubleshooting guidance largely reduces recovery to checking approvals, running a basic command, restarting the app, or starting a narrower new conversation. None of that repairs a persisted task whose internal binding has been damaged.

The official combined [ChatGPT and Codex changelog](https://learn.chatgpt.com/docs/changelog) does not identify Store package `26.901.5003.0` or map it to its embedded Codex, app-server, browser-control, sandbox, and UI component versions. The September 1 notes claim more reliable task loading/reconnects, restored working directories for resumed threads, and continued MCP-tool availability through refreshes. Those are the exact classes of behavior that failed here, yet a Windows customer cannot determine whether those claims apply to the installed package.

The dedicated [Windows app documentation](https://learn.chatgpt.com/docs/windows/windows-app) explains installation and basic configuration but provides no package-by-package release ledger, known-regression list, migration warning, or rollback procedure.

An automatic Store update is therefore an opaque replacement of a critical development environment. Customers cannot tell what changed, which public issues were fixed, which regressions are known, whether active task state will migrate safely, or how to return to a working build. That is a poor product and a poor release process.

## The public bug-reporting process shows little visible ownership

As of 2026-09-06, a census of 13 related Windows reports in `openai/codex` found:

- 10 open and 3 closed;
- 12 of 13 unassigned;
- 9 of 10 open reports unassigned;
- 183 public issue comments; and
- zero comments whose GitHub API `author_association` was `OWNER`, `MEMBER`, or `COLLABORATOR`.

That last count describes public GitHub metadata; it does not claim that no private investigation exists. The customer-facing result is still indefensible: almost none of these reports shows an owner, investigation state, accepted reproduction, target version, root cause, workaround, or confirmed fixed build.

Related reports include #39492, #39638, #39600, #39239, #39130, #25489, #19352, #19437, #19770, #26624, #33483, #13993, and #36272. The most direct earlier Store/bootstrap report, #36272, remains open, unassigned, and has zero comments. Closing duplicates into an unassigned and publicly silent canonical issue is queue reduction, not bug management.

## Required response

Please do not answer this with generic restart/reinstall advice. Provide:

1. Acknowledgment that the task-binding failure has been reproduced, or a precise request for the missing diagnostic artifact.
2. A responsible owner or team and an internal/public tracking identifier.
3. Identification of the persisted stores and bindings involved: task, project, working directory, archive/pin/sidebar state, tool inventory, browser-control runtime, extension/native-host session, and permission profile.
4. A supported repair procedure for an existing damaged task that does not require abandoning its history and creating a replacement task.
5. The first Windows Store package containing the fix.
6. Regression coverage for crash recovery and in-place update with multiple long-running tasks and browser control.
7. A release ledger mapping every Windows Store package to embedded component versions, fixed issue numbers, known problems, migrations, restart behavior, and rollback/recovery instructions.

## Separate paid-account failure requiring support resolution

ChatGPT Pro 20× renewed automatically on **2026-09-04** for **$200.00**, but the available Codex usage did not reset with the paid monthly renewal. On 2026-09-06, both the Windows app and ChatGPT in the browser showed **43% of the weekly limit left**. A direct account-limit read reported the equivalent **57% used** in a 10,080-minute (seven-day) window. The three readings agree; the complaint is that the newly paid monthly term did not supersede or replenish that still-running weekly allowance.

This is not asserted as the technical cause of the Windows crash, and it may require a separate billing/support ticket. It is part of the same unacceptable customer experience and requires a direct answer:

> Why was another $200 collected automatically for a new month of ChatGPT Pro 20× if the customer's available usage did not reset when that paid month began?

OpenAI's current pricing page says that weekly limits may apply, but it does not explain what an automatic monthly renewal does to a weekly window already in progress. If the billing cycle and usage window are intentionally independent, OpenAI should identify where that limitation was disclosed before renewal, provide the account's exact entitlement/consumption/reset ledger, restore the capacity reasonably associated with the renewal, and credit or refund any period for which the renewed capacity was not supplied.

## Full evidence

The longer incident history, prior package failures, issue census, release-process analysis, and recovery attempts are preserved here:

https://github.com/phillipnhenry/chatgpt-windows-app-postmortem
