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
- Latest role report compact summaries if present:
  - `worker-result-[n].md`
  - `test-report-[n].md`
  - `accept-report-[n].md`
- Latest feedback file if present
- `.dm/tasks/[task-id]/brief.md` compact summary and confirmation count when relevant and present; during ongoing `clarifying`, `brief.md` may be intentionally deferred until final one-shot write
- `.dm/design/[task-id]/design.md` compact summary and review status when relevant
- `.dm/session/[task-id]/summary.md` compact summary if present

Read full role reports, `brief.md`, or `design.md` only when the compact summary is missing, stale, or insufficient to explain the current status.

## Write

No writes. The platform status command is always read-only and must not append to `command-log.md`.

## Behavior

1. Resolve task id.
2. Read `state.json`.
3. Read `summary.md`.
4. Report current phase, status, iteration, role states, latest reports, pending human decision, and next action.
5. Report the source-of-truth artifact for the current phase:
   - `clarifying`: `brief.md` if finalized; otherwise report `brief.md` as deferred and use `summary.md` plus current phase state
   - `designing` or later design gates: `design.md`
6. If current phase is `clarifying`, report how many answered meaningful and grill-me-compliant interactive confirmation records `brief.md` contains when it exists. If `brief.md` is deferred, report that the final clarify artifact has not been written yet and that the count is available only from the current conversation/working set.
7. If current phase is `designing` or `design_review`, report whether `design.md` exists, whether it is draft/ready/confirmed, and whether human approval is pending.
8. If any expected artifact is missing for the current phase, list it. Do not list deferred `brief.md` as missing while `clarifying` is still in progress.

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
[answered meaningful grill-me-compliant interactive confirmations: N/3, pass/blocker]

Design Gate:
[design.md: missing/draft/ready_for_review/confirmed, approval: pending/not_required]
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
- In `clarifying`, a finalized `brief.md` with fewer than three answered meaningful and grill-me-compliant interactive confirmation records is clearly listed as a blocker; a deferred `brief.md` is reported as in-progress rather than missing.
- In `designing`, missing or incomplete `design.md` is clearly listed as a blocker; interactive design confirmation counts are not required.
- The response is usable by a human without reading JSON directly.
