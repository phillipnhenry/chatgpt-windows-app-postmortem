# ChatGPT Windows App: A Postmortem

A technical postmortem of repeated failures in the Microsoft Store ChatGPT/Codex Windows application, including failed reinstalls, machine-wide bootstrap errors, recovery work, Procmon evidence, sandbox setup failures, and recommendations for OpenAI and Microsoft.

## Current status — 2026-08-20

Currently running Microsoft Store package:

```text
OpenAI.Codex
Version: 26.814.5517.0
Store product ID: 9PLM9XGG6VKS
```

The application currently launches, so the original complete startup failure on build `26.721.11231.0` should no longer be presented as the current package state. Reliability defects continue:

- completed local tasks fail to archive with thread-store Windows `os error 2`;
- recurring automation runs accumulate in the active sidebar;
- crashes, freezes, duplicated prompts, failed steering, and uncertain responsiveness have continued across later builds;
- Codex crashed while the archive incident report was being prepared on 2026-08-19; and
- immediately afterward, Microsoft Store showed `Update available` and then changed to `Installing` without a separate user confirmation, but the package observed running on 2026-08-20 remains `26.814.5517.0`.

See the current follow-up report:

- [`Codex-Windows-Task-Archive-Failure-20260819.md`](./Codex-Windows-Task-Archive-Failure-20260819.md)

## Public issue tracking — 2026-08-20

- [openai/codex#39492](https://github.com/openai/codex/issues/39492) is the consolidated open report for the current general Windows task-archive failure.
- [openai/codex#39638](https://github.com/openai/codex/issues/39638) was Phil's independent report documenting the additional recurring-automation accumulation and false-success impact. Its unique evidence was transferred to `#39492`, and `#39638` was then closed as a duplicate.
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
- [`Codex-Windows-Task-Archive-Failure-20260819.md`](./Codex-Windows-Task-Archive-Failure-20260819.md) — follow-up report covering broken task archiving, automation-run sidebar accumulation, and a further crash observed while documenting the defect.

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
