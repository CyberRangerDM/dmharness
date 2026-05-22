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

`$dm continue [task-id]` is a session recovery command for old or interrupted tasks; it is not part of the normal post-clarify flow. On resume, read `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]` to decide whether the task is complete; if it is incomplete, reconstruct task context from those files and continue from the first incomplete required phase.

When the user types one of these phrases, including the session recovery continue phrase, treat it as a Harness-DM workflow command, not a shell command. Claude Code may use the project slash-command equivalents:

- `/dm-start [title]`
- `/dm-continue [task-id]`

`/dm-continue [task-id]` has the same session recovery semantics in Claude Code.

Execute these commands by reading and writing the `.dm` file protocol. Start from the specific `.dm/commands/[command].md` file, then read only the current phase artifacts and specific templates or role definitions needed for that action. Do not load all workflow, command, agent, template, or spec files during normal operation.

For phase handoff, treat `.dm/tasks/[task-id]/brief.md` as the clarifying output and `.dm/design/[task-id]/design.md` as the design output. Users may edit either file directly; inspect the latest compact summary, status markers, and required records instead of relying only on conversation context. Read the full file when compact data is missing, inconsistent, or the exact design content is being confirmed or handed off.

During `clarifying`, the agent must complete at least three meaningful CLI-visible interactive clarification rounds before designing. Each round must have a pending point that materially affects requirements, at least 3 options, and `[用户手动填入]`. Do not update `brief.md` after every answer; keep answered rounds in the current clarify working set and write `brief.md` once after interactive clarification is complete. Do not advance to designing unless the final `brief.md` contains at least three answered records and no key ambiguity remains. Do not ask filler questions only to increase the count.

Avoid repeated long-form output in LLM interactions. If the exact same file, report, design section, or long artifact has already been output unchanged, show it fully only the first time; later references should use the path, heading, record id, compact summary, or content hash unless exact full text is required.

After `clarifying` is complete, do not ask for a separate human continue confirmation. Main Agent immediately enters `designing`, writes the actual design to `design.md`, moves through `design_review` as an automatic validation/persistence step, and continues through Worker/Test/Accept until `done`, `blocked`, or an internal rework loop. `designing` is not an interactive discussion phase and does not require design confirmation rounds.

Only `clarifying` requires interactive discussion with the human. Clarifying discussion must follow `.dm/specs/grill-me-discussion.spec.md`: ask one main question at a time, walk the decision tree from upstream dependencies to downstream details, include the agent's recommended answer and reason, and do not ask the human for facts that can be discovered from the codebase or `.dm` files. Other phases should proceed autonomously unless blocked by missing artifacts, platform capability limits, or an explicit rework request in the current session.

Do not invoke an external `harness-dm` CLI. Do not override Codex built-in `/status`, `/review`, `/diff`, `/plan`, or `/agent`.

Codex adapter notes:

- `$dm continue` maps to the logical `/dm:continue` operation for session recovery and does not replace Codex native `/status`, `/review`, `/diff`, or `/plan`.
- `$dm continue` is session recovery only. Normal tasks started with `$dm start` must not require it after clarify; `design_review` is handled automatically by Main Agent.
- Do not instruct Codex users to type `/dm:start`; current Codex CLI versions reject unknown slash commands before the message reaches the agent.
- If platform capability is insufficient, record the limitation in task state or report and ask the human for confirmation.
- Guardrail Engine is deferred in phase 1; do not install custom blocking hooks.
<!-- DM-WORKFLOW:END -->
