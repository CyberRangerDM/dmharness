# AGENTS.md

harness-dm 是一套关于使用 Claude Code 或者 Codex 的工作流和方法论. 本文件只提供知识的导航索引, 你在需要的时候按需去获取.

## 目录结构
```text
.dm/
├── agents/           # 各类 agent 定义
├── specs/            # 规范, 原则类 spec 文件
├── skills/           # skill 文件
├── hooks/            # hooks 预留目录; 第一阶段 Guardrail deferred
├── tasks/            # 任务状态持久化
├── design/           # 设计内容持久化
├── guardrails/       # Guardrail 策略占位; 第一阶段 disabled
└── session/          # 会话状态
```

## 工作流定义
工作流的定义在文件 .dm/workflow.md 中

<!-- DM-WORKFLOW:START -->
## DM Workflow Commands

Codex CLI does not register project-defined `/dm:*` slash commands in phase 1. In Codex, use the `dm` skill invocation form instead:

- `$dm start [title]`
- `$dm continue [task-id]`
- `$dm status [task-id]`
- `$dm feedback [task-id] [text]`

When the user types one of these exact phrases, treat it as a Harness-DM workflow command, not a shell command. Claude Code may use the project slash-command equivalents:

- `/dm-start [title]`
- `/dm-continue [task-id]`
- `/dm-status [task-id]`
- `/dm-feedback [task-id] [text]`

Execute these commands by reading and writing the `.dm` file protocol according to `.dm/workflow.md`, `.dm/commands/*.md`, `.dm/agents/*.md`, `.dm/templates/*`, and `specs/*.spec.md`.

For phase handoff, treat `.dm/tasks/[task-id]/brief.md` as the clarifying output and `.dm/design/[task-id]/design.md` as the design output. Users may edit either file directly before continuing; reread the file instead of relying only on conversation context.

During `clarifying`, the agent must complete at least three meaningful CLI-visible interactive clarification rounds before designing. Each round must have a pending point that materially affects requirements, at least 3 options, and `[用户手动填入]`. Record every answered round in `brief.md`; do not advance to designing unless `brief.md` contains at least three answered records and no key ambiguity remains. Do not ask filler questions only to increase the count.

During `designing`, apply the same interaction standard to `design.md`: complete at least three meaningful CLI-visible interactive design confirmation rounds before `design_review`. Each round must materially affect architecture, approach, scope slicing, tradeoffs, validation, acceptance, rollout, or risk. Record every answered round in `design.md`; do not advance to `design_review` unless `design.md` contains at least three answered design confirmation records and the draft is complete. Do not ask filler questions only to increase the count.

Do not invoke an external `harness-dm` CLI. Do not override Codex built-in `/status`, `/review`, `/diff`, `/plan`, or `/agent`.

Codex adapter notes:

- `$dm continue` is the Codex workflow gate command. It maps to the logical `/dm:continue` operation and does not replace Codex native `/status`, `/review`, `/diff`, or `/plan`.
- Do not instruct Codex users to type `/dm:start`; current Codex CLI versions reject unknown slash commands before the message reaches the agent.
- If platform capability is insufficient, record the limitation in task state or report and ask the human for confirmation.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.
<!-- DM-WORKFLOW:END -->
