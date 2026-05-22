---
description: Create a new Harness-DM task and enter clarifying phase.
argument-hint: [task-title]
---

Treat this as logical `/dm:start [task-title]`.

User arguments are available as `$ARGUMENTS`.

Read first:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-start.md`
- `.dm/specs/phase-controller.spec.md`
- `.dm/specs/persistence.spec.md`

Execute the workflow by reading and writing `.dm` files only. Do not invoke a self-developed CLI.

Required behavior:

- Generate task id in `YYYYMMDDHHmm-xxxxxxxx` format.
- Create `.dm/tasks/[task-id]/`.
- Initialize `state.json`, `events.jsonl`, `brief.md`, `summary.md`, and `command-log.md`.
- Set phase to `clarifying`.
- Append a `task_created` event.
- Report task id, summary path, current phase, and next clarification action.
