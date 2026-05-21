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

`/dm-continue [task-id]` 仅保留为旧任务兼容或异常恢复入口，不属于 clarify 后的常规推进流程。

这些命令分别映射到逻辑命令:

- `/dm:start [title]`
- `/dm:continue [task-id]`

`/dm:continue [task-id]` 仅用于上述兼容/恢复场景。

Codex CLI 不注册项目自定义 `/dm:*` slash commands; 在 Codex 中使用 `dm` skill invocation form:

- `$dm start [title]`
- `$dm continue [task-id]`

`$dm continue [task-id]` 同样仅用于旧任务兼容或异常恢复。

当用户输入上述 Claude Code 命令时, 包括兼容/恢复用的 continue 命令, 将其视为 Harness-DM 工作流命令, 不是 shell 命令. 从对应的 `.dm/commands/[command].md` 开始, 再按当前 phase 读取必要产物、特定模板或角色定义; 正常操作不要一次性读取全部 workflow、commands、agents、templates 或 specs.

For phase handoff, treat `.dm/tasks/[task-id]/brief.md` as the clarifying output and `.dm/design/[task-id]/design.md` as the design output. Users may edit either file directly; inspect the latest compact summary, status markers, and required records instead of relying only on conversation context. Read the full file when compact data is missing, inconsistent, or the exact design content is being confirmed or handed off.

During `clarifying`, the agent must complete at least three meaningful CLI-visible interactive clarification rounds before designing. Each round must have a pending point that materially affects requirements, at least 3 options, and `[用户手动填入]`. Do not update `brief.md` after every answer; keep answered rounds in the current clarify working set and write `brief.md` once after interactive clarification is complete. Do not advance to designing unless the final `brief.md` contains at least three answered records and no key ambiguity remains. Do not ask filler questions only to increase the count.

Avoid repeated long-form output in LLM interactions. If the exact same file, report, design section, or long artifact has already been output unchanged, show it fully only the first time; later references should use the path, heading, record id, compact summary, or content hash unless exact full text is required.

After `clarifying` is complete, do not ask for a separate human continue confirmation. Main Agent immediately enters `designing`, writes the actual design to `design.md`, moves through `design_review` as an automatic validation/persistence step, and continues through Worker/Test/Accept until `done`, `blocked`, or an internal rework loop. `designing` is not an interactive discussion phase and does not require design confirmation rounds.

Only `clarifying` requires interactive discussion with the human. Clarifying discussion must follow `specs/grill-me-discussion.spec.md`: ask one main question at a time, walk the decision tree from upstream dependencies to downstream details, include the agent's recommended answer and reason, and do not ask the human for facts that can be discovered from the codebase or `.dm` files. Other phases should proceed autonomously unless blocked by missing artifacts, platform capability limits, or an explicit rework request in the current session.

Claude Code adapter notes:

- `/dm-continue` maps to the logical `/dm:continue` operation only for legacy/recovery cases.
- Normal tasks started with `/dm-start` must not require `/dm-continue` after clarify; `design_review` is handled automatically by Main Agent.
- Do not invoke an external `harness-dm` CLI.
- Do not override Claude Code built-in commands.
- If platform capability is insufficient, record the limitation in task state or report and ask the human for confirmation.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.
<!-- DM-WORKFLOW:END -->
