# 持久化模块（State + Memory）Spec

## 1. 范围

Persistence 模块负责把任务状态、设计决策、反馈、报告和会话摘要持久化到 `.dm` 目录。它不依赖独立 CLI，所有读写由 Claude Code / Codex Agent 按文件协议完成。

## 2. 设计原则

- 机器状态与人类可读内容分离。
- `brief.md` 和 `design.md` 是可由 Agent 与 Human 共同编辑的阶段产物。
- `brief.md` 必须记录 clarifying 阶段至少三条已回答的有意义 CLI 内交互式确认。
- `design.md` 必须记录 designing 阶段至少三条已回答的有意义 CLI 内交互式设计确认。
- `design.md` 可在 `designing` 阶段作为草案写入；人类确认后，当前文件内容成为实现依据。
- 任务可在新会话中通过 `.dm/tasks/[task-id]/summary.md` 和 `state.json` 恢复。
- 事件历史追加写入，避免只靠最新摘要恢复。
- 第一阶段不考虑 migration。
- 第一阶段不实现上下文耗尽治理。

## 3. 文件结构

```text
.dm/
├── tasks/
│   └── [task-id]/
│       ├── state.json
│       ├── events.jsonl
│       ├── brief.md
│       ├── summary.md
│       ├── command-log.md
│       ├── feedback-001.md
│       ├── worker-result-001.md
│       ├── test-report-001.md
│       └── accept-report-001.md
├── design/
│   └── [task-id]/
│       ├── design.md
│       ├── decisions.md
│       └── revisions.md
├── specs/
│   └── proposals/
└── session/
    └── [task-id]/
        ├── summary.md
        └── manifest.json  # optional
```

## 4. 输入定义

### 任务输入

- 用户原始需求。
- 需求澄清结果，最终以 `.dm/tasks/[task-id]/brief.md` 为准，其中必须包含至少三条已回答的有意义交互式确认记录。
- 设计讨论结果，最终以 `.dm/design/[task-id]/design.md` 为准，其中必须包含至少三条已回答的有意义设计确认记录。
- phase controller 当前状态。
- worker/test/accept agent 报告。
- 人类反馈。
- git/file 变化清单。
- 设计决策变化。
- 命令执行记录。

### 文件输入

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/events.jsonl`
- `.dm/tasks/[task-id]/brief.md`
- `.dm/design/[task-id]/design.md`
- `.dm/design/[task-id]/decisions.md`
- `.dm/session/[task-id]/summary.md`

## 5. 输出定义

### `state.json`

```json
{
  "schema_version": 1,
  "task_id": "202605121536-a1b2c3d4",
  "title": "task title",
  "status": "active",
  "phase": "working",
  "platform": "codex",
  "created_at": "2026-05-12T15:36:00+08:00",
  "updated_at": "2026-05-12T16:20:00+08:00",
  "iteration": 1,
  "roles": {
    "worker": {"status": "done", "latest_report": "worker-result-001.md"},
    "test": {"status": "pending", "latest_report": null},
    "accept": {"status": "pending", "latest_report": null}
  },
  "artifacts": {
    "brief": "brief.md",
    "design": "../../design/202605121536-a1b2c3d4/design.md",
    "decisions": "../../design/202605121536-a1b2c3d4/decisions.md",
    "session_summary": "../../session/202605121536-a1b2c3d4/summary.md"
  },
  "pending_human_decision": null
}
```

### `events.jsonl`

每行一个 JSON 对象：

```json
{"time":"2026-05-12T16:20:00+08:00","type":"phase_transition","from":"working","to":"testing","actor":"main","reason":"worker report created"}
```

### `summary.md`

面向 Agent 恢复上下文，必须包含：

- 当前 phase。
- 任务目标。
- 已确认需求边界。
- 已确认设计摘要。
- 最新 worker/test/accept 结果。
- 下一步动作。

### `.dm/session/[task-id]/summary.md`

面向人类最终验收，第一阶段必须包含：

- 文件新增/修改/删除列表。
- 设计决策变化。

后续可扩展：

- 命令记录。
- 测试结果。
- agent 报告摘要。
- 上下文摘要。

人类主要阅读 `summary.md` 文字内容即可；不要求第一阶段提供额外 UI 或状态视图。

### `.dm/session/[task-id]/manifest.json`

`manifest.json` 是可选的机器可读清单，用途是让后续 Agent 或工具快速读取任务产物列表，而不需要解析 Markdown。

它可记录：

- 本任务新增、修改、删除的文件路径。
- design、decisions、summary、report 等产物文件路径。
- 最新 phase、iteration、完成时间。

第一阶段不强制生成 `manifest.json`；只有当后续需要机器处理完成摘要、跨会话自动恢复或统计任务产物时再启用。

### `command-log.md`

记录命令执行历史，格式：

```md
## 2026-05-12T16:20:00+08:00

- Actor: main
- Command: `git status --short`
- Purpose: 查看工作区变更
- Result: success
```

## 6. 接口设计

接口是文件协议和 Agent 内部操作契约。

```ts
function initTaskStorage(taskId: string, title: string): void;
function readState(taskId: string): TaskState;
function writeState(taskId: string, state: TaskState): void;
function appendEvent(taskId: string, event: TaskEvent): void;
function writeBrief(taskId: string, markdown: string): void;
function writeDesignDraft(taskId: string, design: string): void;
function persistConfirmedDesign(taskId: string, decisions: string): void;
function writeAgentReport(taskId: string, role: "worker" | "test" | "accept", iteration: number, markdown: string): void;
function writeFeedback(taskId: string, iteration: number, markdown: string): void;
function recordCommand(taskId: string, entry: CommandLogEntry): void;
function writeSessionSummary(taskId: string, summary: string): void;
function writeManifest(taskId: string, manifest: SessionManifest): void; // optional in phase 1
```

## 7. 恢复流程

新会话恢复时，Agent 必须按顺序读取：

1. `AGENTS.md`
2. `.dm/workflow.md`
3. `.dm/tasks/[task-id]/state.json`
4. `.dm/tasks/[task-id]/summary.md`
5. `.dm/tasks/[task-id]/brief.md`
6. `.dm/design/[task-id]/design.md`
7. 最新 `worker/test/accept` 报告

如果未指定 task id，Agent 应读取 `.dm/tasks/*/state.json`，选择 `status != done` 且 `updated_at` 最新的任务。

## 8. 写入规则

- `state.json` 覆盖写入，代表当前事实。
- `events.jsonl` 只能追加，不覆盖。
- `.dm/tasks/[task-id]/brief.md` 可在 `clarifying` 阶段由 Agent 根据多轮讨论更新，也可由 Human 直接编辑；进入 `designing` 前必须重新读取，并检查至少三条已回答的有意义交互式确认记录。
- `brief.md` 的每条交互式确认记录必须至少包含：待确认点、需求影响、候选方案列表、用户选择或手动输入、最终取值、状态。能计入门禁的记录还必须影响或确认目标、范围、非目标、约束、验收标准、风险、优先级或关键边界。
- `.dm/design/[task-id]/design.md` 可在 `designing` 阶段由 Agent 根据多轮讨论更新，也可由 Human 直接编辑；进入 `design_review` 前必须重新读取，并检查至少三条已回答的有意义设计确认记录；进入 `design_persisted`、`working` 前也必须重新读取。
- `design.md` 的每条设计确认记录必须至少包含：设计待确认点、设计影响、候选方案列表、用户选择或手动输入、最终取值、状态。能计入门禁的记录还必须影响或确认架构、方案、范围切片、取舍、验证计划、验收标准、发布/回滚或风险。
- `.dm/design/[task-id]/decisions.md` 只在 Human 确认当前 `design.md` 后写入或更新。
- 修改已确认设计时，必须在 `revisions.md` 中记录变更，并回到 `design_review`。
- `feedback-[n].md`、`worker-result-[n].md`、`test-report-[n].md`、`accept-report-[n].md` 使用递增编号，不覆盖旧文件。
- `.dm/specs` 的正式 spec 修改必须人工确认；未确认内容写入 `.dm/specs/proposals`。

## 9. 验收标准

- 每个任务必须有且只有一个 `state.json`。
- `state.json.schema_version` 必须存在，第一阶段固定为 `1`。
- 每次 phase 变化必须追加一条 `events.jsonl`。
- 任一报告文件不得覆盖同名旧文件；编号必须递增。
- `brief.md` 是 clarifying phase 的正式产物，后续 phase 不得只依赖对话上下文恢复需求。
- `brief.md` 必须记录至少三条已回答的有意义交互式确认；少于三条时不得进入 `designing`。
- `design.md` 是 designing phase 的正式产物，后续 phase 不得只依赖对话上下文恢复设计。
- `design.md` 必须记录至少三条已回答的有意义设计确认；少于三条时不得进入 `design_review`。
- `design.md` 可作为确认前草案存在；`decisions.md` 在当前 `design.md` 被人类确认后写入或更新。
- 人类最终验收前，必须生成 `.dm/session/[task-id]/summary.md`。
- session summary 第一阶段必须包含文件新增/修改/删除列表和设计决策变化。
- 命令执行记录必须写入 `command-log.md`，至少包含时间、Actor、Command、Purpose、Result。
- 新会话只依赖 `.dm` 文件和 `AGENTS.md`，应能恢复当前任务 phase、下一步动作和最新反馈。

## 10. 已确认决策

- 人类主要阅读 `summary.md` 文字内容。
- `state.json + events.jsonl + summary.md` 作为第一阶段任务状态组合。
- `manifest.json` 第一阶段不强制生成；它只是机器可读产物清单，后续按需要启用。
