# CLAUDE.md

harness-dm 是一套关于使用 Claude Code 或者 Codex 的工作流和方法论. 本文件提供给 Claude Code 读取, 只作为知识导航索引; 需要执行工作流时按需读取 `.dm/` 下的具体文件.

## 目录结构

```text
.dm/
├── agents/           # 各类 agent 定义
├── commands/         # DM 命令模板
├── specs/            # 规范, 原则类 spec 文件
├── skills/           # skill 文件
├── hooks/            # hooks 预留目录; 第一阶段 Guardrail deferred
├── tasks/            # 任务状态持久化
├── design/           # 设计内容持久化
├── guardrails/       # Guardrail 策略占位; 第一阶段 disabled
└── session/          # 会话状态
```

## 工作流定义

工作流定义在 `.dm/workflow.md` 中. Claude Code 执行 DM 命令时, 按 `.dm/commands/*.md` 的命令模板读取和写入 `.dm` 文件协议, 不调用外部 `harness-dm` CLI.

<!-- DM-WORKFLOW:START -->
## DM Workflow Commands

Claude Code 使用项目 slash-command 等价入口:

- `/dm-start [title]`
- `/dm-continue [task-id]`
- `/dm-status [task-id]`
- `/dm-feedback [task-id] [text]`

这些命令分别映射到逻辑命令:

- `/dm:start [title]`
- `/dm:continue [task-id]`
- `/dm:status [task-id]`
- `/dm:feedback [task-id] [text]`

Codex CLI 不注册项目自定义 `/dm:*` slash commands; 在 Codex 中使用 `dm` skill invocation form:

- `$dm start [title]`
- `$dm continue [task-id]`
- `$dm status [task-id]`
- `$dm feedback [task-id] [text]`

当用户输入上述 Claude Code 命令时, 将其视为 Harness-DM 工作流命令, 不是 shell 命令. 根据 `.dm/workflow.md`, `.dm/commands/*.md`, `.dm/agents/*.md`, `.dm/templates/*`, `.dm/specs/*.md` 和必要的项目 `specs/*.spec.md` 读写 `.dm` 文件协议.

For phase handoff, treat `.dm/tasks/[task-id]/brief.md` as the clarifying output and `.dm/design/[task-id]/design.md` as the design output. Users may edit either file directly before continuing; reread the file instead of relying only on conversation context.

During `clarifying`, the agent must complete at least three meaningful CLI-visible interactive clarification rounds before designing. Each round must have a pending point that materially affects requirements, at least 3 options, and `[用户手动填入]`. Record every answered round in `brief.md`; do not advance to designing unless `brief.md` contains at least three answered records and no key ambiguity remains. Do not ask filler questions only to increase the count.

During `designing`, apply the same interaction standard to `design.md`: complete at least three meaningful CLI-visible interactive design confirmation rounds before `design_review`. Each round must materially affect architecture, approach, scope slicing, tradeoffs, validation, acceptance, rollout, or risk. Record every answered round in `design.md`; do not advance to `design_review` unless `design.md` contains at least three answered design confirmation records and the draft is complete. Do not ask filler questions only to increase the count.

Claude Code adapter notes:

- `/dm-continue` is the Claude Code workflow gate command. It maps to the logical `/dm:continue` operation.
- Do not invoke an external `harness-dm` CLI.
- Do not override Claude Code built-in commands.
- If platform capability is insufficient, record the limitation in task state or report and ask the human for confirmation.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.
<!-- DM-WORKFLOW:END -->
