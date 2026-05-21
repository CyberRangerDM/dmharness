# 流程控制模块（Phase 状态机）Spec

## 1. 范围

Phase Controller 负责把一次任务从需求输入推进到最终验收完成。它不作为独立 CLI 存在，而是在 Claude Code / OpenAI Codex 原生 CLI 内，通过 slash command、skill invocation、项目指令和文件协议执行。

本模块只控制阶段、状态、角色顺序和阶段门禁；不负责具体代码实现、不负责危险命令拦截。

## 2. 设计约束

- 目录统一使用 `.dm/tasks` 与 `.dm/specs`。
- task id 格式为：`YYYYMMDDHHmm-xxxxxxxx`，其中 `xxxxxxxx` 为 8 位 uuid/随机 id。
- 多 agent 顺序执行，不做真实并行。
- `review` 与 `accept` 合并为一个角色：`accept`，负责交付前检查并尽可能模拟人类验收。
- `test` 与 `accept` 只读；发现问题只输出报告和反馈，不直接改代码。
- 人类只负责 clarify 阶段需求澄清和 design_review 阶段设计审批；设计、执行、测试、验收检查和完成收尾默认由 Agent 自动推进。场景固定且可用规则判断的阶段条件使用确定性检查，其余使用 LLM 判断。
- 修改已确认设计时，必须重新经过逻辑 `/dm:continue` 确认；Codex 中对应 `$dm continue`。
- Codex 第一阶段使用 `$dm` skill invocation，不依赖项目级自定义 `/dm:*` slash command。
- Main Agent 必须在 `clarifying` 阶段完成至少三轮有意义的 CLI 内交互式澄清。澄清过程中确认记录保留在当前 clarify 工作集中，不每轮读写 `brief.md`；等交互确认完毕且关键歧义消失后，一次性写入最终 `brief.md`，然后才允许判断 `brief.md` 已足够清晰并自动进入设计。
- 有意义的澄清轮次必须影响或确认需求事实，例如目标、范围、非目标、约束、验收标准、风险、优先级或关键边界；禁止为了凑轮次提出无实质影响的问题。
- `designing` 阶段不进行互动式设计确认。Main Agent 必须根据最新 `brief.md`、项目文件和 `.dm` 产物自主生成完整 `design.md`，然后进入 `design_review` 等待人类审批。
- `brief.md` 和 `design.md` 是 phase 交接的文件级事实来源。`brief.md` 在 clarify 完成时一次性形成；后续 phase 必须读取这些文件，不得只依赖对话上下文。
- 文件级事实来源允许低上下文读取：phase gate 优先读取 compact summary、状态 marker 和相关确认记录；当摘要缺失、不一致、Human 直接编辑无法判断、确认设计或 handoff 需要完整内容时，必须读取完整文件。
- 所有 clarify 对话式澄清必须遵循 `specs/grill-me-discussion.spec.md`：一次只问一个主要问题，按决策树逐个解决依赖，每个问题包含 Agent 推荐答案和推荐理由，且能通过代码库或 `.dm` 文件协议自行查明的问题不得询问 Human。其他阶段默认不做互动式交流，除非记录 blocker 或处理明确 feedback。

## 3. Phase 定义

| Phase | 含义 | 执行者 | 下一步触发 |
|---|---|---|---|
| `idle` | 尚未创建任务 | Human / Main Agent | `/dm:start` 逻辑命令 |
| `clarifying` | 通过讨论形成需求简报，并在完成时一次性写入 `brief.md` | Main Agent + Human | 自动进入 `designing` |
| `designing` | 根据需求简报生成设计草案 | Main Agent | 自动进入 `design_review` |
| `design_review` | 等待人类确认当前 `design.md` | Human | `/dm:continue` 逻辑命令 |
| `design_persisted` | 当前 `design.md` 已被确认 | Main Agent | 自动进入 worker |
| `working` | worker 执行任务 | Worker Agent | worker report |
| `testing` | test 只读验证 | Test Agent | test report |
| `accepting` | accept 只读验收 | Accept Agent | accept report |
| `human_acceptance` | 汇总变更并自动完成 | Main Agent / Human | 自动进入 `done` 或通过反馈回流 |
| `done` | 任务完成 | Main Agent | 无 |
| `blocked` | 缺少决策、能力不支持或外部阻塞 | Human / Main Agent | 人类反馈 |

## 4. 状态转移

```text
idle
  -> clarifying
  -> designing
  -> design_review
  -> design_persisted
  -> working
  -> testing
  -> accepting
  -> human_acceptance
  -> done
```

失败循环：

```text
testing failed -> working
accepting failed -> working
human_acceptance failed -> working
```

设计变更循环：

```text
design_review requested_changes -> designing
confirmed design changed -> design_review
```

阻塞：

```text
any phase -> blocked
blocked -> previous phase or clarifying
```

## 5. 输入定义

### 命令输入

| 逻辑命令 | Codex 入口 | Claude Code 入口 | 含义 |
|---|---|---|---|
| `/dm:start` | `$dm start [task-title]` | `/dm-start [task-title]` | 创建任务并进入 `clarifying` |
| `/dm:continue` | `$dm continue [task-id]` | `/dm-continue [task-id]` | 推进当前任务的下一个自动工作流片段 |
| `/dm:status` | `$dm status [task-id]` | `/dm-status [task-id]` | 展示当前任务状态 |
| `/dm:feedback` | `$dm feedback [task-id] [feedback]` | `/dm-feedback [task-id] [feedback]` | 写入人类反馈并回流 |

说明：

- Claude Code 优先通过项目 slash command 或插件命令实现。
- Codex 通过 `$dm` skill invocation 映射为同一套文件协议；详见 `adapters.spec.md`。

### 文件输入

- `.dm/workflow.md`
- `.dm/target.md`
- `AGENTS.md`
- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/brief.md`，作为 clarifying phase 的正式产物和 designing phase 的输入
- `.dm/design/[task-id]/design.md`，作为 designing phase 的正式产物和后续 implementation/test/accept 的输入
- `.dm/design/[task-id]/decisions.md`
- `.dm/tasks/[task-id]/worker-result-[n].md`
- `.dm/tasks/[task-id]/test-report-[n].md`
- `.dm/tasks/[task-id]/accept-report-[n].md`

## 6. 输出定义

### 状态输出

- 更新 `.dm/tasks/[task-id]/state.json`
- 追加 `.dm/tasks/[task-id]/events.jsonl`
- 更新 `.dm/tasks/[task-id]/summary.md`

### 阶段产物

| Phase | 必须产物 |
|---|---|
| `clarifying` | `.dm/tasks/[task-id]/brief.md` |
| `designing` | `.dm/design/[task-id]/design.md` |
| `design_review` | 等待人类确认的当前 `.dm/design/[task-id]/design.md` |
| `design_persisted` | 已确认的 `.dm/design/[task-id]/design.md`、`.dm/design/[task-id]/decisions.md` |
| `working` | `.dm/tasks/[task-id]/worker-result-[n].md` |
| `testing` | `.dm/tasks/[task-id]/test-report-[n].md` |
| `accepting` | `.dm/tasks/[task-id]/accept-report-[n].md` |
| `human_acceptance` | `.dm/session/[task-id]/summary.md` |
| `done` | `state.json.status = "done"` |

## 7. 文件结构

```text
.dm/
├── tasks/
│   └── [task-id]/
│       ├── state.json
│       ├── events.jsonl
│       ├── brief.md          # clarify 完成时一次性写入，或由 Human 提前编辑
│       ├── summary.md
│       ├── feedback-001.md
│       ├── worker-result-001.md
│       ├── test-report-001.md
│       └── accept-report-001.md
├── design/
│   └── [task-id]/
│       ├── design.md
│       ├── decisions.md
│       └── revisions.md
└── session/
    └── [task-id]/
        └── summary.md
```

## 8. 接口设计

本模块的接口是 Agent 内部操作契约，不要求实现独立 CLI。

```ts
type Phase =
  | "idle"
  | "clarifying"
  | "designing"
  | "design_review"
  | "design_persisted"
  | "working"
  | "testing"
  | "accepting"
  | "human_acceptance"
  | "done"
  | "blocked";

type TransitionResult = {
  ok: boolean;
  from: Phase;
  to?: Phase;
  reason: string;
  missing?: string[];
};

function createTask(title: string, now: string): TaskState;
function loadTask(taskId?: string): TaskState;
function canContinue(state: TaskState): TransitionResult;
function transition(state: TaskState, event: string): TransitionResult;
function appendEvent(taskId: string, event: TaskEvent): void;
function writePhaseSummary(taskId: string, phase: Phase, summary: string): void;
```

### `/dm:continue` 逻辑行为

```text
Input:
  Logical /dm:continue [task-id]
  Codex: $dm continue [task-id]
  Claude Code: /dm-continue [task-id]

Behavior:
  1. 读取当前任务 state.json。
  2. 如果当前 phase 是 clarifying，优先检查当前 clarify 工作集是否已满足至少三条已回答的有意义且 grill-me 合规的交互式确认记录；满足时一次性写入最终 brief.md。若 brief.md 已存在，则检查其 compact summary 和确认记录，必要时读取全文。
  3. 如果当前 phase 是 designing，读取最新 brief.md 和必要项目文件，生成或更新完整 design.md；不得要求互动式设计确认。
  4. 如果当前 phase 是 design_review，重新读取 design.md；确认当前设计时读取完整文件内容。
  5. 检查当前 phase 的可推进条件。
  6. 如果条件满足，更新 state.json.phase。
  7. 追加 events.jsonl。
  8. 告知用户已进入的新 phase 和下一步动作。
  9. 如果条件不满足，列出缺失项，不推进。
```

## 9. Phase 完成条件

| Phase | 可推进条件 |
|---|---|
| `clarifying` | 最终 `brief.md` 已一次性写入，包含至少三条已回答的有意义且 grill-me 合规的交互式确认记录，且 Main Agent 判断文件内容无关键歧义 |
| `designing` | 最新 `brief.md` 足以支撑设计，且 Main Agent 已生成完整 `design.md` |
| `design_review` | 人类明确输入对应平台的 continue 命令，以确认当前 `design.md` |
| `design_persisted` | `design.md` 与 `decisions.md` 存在 |
| `working` | 最新 `worker-result-[n].md` 存在 |
| `testing` | 最新 `test-report-[n].md` 存在且结论为 pass |
| `accepting` | 最新 `accept-report-[n].md` 存在且结论为 pass |
| `human_acceptance` | `.dm/session/[task-id]/summary.md` 已生成，且 Worker/Test/Accept 均通过 |

## 10. 验收标准

- 创建任务时，100% 生成符合 `YYYYMMDDHHmm-xxxxxxxx` 格式的 task id。
- 逻辑 `/dm:continue` 按工作流片段推进：clarifying 通过后可自动生成设计并停在 design_review；design_review 审批后可自动跑完剩余阶段直到 done、blocked 或需要反馈。
- 任一 phase 缺少必须产物时，continue 命令不得推进状态。
- `test` 或 `accept` 失败时，必须生成 `feedback-[n].md` 并回流到 `working`。
- `test` 与 `accept` 阶段不得修改业务代码；只允许写对应报告与反馈文件。
- `brief.md` 必须由至少三轮有意义 CLI 内交互式澄清、多轮交互、发散讨论和用户自定义输入共同形成，并允许人类在进入下一 phase 前直接编辑；每轮澄清必须遵循 grill-me 规范，包含推荐答案、推荐理由、决策影响、前置依赖和探索依据。Agent 不得每轮澄清后更新 `brief.md`，必须在 clarify 完成条件满足后一次性写入最终 `brief.md`。
- clarifying 阶段必须在 Claude Code 或 Codex CLI 内向 Human 展示至少三个有意义待确认点，每个待确认点至少包含 3 个候选方案和 1 个 `[用户手动填入]` 选项。
- Human 回答后，Agent 先在当前 clarify 工作集中记录待确认点、需求影响、候选方案、推荐答案、推荐理由、前置依赖、探索依据、用户选择、最终取值和状态；最终 `brief.md` 必须一次性包含这些记录。少于三条已回答的有意义且 grill-me 合规的确认记录时不得从 `clarifying` 推进到 `designing`。
- 达到三轮后，如果关键歧义仍然存在，Main Agent 必须继续澄清；如果没有新的有意义待确认点，不得提出 filler 问题，应请求 Human 补充上下文或确认遗漏。
- `design.md` 必须由 Main Agent 根据最新 `brief.md`、项目文件、相关 spec 和 `.dm` 产物自主形成；设计中应记录备选方案、取舍、实施计划、验证计划、验收标准和风险。
- designing 阶段不得要求 Human 完成三轮设计确认，也不得为了设计细节开启互动式确认菜单。
- 如果缺少完成设计所需的关键事实，Main Agent 必须记录 blocker 或明确缺失项，不得用 filler 问题代替设计工作。
- 从 `clarifying` 进入 `designing` 时必须读取最新 `brief.md`。
- 从 `designing` 进入 `design_review`、从 `design_review` 进入 `design_persisted`、以及 Worker/Test/Accept handoff 时必须读取最新 `design.md`。
- `design.md` 可以在确认前作为草案写入；`decisions.md` 在人类确认当前 `design.md` 后写入或更新。
- 已确认设计发生变更时，状态必须回到 `design_review`，等待新的 continue 命令。
- 每次 phase 转移必须追加一行 `events.jsonl`。
- 任务完成时，`.dm/session/[task-id]/summary.md` 必须包含文件新增/修改/删除列表与设计决策变化。

## 11. 已确认决策

- 接受 Codex adapter 使用 `$dm` skill invocation 作为等价触发语义。
- 接受 Main Agent 在 `clarifying` 阶段主动判断需求已充分清晰并自动进入设计，前提是至少三条有意义且 grill-me 合规的交互式确认记录已经完成，并已一次性写入最终 `brief.md`，且没有关键歧义遗留。
- 接受 Main Agent 在 `designing` 阶段主动生成完整 `design.md` 并进入 `design_review`，提示用户审批；不再要求设计确认记录。
- 接受 `design.md` 在 `designing` 阶段作为可编辑草案存在，后续确认的是文件当前内容。
- 接受 `specs/grill-me-discussion.spec.md` 作为 clarify 对话式讨论的质量门禁；三轮确认门禁仅保留在 clarifying 阶段，可计入门禁的记录必须包含推荐答案、推荐理由和自行探索依据。
