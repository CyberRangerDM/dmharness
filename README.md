# harness-dm

`harness-dm` 是一套面向 Claude Code 和 OpenAI Codex 的轻量交付工作流。

它不是独立 CLI，而是通过仓库内的 `.dm` 文件协议、平台适配文件和 agent/command 模板，把一次 AI 编程任务拆成可恢复、可审阅、可交付的流程。

## 核心目标

- clarify 阶段与人类充分澄清需求。
- clarify 完成后自动进入设计、实现、测试、验收和总结。
- 将任务状态、设计决策、角色报告和交付摘要持久化到仓库。
- 同时支持 Claude Code 与 Codex 的原生使用方式。

## 工作流

```text
clarifying
  -> designing
  -> design_review
  -> design_persisted
  -> working
  -> testing
  -> accepting
  -> human_acceptance
  -> done
```

只有 `clarifying` 阶段默认需要人机互动。需求简报 `brief.md` 确认后，Main Agent 会继续生成 `design.md`，并顺序调度 Worker / Test / Accept，直到完成、阻塞或进入内部返工循环。

## 使用入口

Claude Code:

```text
/dm-start [title]
/dm-continue [task-id]
```

Codex:

```text
$dm start [title]
$dm continue [task-id]
```

`continue` 只用于旧任务或中断会话恢复，不是 clarify 后的常规推进命令。Codex 中不要使用 `/dm:*` slash command。

## 目录

```text
.dm/workflow.md              # 工作流定义
.dm/commands/                # 逻辑命令说明
.dm/agents/                  # Main / Worker / Test / Accept 角色定义
.dm/templates/               # 任务、设计、报告模板
.dm/specs/                   # 规范和原则沉淀
.dm/tasks/[task-id]/         # 任务状态、brief、报告、返工记录
.dm/design/[task-id]/        # design、decisions、revisions
.dm/session/[task-id]/       # 面向人类的交付摘要

.claude/                     # Claude Code 适配入口
.codex/                      # Codex skill 与配置片段
AGENTS.md                    # Codex 项目指令入口
CLAUDE.md                    # Claude Code 项目指令入口
```

## 当前边界

- 第一阶段基于文件协议运行，不提供 `harness-dm` 可执行命令。
- 多 agent 角色按 Main Agent 调度顺序执行，不做真实并行。
- Test / Accept 只读验证业务代码，只写报告和内部返工记录。

完整协议见 `.dm/workflow.md`。
