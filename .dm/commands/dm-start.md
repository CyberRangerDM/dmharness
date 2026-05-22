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
- `.dm/skills/grill-me.md` when starting or continuing the first clarifying prompt

Read `.dm/templates/task-brief.md` only when the clarified content is ready to be summarized into `brief.md`. Read broader workflow or spec files only if the start behavior is ambiguous. Do not load unrelated command, role, or template files for a normal start.

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

`brief.md` is written after the grill-me clarification produces a usable summary, unless the human explicitly creates or edits it earlier.

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
6. Start clarifying in the CLI using `.dm/skills/grill-me.md`.
7. The first prompt and all later clarifying prompts must follow `.dm/skills/grill-me.md`: interview the human about the plan until shared understanding is reached, walk each branch of the decision tree by resolving dependencies one by one, ask one main question at a time, provide the Main Agent recommended answer, and explore the codebase or `.dm` files instead of asking the human for facts that can be found locally.
8. When the clarified content is ready, write the summarized requirement content to `.dm/tasks/[task-id]/brief.md`.
9. After writing `brief.md`, ask the human whether the file still needs adjustment. If not, proceed to design. If yes, remain in `clarifying` and update `brief.md` from the human's feedback or wait for the human to edit it directly.

`brief.md` is the clarify artifact, not the per-round scratchpad. Tell the human they may continue the conversation or edit `brief.md` directly; if they do, Main Agent must read the latest file before leaving `clarifying`. After the human says no adjustment is needed or that adjustment is complete, Main Agent proceeds into design automatically, treats `design_review` as an automatic validation/persistence step, and continues the remaining phases until `done`, `blocked`, or an internal rework loop.

## Clarifying Rule

Clarifying uses `.dm/skills/grill-me.md`. Read that skill file when the exact discussion behavior is needed.

During `clarifying`, Main Agent only does the clarify work below:

- Ask the human questions according to `.dm/skills/grill-me.md`.
- Ask one main question at a time.
- Provide the Main Agent recommended answer for each question.
- Resolve upstream dependencies before downstream details.
- Explore available project files and `.dm` artifacts before asking; do not ask the human for facts that can be discovered locally.
- When the answers are sufficient to summarize the requirement, read `.dm/templates/task-brief.md` if useful and write `.dm/tasks/[task-id]/brief.md`.
- After writing `brief.md`, ask the human whether the content still needs adjustment.
- If the human says no adjustment is needed, advance to `designing`.
- If the human says adjustment is needed, update `brief.md` from the feedback or wait for direct human edits. After the human says adjustment is complete, reread the latest `brief.md` and advance to `designing`.

## Failure Rules

- If a task directory with the generated id already exists, generate a new suffix.
- Do not overwrite an existing task directory.
- If the original request is insufficient to write a useful `brief.md`, ask the next grill-me question and keep phase as `clarifying`; do not create a skeletal `brief.md` unless the human explicitly needs an editable file.

## User Response

Report:

- task id
- current phase
- path to `summary.md`
- future path to `brief.md`
- next grill-me clarification question; do not ask for a platform continue command to leave clarifying. Once `brief.md` has been written and the human confirms no adjustment is needed or adjustment is complete, proceed directly to design.

## Acceptance Criteria

- A task id matching `YYYYMMDDHHmm-xxxxxxxx` is created.
- `state.json`, `events.jsonl`, `summary.md`, and `command-log.md` exist; `brief.md` is absent or explicitly marked deferred until clarify is complete, unless the human requested an early editable file.
- `state.json.phase` is `clarifying`.
- `events.jsonl` contains a valid JSON line with `type = "task_created"`.
- The user response includes a CLI-visible grill-me clarification question with a recommended answer.
- Clarifying cannot complete until `brief.md` exists and the human has said no adjustment is needed or adjustment is complete.
