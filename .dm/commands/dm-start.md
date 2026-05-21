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

- `.dm/templates/task-state.json`
- `.dm/templates/task-summary.md`
- `.dm/templates/command-log.md`

Read `.dm/templates/task-brief.md` only when clarify is complete and the final `brief.md` is being written. Read broader workflow or spec files only if the start behavior is ambiguous. Do not load unrelated command, role, or template files for a normal start.

## Generate

Task ID format:

```text
YYYYMMDDHHmm-xxxxxxxx
```

Where `xxxxxxxx` is an 8 character uuid/random suffix.

## Write

Create the task directory and write the immediate files:

```text
.dm/tasks/[task-id]/
├── state.json
├── events.jsonl
├── summary.md
└── command-log.md
```

Deferred clarify artifact:

```text
.dm/tasks/[task-id]/brief.md
```

`brief.md` is written once after clarify is complete, unless the human explicitly creates or edits it earlier.

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
5. Fill `summary.md` from `.dm/templates/task-summary.md` with current phase and next action.
6. Start clarifying with an interactive confirmation prompt in the CLI. This first response must include the first meaningful pending confirmation point with 3 or more concrete options plus a manual input option.
7. The first prompt and all later clarifying prompts must follow `specs/grill-me-discussion.spec.md`: ask one main pending point at a time, resolve upstream decisions before downstream details, include the Main Agent recommended answer and reason, and do not ask the human for facts that can be found by reading the codebase or `.dm` files.
8. Do not create or update `brief.md` after every answer. Keep answered confirmation records in the current clarify working set, then write `brief.md` once after the clarify gate is satisfied. If the human explicitly edits or asks for an early `brief.md`, treat that as an exception and merge it into the final one-shot write.

`brief.md` is the final clarify artifact, not the per-round scratchpad. Tell the human they may continue the conversation; if they directly create or edit `brief.md`, Main Agent must treat it as an external edit and merge it before finalizing. After the clarify gate is satisfied, Main Agent proceeds into design automatically and stops at `design_review` for approval.

## Required Clarifying Prompt Format

During `clarifying`, Main Agent must complete at least three meaningful CLI-visible clarification rounds before the task can leave `clarifying`. Each round uses this shape:

```text
对于待确认点A，有多个方案:
1. aaa
2. bbb
3. ccc
4. [用户手动填入]

推荐答案: 2
推荐理由: ...
决策影响: ...
前置依赖: ...
```

Rules:

- Replace `待确认点A` with a concrete unresolved requirement point.
- Provide at least 3 meaningful options before `[用户手动填入]`.
- Ask exactly one main pending point per round.
- Provide the Main Agent recommended answer and recommendation reason.
- State the decision impact and any upstream dependency.
- Explore available project files and `.dm` artifacts before asking; do not ask the human for facts that can be discovered locally.
- Accept either a numbered choice or free-form human input.
- After the human answers, keep the pending point, requirement impact, options, recommended answer, recommendation reason, upstream dependency, exploration evidence, selected answer, final value, and status in the current clarify working set.
- A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case.
- If fewer than three answered meaningful rounds are recorded, ask another confirmation prompt in the same format.
- After three answered meaningful rounds, continue asking only while key ambiguity remains.
- Do not ask filler questions solely to increase the count. If no meaningful next question can be identified, say so and ask the human for missing context or confirmation of omitted requirements.
- When the final clarify gate is satisfied, read `.dm/templates/task-brief.md` and write `.dm/tasks/[task-id]/brief.md` once with all answered records and final requirement content.

## Failure Rules

- If a task directory with the generated id already exists, generate a new suffix.
- Do not overwrite an existing task directory.
- If the original request is insufficient to write a final brief, ask a required interactive confirmation prompt and still keep phase as `clarifying`; do not create a skeletal `brief.md` unless the human explicitly needs an editable file.

## User Response

Report:

- task id
- current phase
- path to `summary.md`
- future path to `brief.md`
- next interactive confirmation prompt; do not ask for a platform continue command to leave clarifying. Once at least three meaningful confirmations have been answered in the clarify working set and no key ambiguity remains, write `brief.md` once and proceed directly to design.

## Acceptance Criteria

- A task id matching `YYYYMMDDHHmm-xxxxxxxx` is created.
- `state.json`, `events.jsonl`, `summary.md`, and `command-log.md` exist; `brief.md` is absent or explicitly marked deferred until clarify is complete, unless the human requested an early editable file.
- `state.json.phase` is `clarifying`.
- `events.jsonl` contains a valid JSON line with `type = "task_created"`.
- The user response includes a CLI-visible confirmation prompt with at least 3 options plus `[用户手动填入]`.
- Clarifying cannot complete until the final one-shot `brief.md` contains at least three answered meaningful confirmation records.
