# DM Command: status

Logical command: `/dm:status [task-id]`

Platform entrypoints:

- Codex: `$dm status [task-id]`
- Claude Code: `/dm-status [task-id]`

Purpose: show the current Harness-DM task state in human-readable form.

This is a platform-neutral command template. Claude Code and Codex adapters may copy or reference this file. Do not invoke a self-developed CLI.

## Inputs

- Optional task id.

If task id is omitted, load the most recently updated active task from `.dm/tasks/*/state.json`.

If multiple active tasks have the same latest `updated_at`, stop and ask the user to specify a task id.

## Read

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- Latest role reports if present:
  - `worker-result-[n].md`
  - `test-report-[n].md`
  - `accept-report-[n].md`
- Latest feedback file if present
- `.dm/tasks/[task-id]/brief.md`
- `.dm/design/[task-id]/design.md` if present
- `.dm/session/[task-id]/summary.md` if present

## Write

No writes. The platform status command is always read-only and must not append to `command-log.md`.

## Behavior

1. Resolve task id.
2. Read `state.json`.
3. Read `summary.md`.
4. Report current phase, status, iteration, role states, latest reports, pending human decision, and next action.
5. Report the source-of-truth artifact for the current phase:
   - `clarifying`: `brief.md`
   - `designing` or later design gates: `design.md`
6. If current phase is `clarifying`, report how many answered meaningful interactive confirmation records `brief.md` contains and whether the minimum of three has been met.
7. If current phase is `designing`, report how many answered meaningful interactive design confirmation records `design.md` contains and whether the minimum of three has been met.
8. If any expected artifact is missing for the current phase, list it.

## User Response Format

```text
Task: [task-id] - [title]
Status: [status]
Phase: [phase]
Iteration: [iteration]

Roles:
- worker: [status] ([latest report])
- test: [status] ([latest report])
- accept: [status] ([latest report])

Pending Human Decision:
[pending_human_decision]

Next Action:
[next_action]

Missing Required Artifacts:
- [artifact path]

Clarifying Gate:
[answered meaningful interactive confirmations: N/3, pass/blocker]

Designing Gate:
[answered meaningful interactive design confirmations: N/3, pass/blocker]
```

## Failure Rules

- If no active task exists and no task id is provided, say no active task was found.
- If task id is provided but `state.json` is missing, report the task as invalid and do not infer state from other files.
- If `state.json` is malformed, report the task as invalid and do not infer state from Markdown alone.
- Do not modify phase.
- Do not mutate files.

## Acceptance Criteria

- Status command does not change task phase.
- Status command does not write any file, including `command-log.md`.
- Status command primarily reports `summary.md` text and `state.json` facts.
- Missing artifacts are clearly listed.
- In `clarifying`, fewer than three answered meaningful interactive confirmation records are clearly listed as a blocker.
- In `designing`, fewer than three answered meaningful interactive design confirmation records are clearly listed as a blocker.
- The response is usable by a human without reading JSON directly.
