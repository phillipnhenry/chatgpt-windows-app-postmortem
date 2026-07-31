# ChatGPT Windows App: A Postmortem

A technical postmortem of repeated failures in the Microsoft Store ChatGPT/Codex Windows application, including failed reinstalls, machine-wide bootstrap errors, recovery work, Procmon evidence, sandbox setup failures, and recommendations for OpenAI and Microsoft.

## Current finding

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

## Scope

This repository documents:

- repeated Microsoft Store installation failures,
- direct AppX removal and reinstall attempts,
- clean-profile testing,
- Procmon and ProcDump findings,
- sandbox bootstrap failure,
- Windows integrity checks,
- recovery of chats, projects, workspace roots, and pinned threads,
- and the case for temporarily suspending the affected Store build.

## Privacy

Public evidence should be sanitized before posting. Raw logs can contain Windows usernames, local project paths, project names, account SIDs, environment variables, and thread identifiers.

## Requested resolution

OpenAI should replace or suspend Store build `26.721.11231.0` until a verified Windows package is available with reliable startup, sandbox setup, diagnostics, concurrent-task handling, and recovery behavior.
