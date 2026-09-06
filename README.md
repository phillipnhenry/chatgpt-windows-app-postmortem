# ChatGPT Windows App: A Postmortem

A technical postmortem of repeated failures in the Microsoft Store ChatGPT/Codex Windows application, including failed reinstalls, machine-wide bootstrap errors, recovery work, Procmon evidence, sandbox setup failures, and recommendations for OpenAI and Microsoft.

## Current verdict — 2026-09-06

Currently installed and running Microsoft Store package:

```text
OpenAI.Codex
Version: 26.901.5003.0
Store product ID: 9PLM9XGG6VKS
ChatGPT.exe file/product version: 152.0.7977.64
```

The Codex Windows app is not production-quality. In ordinary language, it is crap: unreliable, continuity-breaking, difficult to diagnose, and capable of turning a successful development session into hours or days of repair work. The underlying Codex agent can produce excellent work; the Windows application wrapped around it repeatedly damages that work by crashing, losing task capabilities, reviving stale state, confusing archived and active tasks, and providing no dependable recovery path.

The development, update, and release process behind the Windows app is equally unacceptable. Updates have closed working sessions, exposed ambiguous progress, failed to restart as promised, and shipped without preventing regressions in startup, task archiving, task identity, project placement, working-directory state, and tool attachment. These are not cosmetic defects. They attack the continuity guarantees on which a serious development tool depends.

The latest crash on 2026-09-06 damaged a previously functioning Brand Navigation task:

- the task retained its conversation but lost its callable browser-control runtime;
- Chrome, the OpenAI browser extension, and native-host registration were healthy;
- the task received ambient browser-tab metadata but no browser-control interface;
- a forced archive/unarchive reload did not restore the capability;
- the task's recorded working directory regressed to an obsolete, nonexistent path after the correct project and repository location had already been restored;
- sidebar visibility, pinning, archive state, and project placement again required manual repair; and
- the app exposed contradictory state, including a “Could not unarchive this task” error while backend records disagreed about whether the task was archived.

This is an app failure, not a Chrome failure, a Discourse failure, or a user-configuration failure. A crash should not silently change what tools a resumed task can use or resurrect obsolete task metadata. If a task cannot be resumed with the same identity, project, working directory, permissions, and attached capabilities, then the app has not recovered the task—it has only reopened a damaged transcript.

## Billing and usage-reset failure — answer required

ChatGPT Pro 20× renewed automatically on **2026-09-04** for **$200.00**. The available Codex usage did not reset with that paid monthly renewal. On 2026-09-06, the account still reported **57% of the current Codex allowance used**, with its own separate usage-window reset scheduled for later that day.

OpenAI needs to answer this directly: **Why was another $200 collected automatically for a new month of ChatGPT Pro 20× while the paid usage available to the customer did not reset at renewal?** If subscription billing and usage windows intentionally run on unrelated clocks, where was that disclosed clearly before renewal, and what exactly did the new $200 payment replenish at the moment it was charged?

The requested remedy is not marketing language or a generic link to usage documentation. OpenAI should provide the billing-period and usage-window ledger, explain the mismatch, immediately restore the allowance that the renewed month reasonably implies, and credit or refund any paid period for which the advertised 20× capacity was not actually renewed.

## Bug reporting is functioning as a dumping ground, not a managed process

As of 2026-09-06, the 13 OpenAI Codex issues cited by this postmortem show:

- **10 open and three closed**;
- **12 of 13 with no assignee**;
- **nine of the ten open issues with no assignee**;
- **183 comments in total, with zero comments identified by GitHub as coming from an OWNER, MEMBER, or COLLABORATOR**; and
- the exact Store bootstrap report, [#36272](https://github.com/openai/codex/issues/36272), still open, unassigned, and carrying zero comments since July 31.

The reports are [#39492](https://github.com/openai/codex/issues/39492), [#39638](https://github.com/openai/codex/issues/39638), [#39600](https://github.com/openai/codex/issues/39600), [#39239](https://github.com/openai/codex/issues/39239), [#39130](https://github.com/openai/codex/issues/39130), [#25489](https://github.com/openai/codex/issues/25489), [#19352](https://github.com/openai/codex/issues/19352), [#19437](https://github.com/openai/codex/issues/19437), [#19770](https://github.com/openai/codex/issues/19770), [#26624](https://github.com/openai/codex/issues/26624), [#33483](https://github.com/openai/codex/issues/33483), [#13993](https://github.com/openai/codex/issues/13993), and [#36272](https://github.com/openai/codex/issues/36272).

Only #13993, the request for a standalone Windows installer, has an assignee. Closing duplicates while leaving the canonical reports unassigned and publicly silent is not meaningful bug management. Paying users are supplying reproduction steps, logs, cross-version comparisons, and recovery evidence while OpenAI supplies labels and little visible ownership.

## Where are the Windows version-to-version release notes?

OpenAI now publishes a combined [ChatGPT and Codex changelog](https://learn.chatgpt.com/docs/changelog), but it contains no entry for installed Windows package `26.901.5003.0`. It mixes general ChatGPT announcements, iOS releases, Codex CLI releases, and occasional desktop build numbers without providing a dependable mapping from a Microsoft Store package to its embedded Codex/app-server/browser components, fixed issues, known regressions, migrations, or rollback requirements.

The September 1 entries claim more reliable task loading and reconnects, restored working directories for resumed threads, and continued MCP-tool availability through refreshes. The installed Windows app is now failing in those exact areas, yet the changelog does not establish whether `26.901.5003.0` contains those changes. A customer therefore cannot determine what changed, which bug reports were fixed, what remains broken, or whether an update is safe before allowing the Store to replace a working installation.

Every Windows build needs its own release record: exact Store version, release date, rollout status, embedded component versions, fixed GitHub issue numbers, known problems, state migrations, compatibility changes, recovery steps, and rollback path. “Additional performance improvements and bug fixes” is not acceptable change control for software entrusted with long-running development state.

## Earlier status — 2026-08-20

Currently running Microsoft Store package:

```text
OpenAI.Codex
Version: 26.818.2441.0
Store product ID: 9PLM9XGG6VKS
```

The application currently launches, so the original complete startup failure on build `26.721.11231.0` should no longer be presented as the current package state. Reliability defects continue:

- completed local tasks fail to archive with thread-store Windows `os error 2`;
- recurring automation runs accumulate in the active sidebar;
- crashes, freezes, duplicated prompts, failed steering, and uncertain responsiveness have continued across later builds;
- Codex crashed while the archive incident report was being prepared on 2026-08-19;
- another update sequence on 2026-08-20 closed Codex before installation, presented ambiguous download/update progress, and failed to perform the promised automatic restart; and
- after manual relaunch, Microsoft Store and the running processes both reported `26.818.2441.0`, but archive testing confirmed that the defect remains intermittent rather than fixed.

See the current follow-up report:

- [`Codex-Windows-Task-Archive-Failure-20260819.md`](./Codex-Windows-Task-Archive-Failure-20260819.md)

## Public issue tracking — 2026-08-20

- [openai/codex#39492](https://github.com/openai/codex/issues/39492) is the consolidated open report for the current general Windows task-archive failure.
- I independently filed [openai/codex#39638](https://github.com/openai/codex/issues/39638) to document the additional recurring-automation accumulation and false-success impact. I transferred its unique evidence to `#39492` and then closed `#39638` as a duplicate.
- [openai/codex#39600](https://github.com/openai/codex/issues/39600), [#39239](https://github.com/openai/codex/issues/39239), and [#39130](https://github.com/openai/codex/issues/39130) contain path-specific evidence indicating that Windows extended-length `\\?\` rollout-path handling is a likely cause. This is strong community reproduction evidence, not yet an official OpenAI root-cause determination.

## Original critical finding — 2026-07-31

Microsoft Store package:

```text
OpenAI.Codex
Version: 26.721.11231.0
Store product ID: 9PLM9XGG6VKS
```

The application repeatedly exits with:

```text
The application is exiting and cannot service this request
```

A clean Windows-profile test also exposed failure in:

```text
codex-windows-sandbox-setup.exe
```

Fresh Process Monitor traces showed `ChatGPT.exe` loading only itself and `ntdll.dll`, using effectively zero CPU time, and exiting in approximately 0.26 seconds with:

```text
0xC0000001
STATUS_UNSUCCESSFUL
```

## Full postmortem

See:

- [`ChatGPT-Windows-App-A-Postmortem.md`](./ChatGPT-Windows-App-A-Postmortem.md)
- [`Codex-Windows-Task-Archive-Failure-20260819.md`](./Codex-Windows-Task-Archive-Failure-20260819.md) — follow-up report covering broken task archiving, automation-run sidebar accumulation, continuing crashes, and the defect's confirmed persistence in package `26.818.2441.0` on 2026-08-20.

## Scope

This repository documents:

- repeated Microsoft Store installation failures,
- direct AppX removal and reinstall attempts,
- clean-profile testing,
- Procmon and ProcDump findings,
- sandbox bootstrap failure,
- Windows integrity checks,
- recovery of chats, projects, workspace roots, and pinned threads,
- failure to archive completed local tasks and resulting sidebar growth,
- continued crashes observed after the original recovery period,
- and the case for temporarily suspending the affected Store build.

## Evidence

- [Sanitized public evidence package](./evidence/ChatGPT-Codex-Public-Evidence-SANITIZED-20260730.zip)
- [Full repository](https://github.com/phillipnhenry/chatgpt-windows-app-postmortem)

## Privacy

Public evidence should be sanitized before posting. Raw logs can contain Windows usernames, local project paths, project names, account SIDs, environment variables, and thread identifiers.

## Requested resolution

OpenAI should treat the Windows failures as a continuing reliability and state-management problem rather than a defect confined to one superseded build. Requested work now includes:

- repair task archiving and add create/archive/list/restore integration coverage;
- prevent recurring automation-run tasks from accumulating in the active sidebar;
- return structured operation failures that cannot be mistaken for success;
- normalize and consistently handle Windows extended-length rollout paths during task resume and archive operations;
- provide supported thread-store repair/reindex and local-state recovery tooling;
- investigate continuing crashes, freezes, failed steering, duplicate prompts, and abnormal idle resource use;
- make Store update initiation and completion explicit and diagnosable; and
- regression-test startup, sandbox setup, updates, reinstalls, concurrent tasks, recovery, and sidebar state across supported Windows versions and hardware.
