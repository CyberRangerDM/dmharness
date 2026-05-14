# Codex Adapter Configuration Fragment

This project uses Harness-DM through Codex native CLI behavior.

The Codex adapter is instruction/skill based.

## Activation

Codex should read `AGENTS.md` and this skill:

- `.codex/skills/dm/SKILL.md`

The workflow is triggered by Codex skill invocations:

- `$dm start [title]`
- `$dm continue [task-id]`
- `$dm status [task-id]`
- `$dm feedback [task-id] [text]`

These phrases are workflow triggers, not registered shell commands.

Do not use `/dm:*` in interactive Codex. Current Codex CLI versions reject unknown slash commands before they reach the agent, so `/dm:start` fails with `Unrecognized command`.

## Source Of Truth

Use the shared file protocol:

- `.dm/workflow.md`
- `.dm/commands/`
- `.dm/agents/`
- `.dm/templates/`
- `.dm/tasks/`
- `.dm/design/`
- `.dm/session/`
- `specs/`

## Constraints

- Do not invoke a self-developed `harness-dm` CLI.
- Do not override Codex native `/status`, `/review`, `/diff`, `/plan`, or `/agent`.
- Do not install Guardrail hooks in phase 1.
- If platform capability is insufficient, record it and ask the human for confirmation.

## Shared State

Codex and Claude Code share the same task state:

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- `.dm/tasks/[task-id]/brief.md`
- `.dm/design/[task-id]/design.md`
- `.dm/session/[task-id]/summary.md`

`brief.md` is the clarifying output and `design.md` is the design output. Users may edit either file directly before continuing; Codex must reread the file at phase gates.

Clarifying must include at least three meaningful CLI-visible confirmation menus. Each menu needs a pending point, 3 or more options, and `[用户手动填入]`. Every answered confirmation must be recorded in `brief.md` before Codex advances to design, and filler questions solely for increasing the count are forbidden.

Designing must include at least three meaningful CLI-visible design confirmation menus. Each menu needs a design pending point, 3 or more options, and `[用户手动填入]`. Every answered confirmation must be recorded in `design.md` before Codex advances to design review, and filler questions solely for increasing the count are forbidden.

## Phase 1 Limitations

- No Guardrail hook.
- No path-level write enforcement for Test/Accept.
- No self-developed CLI.
