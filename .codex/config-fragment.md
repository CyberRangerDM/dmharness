# Codex Adapter Configuration Fragment

Harness-DM 在 Codex 中通过项目指令和 `dm` skill 运行，不提供独立 CLI，也不注册 `/dm:*` slash command。

## Activation

Codex should read:

- `AGENTS.md`
- `.codex/skills/dm/SKILL.md`
- `.codex/skills/grill-me/SKILL.md` when the workflow is in `clarifying`

Workflow triggers:

- `$dm start [title]`
- `$dm continue [task-id]`

These phrases are workflow triggers, not shell commands. Do not tell Codex users to type `/dm:*`; current Codex CLI versions reject unknown slash commands before they reach the agent.

## Rules

- Operate through the shared `.dm` file protocol.
- Do not invoke a self-developed `harness-dm` CLI.
- Do not override Codex native `/status`, `/review`, `/diff`, `/plan`, or `/agent`.
- Do not install Guardrail hooks in phase 1.
- If platform capability is insufficient, record it in task state or report and ask the human for confirmation.

## Recovery

`$dm continue [task-id]` is only for session recovery. Codex reads `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`, reports completion if the task is already done, and resumes from the first incomplete required phase otherwise.
