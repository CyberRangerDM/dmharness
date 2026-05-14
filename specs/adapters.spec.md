# 适配层（Claude Code adapter / Codex adapter）Spec

## 1. 范围

Adapters 负责把同一套 `.dm` 文件协议接入 Claude Code 与 OpenAI Codex 的原生 CLI 使用方式。适配层不得要求用户离开 Claude Code / Codex CLI 去操作自研 CLI。

## 2. 共同协议

两个 adapter 必须共享：

- `.dm/workflow.md`
- `.dm/tasks`
- `.dm/design`
- `.dm/specs`
- `.dm/session`
- phase 状态机
- worker/test/accept 顺序执行协议
- state + memory 文件格式
- `brief.md` 和 `design.md` 的文件级交接语义

平台差异只存在于：

- slash command / skill invocation 的注册方式。
- agent/subagent 的声明方式。
- hook/permission 能力。
- skills/plugins 的承载方式。

## 3. 输入定义

### 初始化输入

```text
adapter_target = "claude" | "codex" | "both"
project_root = repository root
```

### 运行输入

- 用户在 Claude Code / Codex CLI 中输入的工作流命令。
- `.dm` 文件协议。
- 平台原生配置文件。

### 支持的工作流命令

| 逻辑命令 | Codex 入口 | Claude Code 入口 | 含义 |
|---|---|---|---|
| `/dm:start [title]` | `$dm start [title]` | `/dm-start [title]` | 创建任务 |
| `/dm:continue [task-id]` | `$dm continue [task-id]` | `/dm-continue [task-id]` | 推进 phase |
| `/dm:status [task-id]` | `$dm status [task-id]` | `/dm-status [task-id]` | 查看任务状态 |
| `/dm:feedback [task-id] [text]` | `$dm feedback [task-id] [text]` | `/dm-feedback [task-id] [text]` | 写入反馈并回流 |

说明：

- `/dm:*` 是平台无关的逻辑命令名。
- Claude Code 普通项目命令使用 `/dm-*` 形式。
- Codex CLI 使用 `$dm ...` skill invocation 形式。当前 Codex CLI 会拒绝未注册的 `/dm:*` slash command，因此不得把 `/dm:start` 作为 Codex 交互式入口。

### 交互与文件编辑一致性

Claude Code 与 Codex adapter 都必须支持同一语义：

- `clarifying` 阶段必须通过至少三轮有意义 CLI 内交互式澄清，并可继续通过多轮交互、发散式讨论和 Human 自定义输入形成 `.dm/tasks/[task-id]/brief.md`。
- 每轮确认必须展示待确认点、至少 3 个候选方案，以及 `[用户手动填入]` 选项；Claude Code 和 Codex 的表现形式必须一致。
- 有意义的澄清轮次必须影响或确认目标、范围、非目标、约束、验收标准、风险、优先级或关键边界；不得为了凑轮次提出无实质影响的问题。
- Human 回答后，Agent 必须把每条确认记录写入 `brief.md`，包括待确认点、需求影响、候选方案、用户选择、最终取值和状态。
- Human 可以在进入下一 phase 前直接编辑 `brief.md`；continue 时 Agent 必须重新读取该文件。
- `designing` 阶段必须通过至少三轮有意义 CLI 内交互式设计确认，并可继续通过多轮交互、发散式讨论和 Human 自定义输入形成 `.dm/design/[task-id]/design.md`。
- 每轮设计确认必须展示设计待确认点、至少 3 个候选方案，以及 `[用户手动填入]` 选项；Claude Code 和 Codex 的表现形式必须一致。
- 有意义的设计确认轮次必须影响或确认架构、方案、范围切片、取舍、验证计划、验收标准、发布/回滚或风险；不得为了凑轮次提出无实质影响的问题。
- Human 回答后，Agent 必须把每条设计确认记录写入 `design.md`，包括设计待确认点、设计影响、候选方案、用户选择、最终取值和状态。
- Human 可以在设计确认或进入下一 phase 前直接编辑 `design.md`；continue、确认和 handoff 时 Agent 必须重新读取该文件。
- 后续 Worker/Test/Accept 不得只依赖 Claude Code 或 Codex 的对话上下文恢复 clarifying/design 产物。

## 4. 输出定义

### Claude Code adapter 输出

```text
.claude/
├── commands/
│   ├── dm-start.md
│   ├── dm-continue.md
│   ├── dm-status.md
│   └── dm-feedback.md
├── agents/
│   ├── dm-worker.md
│   ├── dm-test.md
│   └── dm-accept.md
```

如果后续采用 Claude Code plugin 形式，可额外输出：

```text
.claude/plugins/dm/
├── plugin.json
└── commands/
    ├── start.md
    ├── continue.md
    ├── status.md
    └── feedback.md
```

说明：

- Claude Code 项目 commands 使用 `/dm-continue`；语义上等价于 `/dm:continue`。

### Codex adapter 输出

```text
AGENTS.md
.codex/
├── skills/
│   └── dm/
│       └── SKILL.md
└── config-fragment.md
```

可选输出：

```text
.codex/plugins/dm/
└── plugin.json
```

说明：

- Codex adapter 第一阶段以 `AGENTS.md` + `$dm` skill 触发为主。
- 当前 Codex 交互式入口使用 `$dm start|continue|status|feedback`，映射到同名逻辑 `/dm:*` 操作。
- Codex 原生已有 `/plan`、`/review`、`/diff`、`/status`、`/agent` 等命令；adapter 不应覆盖这些命令。

## 5. Claude Code 接口设计

### Project command：`/dm-continue`

文件：`.claude/commands/dm-continue.md`

```md
---
description: Advance the current DM task by one phase if completion conditions are met.
argument-hint: [task-id]
---

Read AGENTS.md, .dm/workflow.md, specs/phase-controller.spec.md, and the current task state.
Treat this command as logical /dm:continue.
Advance at most one phase.
If required artifacts are missing, report missing items and do not advance.
```

### Subagent 定义

`dm-worker`：

```text
Purpose: Execute implementation work according to confirmed design.
Write access: allowed for task implementation.
Output: .dm/tasks/[task-id]/worker-result-[n].md
```

`dm-test`：

```text
Purpose: Run or reason through tests.
Write access: only .dm/tasks/[task-id]/test-report-[n].md and feedback files.
Output: pass/fail test report.
```

`dm-accept`：

```text
Purpose: Delivery review and simulated human acceptance.
Write access: only .dm/tasks/[task-id]/accept-report-[n].md and feedback files.
Output: pass/fail acceptance report.
```

### Hook 接入

第一阶段不启用 Guardrail hook。未来可使用：

- `PreToolUse`
- `PostToolUse`
- `UserPromptSubmit`
- `SubagentStart`
- `SubagentStop`

## 6. Codex 接口设计

### AGENTS.md 约定

在项目 `AGENTS.md` 中加入 DM 工作流索引：

```md
## DM Workflow Commands

In Codex, use the dm skill invocation form:

- $dm start [title]
- $dm continue [task-id]
- $dm status [task-id]
- $dm feedback [task-id] [text]

These are workflow commands, not shell commands. Execute them by reading and writing .dm files according to specs/. Do not tell Codex users to type `/dm:*`; unknown slash commands are rejected before they reach the agent.
```

### Skill：`.codex/skills/dm/SKILL.md`

```md
# DM Workflow Skill

Use this skill when the user asks for $dm start, $dm continue, $dm status, $dm feedback, or asks to run the Harness DM workflow.

Read:
- AGENTS.md
- .dm/workflow.md
- specs/phase-controller.spec.md
- specs/persistence.spec.md

Do not invoke an external harness-dm CLI.
Operate through .dm file protocol only.
```

### Codex 原生命令协作

| Codex 命令 | 用法 |
|---|---|
| `/plan` | 可用于进入设计/计划阶段，但不替代 `$dm continue` |
| `/review` | 可辅助 accept 阶段，但结果必须写入 accept report |
| `/diff` | 可辅助 session summary 的文件变化列表 |
| `/status` | 查看 Codex 会话状态，不等价于 `$dm status` |
| `/agent` | 如启用 subagent，可用于查看或切换 agent thread |

## 7. 文件结构

```text
.
├── AGENTS.md
├── specs/
│   ├── phase-controller.spec.md
│   ├── guardrail-engine.spec.md
│   ├── persistence.spec.md
│   └── adapters.spec.md
├── .dm/
│   ├── workflow.md
│   ├── tasks/
│   ├── design/
│   ├── specs/
│   └── session/
├── .claude/
│   ├── commands/
│   └── agents/
└── .codex/
    └── skills/
```

## 8. 初始化行为

不提供 `harness-dm init` CLI。初始化由 Agent 在 Claude/Codex 会话中按 adapter spec 写文件完成。

### Claude adapter setup

必须生成或更新：

- `.claude/commands`
- `.claude/agents`

不得删除已有 `.dm` 内容。

### Codex adapter setup

必须生成或更新：

- `AGENTS.md` 中的 DM workflow 指令块。
- `.codex/skills/dm/SKILL.md`

不得删除已有 `.dm` 内容。

### Both adapters setup

顺序完成 Claude adapter setup 与 Codex adapter setup，保留已有文件内容，只追加或更新 DM 标记区块。

## 9. 验收标准

- 适配层不得要求用户运行自研 CLI。
- Claude adapter 必须能在 Claude Code 原生 CLI 中触发 DM 工作流命令。
- Codex adapter 必须能在 Codex 原生 CLI 中通过 AGENTS/skill 触发同一套 DM 文件协议。
- 同一个任务在 Claude 与 Codex 间切换时，必须继续读取同一 `.dm/tasks/[task-id]/state.json`。
- 初始化 Claude 与 Codex adapter 时，不得删除已有 `.dm`、`.claude`、`.codex`、`AGENTS.md` 内容。
- worker/test/accept 的角色说明必须在两个平台都存在。
- Codex adapter 不得覆盖 Codex 内置 `/status`、`/review`、`/diff` 等命令语义。
- 当平台能力不一致时，Agent 必须记录到任务状态或报告中，并交给人类手动确认。

## 10. 已确认限制与取舍

- Codex 第一阶段使用 `$dm` skill invocation 触发语义，不依赖项目级自定义 `/dm:*` slash command。
- Claude Code 接受普通 project command `/dm-continue`，不强制实现严格的 `/dm:continue` 命名空间命令。
- Guardrail hook 第一阶段不启用，因此 adapter 暂不承担危险操作拦截。

## 11. 参考来源

- Claude Code 支持项目自定义 slash commands、命令 frontmatter、文件引用和 bash 上下文。
- Claude Code 支持 subagents，且 subagent 可拥有独立上下文和工具配置。
- Claude Code hooks 提供 `PreToolUse`、`PostToolUse`、`UserPromptSubmit` 等接入点。
- Codex CLI 提供内置 slash commands，包括 `/plan`、`/review`、`/diff`、`/status`、`/agent` 等；自定义 skill 在当前适配中使用 `$skill` 形式触发。
- Codex 通过 `AGENTS.md` 读取项目指令，并按目录层级合并指导。
