# DM Command: feedback

Logical command: `/dm:feedback [task-id] [feedback]`

Platform entrypoints:

- Codex: `$dm feedback [task-id] [feedback]`
- Claude Code: `/dm-feedback [task-id] [feedback]`

Purpose: record human, test, or accept feedback and route the task back to `working` when appropriate.

This is a platform-neutral command template. Claude Code and Codex adapters may copy or reference this file. Do not invoke a self-developed CLI.

## Inputs

- Optional task id.
- Feedback text.
- Feedback source:
  - `human`
  - `test`
  - `accept`
  - `main`

If task id is omitted, load the most recently updated active task from `.dm/tasks/*/state.json`.

If multiple active tasks have the same latest `updated_at`, stop and ask the user to specify a task id.

## Read

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- Existing `.dm/tasks/[task-id]/feedback-[n].md` files
- Latest role reports if relevant

## Write

- `.dm/tasks/[task-id]/feedback-[next].md`
- Updated `.dm/tasks/[task-id]/state.json`
- Appended `.dm/tasks/[task-id]/events.jsonl`
- Updated `.dm/tasks/[task-id]/summary.md`

## Feedback File Format

Create the next available numbered file:

```text
.dm/tasks/[task-id]/feedback-001.md
.dm/tasks/[task-id]/feedback-002.md
```

Use zero-padded 3 digit numbering. Never overwrite an existing feedback file.

Content:

```md
# Feedback {{NUMBER}}: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Source: `{{SOURCE}}`
- Previous Phase: `{{PREVIOUS_PHASE}}`
- Created At: `{{CREATED_AT}}`

## Feedback

{{FEEDBACK}}

## Required Response

{{REQUIRED_RESPONSE}}
```

## Behavior

1. Resolve task id.
2. Determine next feedback number.
3. Write feedback file.
4. Update state:
   - `phase`: `working`
   - increment `iteration` when feedback requires another worker pass
   - set relevant role statuses back to `pending` as needed
5. Append a `feedback_recorded` event.
6. Update `summary.md` with latest feedback and next action.

## Phase Rules

- Feedback during `testing` returns to `working`.
- Feedback during `accepting` returns to `working`.
- Feedback during `human_acceptance` returns to `working`.
- Feedback during `clarifying` should update or request edits to `brief.md` and may stay in `clarifying`.
- Feedback about requirements after `clarifying` should return to `clarifying` when `brief.md` needs revision.
- Feedback about the design during `designing` or `design_review` should update or request edits to `design.md`; it returns to `designing` when a revised draft is needed or stays in `design_review` when only confirmation is pending.
- Feedback about already confirmed design returns to `design_review` before implementation continues, and the current `design.md` must be reconfirmed.
- Feedback during `blocked` may return to the previous phase or `clarifying`, depending on the human response.

## Failure Rules

- If feedback text is empty, ask the user for feedback content and do not write a file.
- If no active task exists and no task id is provided, say no active task was found.
- If `state.json` is missing or malformed, stop and do not infer state from Markdown alone.
- Never overwrite an existing feedback file.
- Do not modify business code.

## Event Format

Append one JSON line to `events.jsonl`:

```json
{"time":"{{TIMESTAMP}}","type":"feedback_recorded","actor":"{{SOURCE}}","phase":"{{PREVIOUS_PHASE}}","to":"working","feedback_file":"feedback-{{NUMBER}}.md","reason":"feedback requires another worker pass"}
```

## User Response

Report:

- task id
- feedback file path
- previous phase
- new phase
- next action for Worker

## Acceptance Criteria

- Feedback creates the next zero-padded `feedback-[n].md`.
- Feedback never overwrites existing feedback.
- Feedback that requires rework returns task to `working`.
- Feedback event is appended to `events.jsonl`.
- `summary.md` reflects latest feedback and next action.
