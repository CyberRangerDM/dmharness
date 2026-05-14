---
description: Record Harness-DM feedback and route the task to the correct phase.
argument-hint: [task-id] [feedback]
---

Treat this as logical `/dm:feedback [task-id] [feedback]`.

User arguments are available as `$ARGUMENTS`.

Read first:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-feedback.md`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`

Execute the workflow by reading and writing `.dm` files only. Do not invoke a self-developed CLI.

Required behavior:

- Load the specified task, or the latest active task by `updated_at`.
- If multiple active tasks share the latest `updated_at`, ask the user to specify task id.
- If feedback text is empty, ask the user for feedback content and do not write a file.
- Create the next zero-padded `.dm/tasks/[task-id]/feedback-[n].md`.
- Never overwrite existing feedback.
- Append a `feedback_recorded` event.
- Update `state.json` and `summary.md`.
- Route implementation/test/accept/human feedback back to `working` when rework is required.
- Route design feedback back to `designing` or `design_review` as appropriate.
