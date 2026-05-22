---
name: dm
description: Use when the user asks for $dm start, $dm continue, or asks to run the Harness-DM workflow in Codex.
metadata:
  short-description: Run Harness-DM workflow
  version: 1.2.0
---

# DM Workflow Skill

Use this skill when the user asks for `$dm start`, `$dm continue`, or asks to run the Harness-DM workflow.

Codex CLI phase 1 entrypoint:

- `$dm start [title]`
- `$dm continue [task-id]`

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

Then read phase-specific artifacts required by that command. Prefer compact summary sections, status markers, `state.json`, and targeted `rg`/`jq` checks over full-file reads. Read full workflow/spec/template/role files only when a gate is ambiguous, a required compact summary is missing or stale, a file was directly edited by the human, or the current phase explicitly needs the full artifact for handoff.

For templates, read only the template being instantiated. Do not read `.dm/templates/*` as a group.

If an unchanged file, report, design section, or other long artifact has already been fully shown in the current LLM interaction chain, do not show it verbatim again. Refer to it by path, heading, confirmation id, short summary, or content hash unless exact full text is required for the current gate.

## Operating Rules

- Do not invoke an external or self-developed `harness-dm` CLI.
- Operate through `.dm` file protocol only.
- Treat `$dm ...` phrases as workflow commands, not shell commands.
- If a legacy `/dm:*` phrase is present in a non-interactive prompt that reaches the agent, map it to the corresponding logical operation; do not recommend it for interactive Codex.
- Use `.dm/templates` when creating task, design, report, internal failure feedback, and summary files, reading only the specific template needed.
- Treat `.dm/tasks/[task-id]/brief.md` as the clarifying phase output and reread its compact summary and adjustment status before design.
- During `clarifying`, apply the project `grill-me` skill as the active discussion mode. Prefer `.codex/skills/grill-me/SKILL.md` when available; otherwise read `.dm/skills/grill-me.md`. This is not a loose citation: the visible conversation must behave like a grill-me interview.
- Clarifying only asks grill-me questions, writes the summarized requirement to `brief.md`, and asks whether `brief.md` needs adjustment. Advance only after the human says no adjustment is needed or adjustment is complete, then reread the latest `brief.md`.
- During ongoing `clarifying`, do not persist each question or answer to
  `.dm/tasks/[task-id]/events.jsonl`, and do not rewrite
  `.dm/tasks/[task-id]/summary.md` after every grill-me exchange. Keep
  intermediate answers in the live conversation, then batch the clarified
  decisions into `brief.md`, refresh `summary.md`, and append only required
  phase/blocker/completion events.
- Treat `.dm/design/[task-id]/design.md` as the design phase output. Reread its compact summary and decision/status sections before design review, confirmation, and Worker/Test/Accept handoff; read the full file when confirming the exact design or handing off implementation.
- `designing` is autonomous: Main Agent writes the design from the latest `brief.md` without interactive design confirmation rounds, then moves through `design_review` as an automatic validation/persistence step.
- Main Agent owns phase transitions and `state.json.phase`.
- `$dm continue` is session recovery only. On resume, reconstruct context from `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`, decide whether the task is already complete, and continue from the first incomplete required phase. Normal tasks started with `$dm start` must not require it after clarify; Main Agent runs design, automatic design review, Worker/Test/Accept, session summary, and `done` automatically until `blocked` or internal rework is required.
- If required artifacts are missing, report missing items and do not advance.
- If `state.json` is missing or malformed, stop and do not infer task state from Markdown alone.
- Never overwrite numbered files such as `feedback-001.md`, `worker-result-001.md`, `test-report-001.md`, or `accept-report-001.md`.
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

## Clarify Interaction Contract

Use this contract for every `clarifying` turn, including the first visible response after `$dm start`:

1. Load the grill-me instructions before composing the user-facing clarify prompt.
2. Inspect local files first for discoverable facts; ask the human only for choices, priorities, constraints, or intent that are not locally knowable.
3. Ask exactly one main question. The question can include brief context, but it must not contain multiple independent decisions.
4. Present the prompt in the user's language. Use an explicit question heading such as `Question 1:` or, in Chinese, `第 1 个问题：`.
5. Include the recommended answer immediately after the question, with the Main Agent's concrete recommendation. Localize the label when appropriate, for example `Recommended answer:` in English or `我的推荐答案：` in Chinese.
6. Pick questions by dependency order. For product/app requests, first resolve the primary user/audience and their job-to-be-done or success signal, unless the human already stated it; only then ask about target surface, scope boundaries, UX/API, data, and implementation details.
7. Keep Harness-DM metadata out of the visible clarify prompt unless it is needed to resolve a user question or blocker. Do not include task id, paths, command logs, or phase explanations before an ongoing grill-me question.
8. Continue the grill-me loop until the requirement is specific enough to write a design-ready `brief.md`.
9. Do not use `events.jsonl` or `summary.md` as per-question clarify scratchpads.
10. Only then write `brief.md`, refresh `summary.md` from the batched clarify summary, and ask whether that file needs adjustment.

## Native Codex Command Coordination

- `/plan` may help planning but does not replace Harness-DM phase files or automatic phase handling.
- `/review` may help the Accept phase, but results must be written to `accept-report-[n].md`.
- `/diff` may help build `.dm/session/[task-id]/summary.md`.
- `/status` is Codex session status and is not a Harness-DM command.
- `/agent` may help manage sub agent threads if available, but `.dm` files remain the source of truth.

## Platform Limitations

If Codex cannot provide a feature required by a spec, record the limitation in the task report or `summary.md`, then ask the human for confirmation before proceeding.
