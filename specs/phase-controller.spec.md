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
- 人类只负责最终结果与验收；场景固定且可用规则判断的阶段条件使用确定性检查，其余使用 LLM 判断。
- 修改已确认设计时，必须重新经过逻辑 `/dm:continue` 确认；Codex 中对应 `$dm continue`。
- Codex 第一阶段使用 `$dm` skill invocation，不依赖项目级自定义 `/dm:*` slash command。
- Main Agent 必须在 `clarifying` 阶段完成至少三轮有意义的 CLI 内交互式澄清，并把确认记录写入 `brief.md`，然后才允许判断 `brief.md` 已足够清晰并提示用户输入对应平台的 continue 命令。
- 有意义的澄清轮次必须影响或确认需求事实，例如目标、范围、非目标、约束、验收标准、风险、优先级或关键边界；禁止为了凑轮次提出无实质影响的问题。
- Main Agent 必须在 `designing` 阶段完成至少三轮有意义的 CLI 内交互式设计确认，并把确认记录写入 `design.md`，然后才允许判断 `design.md` 已形成完整设计草案并提示用户输入对应平台的 continue 命令。
- 有意义的设计确认轮次必须影响或确认设计事实，例如架构、方案、范围切片、取舍、验证计划、验收标准、发布/回滚或风险；禁止为了凑轮次提出无实质影响的问题。
- `brief.md` 和 `design.md` 是 phase 交接的文件级事实来源。后续 phase 必须读取这些文件，不得只依赖对话上下文。

## 3. Phase 定义

| Phase | 含义 | 执行者 | 下一步触发 |
|---|---|---|---|
| `idle` | 尚未创建任务 | Human / Main Agent | `/dm:start` 逻辑命令 |
| `clarifying` | 通过讨论和文件编辑形成需求简报 | Main Agent + Human | `/dm:continue` 逻辑命令 |
| `designing` | 通过讨论和文件编辑形成设计草案 | Main Agent + Human | `/dm:continue` 逻辑命令 |
| `design_review` | 等待人类确认当前 `design.md` | Human | `/dm:continue` 逻辑命令 |
| `design_persisted` | 当前 `design.md` 已被确认 | Main Agent | 自动进入 worker |
| `working` | worker 执行任务 | Worker Agent | worker report |
| `testing` | test 只读验证 | Test Agent | test report |
| `accepting` | accept 只读验收 | Accept Agent | accept report |
| `human_acceptance` | 汇总变更，等待人类最终验收 | Human | `/dm:continue` 逻辑命令或反馈 |
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
| `/dm:continue` | `$dm continue [task-id]` | `/dm-continue [task-id]` | 尝试推进当前任务到下一 phase |
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
│       ├── brief.md
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
  2. 如果当前 phase 是 clarifying，重新读取 brief.md，并检查是否存在至少三条已回答的有意义交互式确认记录。
  3. 如果当前 phase 是 designing，重新读取 design.md，并检查是否存在至少三条已回答的有意义设计确认记录。
  4. 如果当前 phase 是 design_review，重新读取 design.md。
  5. 检查当前 phase 的可推进条件。
  6. 如果条件满足，更新 state.json.phase。
  7. 追加 events.jsonl。
  8. 告知用户已进入的新 phase 和下一步动作。
  9. 如果条件不满足，列出缺失项，不推进。
```

## 9. Phase 完成条件

| Phase | 可推进条件 |
|---|---|
| `clarifying` | 最新 `brief.md` 存在，包含至少三条已回答的有意义交互式确认记录，且 Main Agent 判断文件内容无关键歧义 |
| `designing` | 最新 `design.md` 存在，包含至少三条已回答的有意义设计确认记录，且 Main Agent 判断文件内容是完整设计草案 |
| `design_review` | 人类明确输入对应平台的 continue 命令，以确认当前 `design.md` |
| `design_persisted` | `design.md` 与 `decisions.md` 存在 |
| `working` | 最新 `worker-result-[n].md` 存在 |
| `testing` | 最新 `test-report-[n].md` 存在且结论为 pass |
| `accepting` | 最新 `accept-report-[n].md` 存在且结论为 pass |
| `human_acceptance` | 人类明确输入对应平台的 continue 命令 |

## 10. 验收标准

- 创建任务时，100% 生成符合 `YYYYMMDDHHmm-xxxxxxxx` 格式的 task id。
- 逻辑 `/dm:continue` 每次最多推进 1 个 phase。
- 任一 phase 缺少必须产物时，continue 命令不得推进状态。
- `test` 或 `accept` 失败时，必须生成 `feedback-[n].md` 并回流到 `working`。
- `test` 与 `accept` 阶段不得修改业务代码；只允许写对应报告与反馈文件。
- `brief.md` 必须由至少三轮有意义 CLI 内交互式澄清、多轮交互、发散讨论和用户自定义输入共同形成，并允许人类在进入下一 phase 前直接编辑。
- clarifying 阶段必须在 Claude Code 或 Codex CLI 内向 Human 展示至少三个有意义待确认点，每个待确认点至少包含 3 个候选方案和 1 个 `[用户手动填入]` 选项。
- Human 回答后，`brief.md` 必须记录待确认点、需求影响、候选方案、用户选择、最终取值和状态；少于三条已回答的有意义确认记录时不得从 `clarifying` 推进到 `designing`。
- 达到三轮后，如果关键歧义仍然存在，Main Agent 必须继续澄清；如果没有新的有意义待确认点，不得提出 filler 问题，应请求 Human 补充上下文或确认遗漏。
- `design.md` 必须由至少三轮有意义 CLI 内交互式设计确认、多轮交互、发散讨论和用户自定义输入共同形成，并允许人类在进入下一 phase 前直接编辑。
- designing 阶段必须在 Claude Code 或 Codex CLI 内向 Human 展示至少三个有意义设计待确认点，每个待确认点至少包含 3 个候选方案和 1 个 `[用户手动填入]` 选项。
- Human 回答后，`design.md` 必须记录设计待确认点、设计影响、候选方案、用户选择、最终取值和状态；少于三条已回答的有意义设计确认记录时不得从 `designing` 推进到 `design_review`。
- 达到三轮后，如果关键设计歧义仍然存在，Main Agent 必须继续确认；如果没有新的有意义设计待确认点，不得提出 filler 问题，应请求 Human 补充上下文或确认遗漏设计约束。
- 从 `clarifying` 进入 `designing` 时必须读取最新 `brief.md`。
- 从 `designing` 进入 `design_review`、从 `design_review` 进入 `design_persisted`、以及 Worker/Test/Accept handoff 时必须读取最新 `design.md`。
- `design.md` 可以在确认前作为草案写入；`decisions.md` 在人类确认当前 `design.md` 后写入或更新。
- 已确认设计发生变更时，状态必须回到 `design_review`，等待新的 continue 命令。
- 每次 phase 转移必须追加一行 `events.jsonl`。
- 任务完成时，`.dm/session/[task-id]/summary.md` 必须包含文件新增/修改/删除列表与设计决策变化。

## 11. 已确认决策

- 接受 Codex adapter 使用 `$dm` skill invocation 作为等价触发语义。
- 接受 Main Agent 在 `clarifying` 阶段主动判断 `brief.md` 已充分清晰，并提示用户输入对应平台的 continue 命令，但前提是至少三条有意义交互式确认记录已经写入 `brief.md`，且没有关键歧义遗留。
- 接受 Main Agent 在 `designing` 阶段主动判断 `design.md` 已形成完整设计草案，并提示用户输入对应平台的 continue 命令，但前提是至少三条有意义设计确认记录已经写入 `design.md`，且没有关键设计歧义遗留。
- 接受 `design.md` 在 `designing` 阶段作为可编辑草案存在，后续确认的是文件当前内容。
