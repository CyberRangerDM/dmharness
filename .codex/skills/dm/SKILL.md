---
name: dm
description: Use when the user asks for $dm start, $dm continue, $dm status, $dm feedback, or asks to run the Harness-DM workflow in Codex.
metadata:
  short-description: Run Harness-DM workflow
---

# DM Workflow Skill

Use this skill when the user asks for `$dm start`, `$dm continue`, `$dm status`, `$dm feedback`, or asks to run the Harness-DM workflow.

Codex CLI phase 1 entrypoint:

- `$dm start [title]`
- `$dm continue [task-id]`
- `$dm status [task-id]`
- `$dm feedback [task-id] [text]`

Do not tell Codex users to type `/dm:*`; current Codex CLI versions reject unknown slash commands before the prompt reaches the agent. The `$dm ...` forms map to the logical `/dm:*` operations documented in `.dm/commands/`.

## Required Reading

Read these files before acting:

- `AGENTS.md`
- `.dm/workflow.md`
- `.dm/commands/dm-start.md`
- `.dm/commands/dm-continue.md`
- `.dm/commands/dm-status.md`
- `.dm/commands/dm-feedback.md`
- `.dm/agents/main.md`
- `.dm/agents/worker.md`
- `.dm/agents/test.md`
- `.dm/agents/accept.md`
- `.dm/templates/*`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`
- `specs/adapters.spec.md`

## Operating Rules

- Do not invoke an external or self-developed `harness-dm` CLI.
- Operate through `.dm` file protocol only.
- Treat `$dm ...` phrases as workflow commands, not shell commands.
- If a legacy `/dm:*` phrase is present in a non-interactive prompt that reaches the agent, map it to the corresponding logical operation; do not recommend it for interactive Codex.
- Use `.dm/templates` when creating task, design, report, feedback, and summary files.
- Treat `.dm/tasks/[task-id]/brief.md` as the clarifying phase output and reread it before design.
- During `clarifying`, complete at least three meaningful CLI-visible interactive clarification rounds before design. Each round must have a pending point, at least 3 options, and `[用户手动填入]`; record every answered round in `brief.md`. Do not ask filler questions only to increase the count.
- Treat `.dm/design/[task-id]/design.md` as the design phase output and reread it before design review, confirmation, and Worker/Test/Accept handoff.
- During `designing`, complete at least three meaningful CLI-visible interactive design confirmation rounds before design review. Each round must have a design pending point, at least 3 options, and `[用户手动填入]`; record every answered round in `design.md`. Do not ask filler questions only to increase the count.
- Main Agent owns phase transitions and `state.json.phase`.
- Advance at most one phase per `$dm continue`.
- If required artifacts are missing, report missing items and do not advance.
- If `state.json` is missing or malformed, stop and do not infer task state from Markdown alone.
- Never overwrite numbered files such as `feedback-001.md`, `worker-result-001.md`, `test-report-001.md`, or `accept-report-001.md`.
- `$dm status` is read-only and must not write `command-log.md`.
- Test and Accept are read-only for business code in phase 1; this is instruction-level enforcement.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.

## Command Mapping

| User phrase | Source template |
|---|---|
| `$dm start [title]` | `.dm/commands/dm-start.md` |
| `$dm continue [task-id]` | `.dm/commands/dm-continue.md` |
| `$dm status [task-id]` | `.dm/commands/dm-status.md` |
| `$dm feedback [task-id] [text]` | `.dm/commands/dm-feedback.md` |

## Role Mapping

| Role | Source definition |
|---|---|
| Main | `.dm/agents/main.md` |
| Worker | `.dm/agents/worker.md` |
| Test | `.dm/agents/test.md` |
| Accept | `.dm/agents/accept.md` |

Main stays in the current Codex session. Worker/Test/Accept follow `.dm/agents/*.md`. If no real subagent is available, Main may simulate the handoff and must record the platform limitation where relevant.

## Native Codex Command Coordination

- `/plan` may help planning but does not replace `$dm continue`.
- `/review` may help the Accept phase, but results must be written to `accept-report-[n].md`.
- `/diff` may help build `.dm/session/[task-id]/summary.md`.
- `/status` is Codex session status and is not the same as `$dm status`.
- `/agent` may help manage sub agent threads if available, but `.dm` files remain the source of truth.

## Platform Limitations

If Codex cannot provide a feature required by a spec, record the limitation in the task report or `summary.md`, then ask the human for confirmation before proceeding.
