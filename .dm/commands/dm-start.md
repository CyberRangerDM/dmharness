# DM Command: start

Logical command: `/dm:start [task-title]`

Platform entrypoints:

- Codex: `$dm start [task-title]`
- Claude Code: `/dm-start [task-title]`

Purpose: create a new Harness-DM task and enter the `clarifying` phase.

This is a platform-neutral command template. Claude Code and Codex adapters may copy or reference this file. Do not invoke a self-developed CLI.

## Inputs

- Optional task title from the user.
- Current date/time.
- Current platform: `claude-code` or `codex`.
- User's original task request from the conversation.

## Read

- `AGENTS.md`
- `.dm/workflow.md`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`
- `.dm/templates/task-state.json`
- `.dm/templates/task-brief.md`
- `.dm/templates/task-summary.md`
- `.dm/templates/command-log.md`

## Generate

Task ID format:

```text
YYYYMMDDHHmm-xxxxxxxx
```

Where `xxxxxxxx` is an 8 character uuid/random suffix.

## Write

Create:

```text
.dm/tasks/[task-id]/
├── state.json
├── events.jsonl
├── brief.md
├── summary.md
└── command-log.md
```

Initial state:

- `status`: `active`
- `phase`: `clarifying`
- `iteration`: `1`
- `roles.worker.status`: `pending`
- `roles.test.status`: `pending`
- `roles.accept.status`: `pending`

Append one `task_created` event to `events.jsonl`.

## Behavior

1. If no task title is provided, infer a short title from the user's request.
2. Generate a task id.
3. Create the task directory.
4. Fill `state.json` from `.dm/templates/task-state.json`.
5. Fill `brief.md` from `.dm/templates/task-brief.md` using known request details.
6. Fill `summary.md` from `.dm/templates/task-summary.md` with current phase and next action.
7. Start clarifying with an interactive confirmation prompt in the CLI. This first response must include the first meaningful pending confirmation point with 3 or more concrete options plus a manual input option.

`brief.md` is immediately user-editable. Tell the human they may either continue the conversation or directly edit `brief.md` before invoking the platform continue command.

## Required Clarifying Prompt Format

During `clarifying`, Main Agent must complete at least three meaningful CLI-visible clarification rounds before the task can leave `clarifying`. Each round uses this shape:

```text
对于待确认点A，有多个方案:
1. aaa
2. bbb
3. ccc
4. [用户手动填入]
```

Rules:

- Replace `待确认点A` with a concrete unresolved requirement point.
- Provide at least 3 meaningful options before `[用户手动填入]`.
- Accept either a numbered choice or free-form human input.
- After the human answers, update `brief.md` with the pending point, requirement impact, options, selected answer, final value, and status.
- A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case.
- If fewer than three answered meaningful rounds are recorded, ask another confirmation prompt in the same format.
- After three answered meaningful rounds, continue asking only while key ambiguity remains.
- Do not ask filler questions solely to increase the count. If no meaningful next question can be identified, say so and ask the human for missing context or confirmation of omitted requirements.

## Failure Rules

- If a task directory with the generated id already exists, generate a new suffix.
- Do not overwrite an existing task directory.
- If the original request is insufficient to write a basic brief, create a skeletal `brief.md`, ask a required interactive confirmation prompt, and still keep phase as `clarifying`.

## User Response

Report:

- task id
- current phase
- path to `summary.md`
- next interactive confirmation prompt; do not state that the task is ready for the platform continue command until at least three meaningful confirmations have been answered and recorded in `brief.md`

## Acceptance Criteria

- A task id matching `YYYYMMDDHHmm-xxxxxxxx` is created.
- `state.json`, `events.jsonl`, `brief.md`, `summary.md`, and `command-log.md` exist.
- `state.json.phase` is `clarifying`.
- `events.jsonl` contains a valid JSON line with `type = "task_created"`.
- The user response includes a CLI-visible confirmation prompt with at least 3 options plus `[用户手动填入]`.
- Clarifying cannot be presented as ready for continue until `brief.md` contains at least three answered meaningful confirmation records.
