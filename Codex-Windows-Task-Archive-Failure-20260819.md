# Codex Windows task archiving fails and completed tasks accumulate in the sidebar

Observed: 2026-08-19

Platform: Codex Desktop for Windows

Installed package observed from the running process: `OpenAI.Codex_26.814.5517.0_x64__2p2nqsd0c76g0`

## Summary

Codex Desktop cannot archive existing local tasks. The task archive operation reaches the app but fails in the underlying thread store with Windows `os error 2`. The task remains visible in the active sidebar.

This is especially damaging for recurring automations because each run creates another task. Nineteen completed runs of one daily automation were visible during this test, along with retired and otherwise completed tasks that could not be removed from the active tree.

## Relationship to the broader Windows failure history

This is a follow-up incident to the larger Windows app postmortem:

- [ChatGPT Windows App: A Postmortem](https://github.com/phillipnhenry/chatgpt-windows-app-postmortem)
- [Full technical postmortem in this repository](./ChatGPT-Windows-App-A-Postmortem.md)
- [Sanitized public evidence package](./evidence/ChatGPT-Codex-Public-Evidence-SANITIZED-20260730.zip)

That record documents repeated Codex/ChatGPT Windows crashes and unlaunchable states, failed updates and reinstalls, machine-wide bootstrap failure, sandbox setup failure, clean-profile testing, Procmon evidence, and the substantial recovery work required to reconstruct projects, chats, workspace roots, and pinned tasks.

The archive failure is distinct from the earlier startup/bootstrap failure, but it compounds the same reliability and recoverability problem: the app cannot reliably maintain its own task state, and the supported UI operation has no safe manual fallback.

## Additional CodeProjects incident history

A search of the current `CodeProjects` workspace and its root Git history found the following additional reliability evidence. This is a sanitized summary; private task identifiers and local source-document paths are intentionally omitted.

### 2026-07-20 — crash or freeze during ordinary planning

- Codex crashed or froze while the user was entering a normal product-planning thought.
- The interruption converted an ordinary planning/build moment into hours of recovery and reliability work.
- The incident established a standing workspace rule to treat Codex crashes, freezes, approval timeouts, and sandbox friction as operational debt rather than isolated inconvenience.

### 2026-07-24 — task-level agent loop died

- A task remained readable but could not accept new turns.
- Two routed attempts failed with:

  ```text
  failed to start turn: internal error; agent loop died unexpectedly
  ```

- Restarting Codex, logging off, ending Codex tasks, and retrying did not recover the task.
- Work had to move to a replacement task while the original became a provenance lane.

### 2026-07-25 — crash followed by failure to restart

- Codex crashed again during active product coordination.
- The application would not start normally afterward.
- Recovery required reinstalling Codex.
- The workspace incident record explicitly describes reinstall-as-recovery as too heavy for ordinary daily work.

### 2026-08-06 — Store update hangs and then reports failure

- The in-app update control opened Microsoft Store.
- Store remained at `Installing (100%)` and `Almost done...` for approximately one hour.
- Leaving the machine idle did not resolve the update.
- End Task had previously failed to stop the app; shutdown stopped Codex, but uninstall also failed.
- A screenshot at 13:05:44 preserved the `Installing (100%)` / `Almost done...` state.
- A later screenshot at 17:14:28 showed `Retry` and `Something went wrong`.
- The subsequently running package was `OpenAI.Codex_26.730.8199.0`.
- Because no reliable pre-attempt package identity was captured, the evidence does not prove whether Store installed a newer package before reporting failure.
- Codex subsequently crashed while the user was dictating a workflow correction.

### 2026-08-14 — repeated freezes, crashes, duplicate prompts, and failed steering

- A long-running task recorded repeated app freezes, crashes, duplicated prompts, failed steering, and uncertain responsiveness.
- A read-only ten-second process sample found approximately 40.3 percent total CPU use while no build, script, migration, server, or workspace job was running.
- The main process used approximately 31.4 percent CPU, the GPU process 5.3 percent, the main renderer 3.1 percent, and the app server 0.1 percent.
- Combined working set was approximately 2.36 GB across 14 application processes.
- Node helpers and the code-mode host were idle.
- The installed package observed during the earlier process inspection was `OpenAI.Codex_26.810.4967.0`.

### 2026-08-19 — archive failure, crash during reporting, and unexpected Store installation

- Package `OpenAI.Codex_26.814.5517.0` was observed from the running process before the newly observed Store installation completed.
- Archiving completed tasks failed with the thread-store `os error 2` documented in this report.
- Codex crashed while this incident report was being prepared.
- Immediately afterward, Microsoft Store showed `Update available` and then changed to `Installing` without a separate user confirmation.

### Limits of the workspace search

The search covered current Markdown, text, log, JSON, and CSV records under `CodeProjects`, filename-based incident candidates, and relevant root Git history paths. It found the central reliability incident ledger, Boss continuity checkpoints, the August update evidence, and task-lane failure notes.

This does not claim that every crash generated a durable note or Windows diagnostic artifact. Several workspace records explicitly describe lost context and the need to improve crash-safe checkpoints, so undocumented crashes may also have occurred.

## Steps to reproduce

1. Open Codex Desktop on Windows.
2. Identify a completed local Codex task that remains visible in the sidebar.
3. Invoke the app task-archive operation for that task.
4. Refresh or inspect the active task list.
5. Observe that the task remains visible.
6. Retry the archive operation and inspect the returned error.

## Actual result

```text
failed to archive session: thread-store internal error: failed to archive thread: The system cannot find the file specified. (os error 2)
```

The task remains visible in the sidebar.

During a batch cleanup attempt, archive calls resolved at the tool-call transport level but did not remove the tasks. A direct retry exposed the thread-store error above. Callers that treat a resolved tool invocation as success without inspecting the returned text can therefore misreport the operation as successful.

## Expected result

The completed task should disappear from the active sidebar and remain recoverable through the archived-task view.

Archiving should either:

- complete atomically and return success; or
- fail with a structured error that the app and automation runner cannot mistake for success.

## Impact

- Recurring automations create a continuously growing visible task tree.
- Retired or completed operational tasks cannot be removed from the active sidebar.
- Sidebar usability degrades over time.
- Automation prompts that explicitly self-archive after completion cannot enforce cleanup while the thread store is broken.
- There is no safe user workaround. Deleting local task-store files manually could destroy history or corrupt the task index.

## Additional crash observed while documenting this incident

While this report was being prepared in Codex Desktop, the app crashed again and the work had to resume after recovery. The temporary repository clone and conversation context survived this particular interruption.

This observation is included as another instance of the continuing crash pattern. It does not establish that the archive failure caused the crash.

Immediately after that crash, Microsoft Store displayed `Update available` for the app and then changed to `Installing` without the user explicitly confirming an install action. This is recorded as observed update behavior, not as proof of what process triggered the Store transition.

The sequence observed was:

1. Codex Desktop crashed while the incident report was being prepared.
2. Microsoft Store was opened immediately afterward.
3. The Codex app showed `Update available`.
4. The Store changed to `Installing` without a separate user confirmation.

This sequence matters because the broader postmortem already documents failures associated with Store-delivered updates and reinstalls. OpenAI and Microsoft should be able to correlate the package update transaction, app crash, and task-store failure by timestamp in their respective diagnostics.

## Privacy and sanitization

The public report omits real local task IDs, Windows usernames, private project names, and raw local paths. Those identifiers can be supplied privately if OpenAI requests a diagnostic correlation sample.

## Requested resolution

1. Repair task archiving in the Windows thread store.
2. Return structured operation failures instead of failure text through a nominally resolved call.
3. Add an integration test that creates, archives, lists, and restores a local task.
4. Add a recurring-automation test confirming that completed run tasks self-archive and do not accumulate in the active sidebar.
5. Provide a supported repair or reindex operation for existing local task stores affected by missing-file references.
6. Preserve recovery access to task history while repairing the active index.
7. Continue investigating the broader Windows crash, update, reinstall, bootstrap, and sandbox failures documented in the linked postmortem.
8. Clarify and test Microsoft Store update initiation behavior so an available update does not unexpectedly transition to installation while active Codex work and local state may be in use.

## Severity

Moderate and progressively worsening on its own; higher in combination with the established Windows crash and recovery failures. The defect does not immediately destroy visible content, but it makes recurring automations operationally impractical and steadily degrades the primary navigation surface.
