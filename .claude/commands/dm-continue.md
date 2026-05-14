---
description: Advance the current Harness-DM task by one phase if completion conditions are met.
argument-hint: [task-id]
---

Treat this command as logical `/dm:continue [task-id]`.

User arguments are available as `$ARGUMENTS`.

Read first:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-continue.md`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`

Execute the workflow by reading and writing `.dm` files only. Do not invoke a self-developed CLI.

Required behavior:

- Load the specified task, or the latest active task by `updated_at`.
- If multiple active tasks share the latest `updated_at`, ask the user to specify task id.
- Advance at most one phase.
- If required artifacts are missing, report missing items and do not advance.
- If `state.json` is missing or malformed, stop and do not infer state from Markdown.
- Do not advance `done` tasks.
- Append one `phase_transition` event when phase changes.
- Update `.dm/tasks/[task-id]/summary.md`.
- Persist formal design only after human confirmation.
- Create `.dm/session/[task-id]/summary.md` when entering `human_acceptance`.

Claude command name note: `/dm-continue` is accepted as the Claude Code equivalent of `/dm:continue`.
