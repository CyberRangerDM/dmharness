---
description: Recover an interrupted Harness-DM task from persisted .dm files.
argument-hint: [task-id]
---

Treat this command as logical `/dm:continue [task-id]`.

User arguments are available as `$ARGUMENTS`.

Read first:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-continue.md`
- `.dm/specs/phase-controller.spec.md`
- `.dm/specs/persistence.spec.md`

Execute the workflow by reading and writing `.dm` files only. Do not invoke a self-developed CLI.

Required behavior:

- Load the specified task, or the latest active task by `updated_at`.
- If multiple active tasks share the latest `updated_at`, ask the user to specify task id.
- Recover task context from `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`.
- If the task is already complete, report completion and do not advance.
- If incomplete, resume from the first incomplete required phase without rerunning phases already proven complete by files.
- If required artifacts are missing, report missing items and do not advance.
- If `state.json` is missing or malformed, stop and do not infer state from Markdown.
- Do not advance `done` tasks.
- Append one `phase_transition` event when phase changes.
- Update `.dm/tasks/[task-id]/summary.md`.
- Persist formal design through the automatic design review step.
- Create `.dm/session/[task-id]/summary.md` when entering `human_acceptance`.

Claude command name note: `/dm-continue` is accepted as the Claude Code equivalent of `/dm:continue`.
