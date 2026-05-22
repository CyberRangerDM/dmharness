# 适配层 Spec

## 1. 范围

Adapters 负责把同一套 `.dm` 文件协议接入 Claude Code 与 OpenAI Codex 的原生 CLI 使用方式。适配层不得要求用户离开 Claude Code / Codex CLI 去操作自研 CLI。

## 2. 共享协议

Claude Code 和 Codex 必须共享：

- `.dm/workflow.md`
- `.dm/commands/`
- `.dm/agents/`
- `.dm/tasks/`
- `.dm/design/`
- `.dm/session/`
- `.dm/skills/grill-me.md`
- `.dm/specs/phase-controller.spec.md`
- `.dm/specs/persistence.spec.md`
- `.dm/specs/software-language-policy.spec.md`

平台差异只存在于入口形式、agent/subagent 声明方式、权限能力和指令承载方式。

## 3. 命令入口

| 逻辑命令 | Codex 入口 | Claude Code 入口 | 含义 |
|---|---|---|---|
| `/dm:start [title]` | `$dm start [title]` | `/dm-start [title]` | 创建任务 |
| `/dm:continue [task-id]` | `$dm continue [task-id]` | `/dm-continue [task-id]` | 会话恢复时从 `.dm` 文件判断完成状态并继续未完成任务 |

Rules:

- `/dm:*` 是平台无关逻辑命令名。
- Claude Code 使用普通项目命令 `/dm-*`。
- Codex 使用 `$dm ...` skill invocation；不得把 `/dm:*` 作为 Codex 交互式入口。
- `$dm continue` / `/dm-continue` 只用于会话恢复，不是常规阶段门禁。

## 4. Codex Adapter

Codex adapter 由以下文件承载：

- `AGENTS.md`
- `.codex/skills/dm/SKILL.md`
- `.codex/skills/grill-me/SKILL.md`
- `.codex/config-fragment.md`

Codex rules:

- 先读命令对应的 `.dm/commands/*.md`，再按 phase 读取必要 artifact。
- 不覆盖 Codex 原生 `/status`、`/review`、`/diff`、`/plan`、`/agent`。
- `/review` 可辅助 Accept phase，但结果必须写入 `accept-report-[n].md`。
- `/diff` 可辅助生成 `.dm/session/[task-id]/summary.md`。

## 5. Claude Code Adapter

Claude Code adapter 使用项目命令和角色文件承载同一协议：

- `.claude/commands/dm-start.md`
- `.claude/commands/dm-continue.md`
- `.claude/agents/dm-worker.md`
- `.claude/agents/dm-test.md`
- `.claude/agents/dm-accept.md`

`/dm-continue` must:

- Treat the command as logical `/dm:continue`.
- Reconstruct context from `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`.
- Report completion if the task is already complete.
- Resume from the first incomplete required phase if incomplete.

## 6. Cross-Platform Consistency

- Clarifying must follow active grill-me behavior on both platforms. Codex should load `.codex/skills/grill-me/SKILL.md` when available; Claude Code and shared protocol references use `.dm/skills/grill-me.md`.
- Software task language selection must follow `.dm/specs/software-language-policy.spec.md` on both platforms.
- `brief.md` and `design.md` are file-level handoff artifacts; later phases must reread them instead of relying on conversation context.
- Human may edit `brief.md` or `design.md`; phase gates must inspect the latest file.
- Worker/Test/Accept must use persisted `.dm` artifacts, not platform conversation history.
- Same task id must remain portable between Claude Code and Codex by reading the same `.dm/tasks/[task-id]/state.json`.
- Platform capability gaps must be recorded in task state or report and surfaced to the human.

## 7. Acceptance Criteria

- Adapters do not require a self-developed CLI.
- Codex triggers DM through `AGENTS.md` + `dm` skill.
- Claude Code triggers DM through project commands.
- Both platforms read and write the same `.dm` protocol.
- Codex native command semantics are not overridden.
