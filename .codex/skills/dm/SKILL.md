---
name: dm
description: Use when the user asks for $dm start, $dm continue, $dm status, $dm feedback, or asks to run the Harness-DM workflow in Codex.
metadata:
  short-description: Run Harness-DM workflow
  version: 1.2.0
---

# DM Workflow Skill

Use this skill when the user asks for `$dm start`, `$dm continue`, `$dm status`, `$dm feedback`, or asks to run the Harness-DM workflow.

Codex CLI phase 1 entrypoint:

- `$dm start [title]`
- `$dm continue [task-id]`
- `$dm status [task-id]`
- `$dm feedback [task-id] [text]`

Do not tell Codex users to type `/dm:*`; current Codex CLI versions reject unknown slash commands before the prompt reaches the agent. The `$dm ...` forms map to the logical `/dm:*` operations documented in `.dm/commands/`.

## Reading Policy

Keep the workflow file protocol as the source of truth, but avoid loading the whole `.dm` knowledge base on every turn.

Skill body injection is needed only once per Codex session. After the skill has been loaded, later DM turns should retain only the skill identity metadata in active context when possible:

- skill id/name: `dm`
- skill version: `1.2.0`
- skill path: `.codex/skills/dm/SKILL.md`

Always read only the minimal command entrypoint first:

| User phrase | Required command file |
|---|---|
| `$dm start [title]` | `.dm/commands/dm-start.md` |
| `$dm continue [task-id]` | `.dm/commands/dm-continue.md` |
| `$dm status [task-id]` | `.dm/commands/dm-status.md` |
| `$dm feedback [task-id] [text]` | `.dm/commands/dm-feedback.md` |

Then read phase-specific artifacts required by that command. Prefer compact summary sections, status markers, `state.json`, and targeted `rg`/`jq` checks over full-file reads. Read full workflow/spec/template/role files only when a gate is ambiguous, a required compact summary is missing or stale, a file was directly edited by the human, or the current phase explicitly needs the full artifact for handoff.

For templates, read only the template being instantiated. Do not read `.dm/templates/*` as a group.

If an unchanged file, report, design section, or other long artifact has already been fully shown in the current LLM interaction chain, do not show it verbatim again. Refer to it by path, heading, confirmation id, short summary, or content hash unless exact full text is required for the current gate.

## Operating Rules

- Do not invoke an external or self-developed `harness-dm` CLI.
- Operate through `.dm` file protocol only.
- Treat `$dm ...` phrases as workflow commands, not shell commands.
- If a legacy `/dm:*` phrase is present in a non-interactive prompt that reaches the agent, map it to the corresponding logical operation; do not recommend it for interactive Codex.
- Use `.dm/templates` when creating task, design, report, feedback, and summary files, reading only the specific template needed.
- Treat `.dm/tasks/[task-id]/brief.md` as the clarifying phase output and reread its compact summary and confirmation records before design.
- During `clarifying`, complete at least three meaningful CLI-visible interactive clarification rounds before design. Each round must have a pending point, at least 3 options, and `[用户手动填入]`; keep answered rounds in the clarify working set and write `brief.md` once after interactive clarification is complete. Do not ask filler questions only to increase the count.
- Treat `.dm/design/[task-id]/design.md` as the design phase output. Reread its compact summary and decision/status sections before design review, confirmation, and Worker/Test/Accept handoff; read the full file when confirming the exact design or handing off implementation.
- `designing` is autonomous: Main Agent writes the design from the latest `brief.md` without interactive design confirmation rounds, then moves to `design_review` and asks the human to approve the design.
- Main Agent owns phase transitions and `state.json.phase`.
- `$dm continue` advances by workflow segment: from `clarifying`, Main Agent may enter `designing`, complete design, and stop at `design_review`; from `design_review`, human approval lets Main Agent run the remaining phases automatically until `done`, `blocked`, or feedback is required.
- If required artifacts are missing, report missing items and do not advance.
- If `state.json` is missing or malformed, stop and do not infer task state from Markdown alone.
- Never overwrite numbered files such as `feedback-001.md`, `worker-result-001.md`, `test-report-001.md`, or `accept-report-001.md`.
- `$dm status` is read-only and must not write `command-log.md`.
- Test and Accept are read-only for business code in phase 1; this is instruction-level enforcement.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.

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
