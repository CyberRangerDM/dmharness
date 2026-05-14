---
description: Show the current Harness-DM task status.
argument-hint: [task-id]
---

Treat this as logical `/dm:status [task-id]`.

User arguments are available as `$ARGUMENTS`.

Read first:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-status.md`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`

Execute the workflow by reading `.dm` files only. Do not invoke a self-developed CLI.

Required behavior:

- Load the specified task, or the latest active task by `updated_at`.
- If multiple active tasks share the latest `updated_at`, ask the user to specify task id.
- Read `state.json` and `summary.md`.
- Read latest worker/test/accept reports if present.
- Display phase, status, iteration, role states, latest reports, pending human decision, missing artifacts, and next action.
- Do not mutate files, do not write `command-log.md`, and do not change phase.
