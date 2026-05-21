# Harness-DM Agent Workflow

> 通过主 Agent 协调顺序执行的 Worker / Test / Accept Sub Agent，完成从需求分析到最终交付的流程。

## 基本信息

| 字段 | 内容 |
|------|------|
| 版本 | v1.1 |
| 适用 Agent | Claude Code / OpenAI Codex |
| 触发方式 | Claude Code project commands / Codex `$dm` skill invocation |
| 参与角色 | Main Agent、Worker Sub Agent、Test Sub Agent、Accept Sub Agent、Human |
| 第一阶段限制 | 不开发独立 CLI；Guardrail Engine 暂不启用实际拦截 |

## 目标

由 Main Agent 驱动，通过与用户协作确认需求、自动设计、按阶段持久化状态、顺序调度 Worker / Test / Accept 角色，最终产出已通过自动交付检查并可供人类审阅的交付物。

本工作流的唯一共享事实来源是项目内文件协议：

- `.dm/tasks/`
- `.dm/design/`
- `.dm/session/`
- `.dm/specs/`
- `specs/`

所有 clarify 阶段的 AI Agent 与 Human 对话式讨论，还必须遵循 `specs/grill-me-discussion.spec.md`。

## Context Budget Policy

Harness-DM 的流程语义依赖 `.dm` 文件协议，但 Agent 不应在每个命令中重复加载全部 workflow、spec、agent、command 和 template 文件。默认读取策略是：

- 先读当前命令文件、`state.json`、`summary.md` 和当前 phase 的源产物。
- 优先读取 `DM Compact Summary`、状态 marker、结果行、确认记录表和必要锚点。
- 用 `jq` 检查 JSON 状态，用 `rg` 定位 Markdown 记录；避免对大型 Markdown、报告或业务文件做宽范围全文输出。
- 只有在摘要缺失、marker 不一致、Human 直接编辑导致无法判断、门禁异常、或 Worker/Test/Accept handoff 需要完整依据时，才读取完整源文件或相关 spec。
- 模板按需读取；创建什么文件只读对应模板，不读取 `.dm/templates/*`。
- 同一份完全相同的文件内容、报告内容或长段设计内容，在一次 LLM 交互链路中只完整输出一次。后续如果需要再次引用，只输出路径、章节锚点、记录 ID、简短摘要或内容 hash；只有当内容已经变化、首次输出不在当前上下文、或当前动作必须验证精确全文时，才再次输出完整内容。

这条策略不改变 `brief.md`、`design.md`、report 和 session summary 作为事实来源的地位。它只约束 Agent 如何高效读取这些事实来源。

## Phase 状态机

| Phase | 含义 | 执行者 | 推进方式 |
|---|---|---|---|
| `idle` | 尚未创建任务 | Human / Main Agent | 逻辑 `/dm:start`；Codex 用 `$dm start`；Claude Code 用 `/dm-start` |
| `clarifying` | 通过讨论和文件编辑形成需求简报 | Main Agent + Human | Clarify 完成后由 Main Agent 自动进入 `designing` |
| `designing` | 根据最新需求简报生成设计草案 | Main Agent | 自动进入 `design_review` |
| `design_review` | 等待人类确认当前 `design.md` | Human | 逻辑 `/dm:continue`；Codex 用 `$dm continue`；Claude Code 用 `/dm-continue` |
| `design_persisted` | 当前 `design.md` 已被确认 | Main Agent | 自动进入 `working` |
| `working` | Worker 执行任务 | Worker Sub Agent | 自动进入 `testing` |
| `testing` | Test 只读验证 | Test Sub Agent | 自动进入 `accepting` 或回流 `working` |
| `accepting` | Accept 只读交付前检查 | Accept Sub Agent | 自动进入 `human_acceptance` 或回流 `working` |
| `human_acceptance` | 汇总变更并自动完成 | Main Agent / Human | 自动完成为 `done`；反馈用 `$dm feedback` / `/dm-feedback` |
| `done` | 任务完成 | Main Agent | 无 |
| `blocked` | 缺少决策、能力不支持或外部阻塞 | Main Agent / Human | 人类反馈 |

说明：

- Claude Code 第一阶段接受 `/dm-continue` 作为 `/dm:continue` 的等价命令。
- Codex 第一阶段使用 `$dm continue` 作为 `/dm:continue` 的等价入口；不要在交互式 Codex 中输入 `/dm:*`，因为当前 CLI 会在消息到达 Agent 前拒绝未知 slash command。
- Main Agent 必须在 `clarifying` 阶段完成至少三轮有意义的 CLI 内交互式澄清。澄清过程中不得每轮都读写 `brief.md`；确认记录先保留在当前对话/内存工作集，等 Human 交互确认完毕、关键歧义消失后，一次性写入最终 `brief.md`，然后才可以判断 `brief.md` 已足够清晰并自动进入设计。
- “有意义的澄清轮次”必须影响需求事实，例如目标、范围、非目标、约束、验收标准、风险、优先级或关键边界；禁止为了凑轮次提出无实质影响的问题。
- 每轮澄清还必须遵循 grill-me 规范：一次只问一个主要待确认点，沿决策树逐个解决依赖，为每个问题提供 Main Agent 推荐答案和推荐理由；能通过读取代码库或 `.dm` 文件协议查明的问题，不得转嫁给 Human。
- `designing` 阶段不再进行互动式设计确认。Main Agent 必须基于最新 `brief.md`、项目文件和相关规范自主完成实际设计，写入 `design.md`，然后进入 `design_review` 并提醒 Human 审批。
- 除 `clarifying` 外，其他阶段默认不与 Human 进行互动式交流；遇到缺失产物、平台能力不足、报告失败或明确反馈时，记录阻塞或反馈并给出下一步。
- `brief.md` 和 `design.md` 是跨阶段交接的源文件。`brief.md` 在 clarify 完成时一次性形成，`design.md` 在 designing 完成时形成。后续 phase 必须重新读取对应文件的 compact summary、状态 marker 和必要记录，不得只依赖对话上下文；在确认设计或交付实现前读取完整设计内容。

## 流程步骤

### Step 1 - 初始化适配

- 执行者：Main Agent
- 操作：根据目标平台生成或更新 Claude Code / Codex 原生适配文件。
- 完成条件：项目具备 `.dm` 基础目录，以及对应平台可读取的工作流说明。

第一阶段不提供 `harness-dm init` CLI。

### Step 2 - 用户输入需求

- 执行者：Human
- 操作：向 Main Agent 描述任务目标或需求。
- 完成条件：Main Agent 创建任务目录并进入 `clarifying`。

### Step 3 - 需求讨论与细节确认

- 执行者：Main Agent + Human
- 操作：Main Agent 通过多轮对话收集确认记录，双方对齐任务边界与预期。澄清未完成前，不要在每一轮后更新 `.dm/tasks/[task-id]/brief.md`；确认记录和中间判断保留在当前对话/内存工作集，直到满足 clarify 完成条件后一次性生成最终 `.dm/tasks/[task-id]/brief.md`。
- 强制交互：
  - Main Agent 必须在 Claude Code 或 Codex 的 CLI 对话中完成至少三个“待确认点”的有意义澄清轮次。
  - 每个待确认点必须提供 3 个或以上候选方案，并提供一个用户手动填入选项。
  - 每轮只能包含一个主要待确认点；必须提供 Main Agent 的推荐答案、推荐理由、决策影响和前置依赖。
  - 提问前必须先读取可用项目文件、`.dm` 产物和相关 spec；能自行查明的事实不得询问 Human。
  - 对话格式必须类似：
    ```text
    对于待确认点A，有多个方案:
    1. aaa
    2. bbb
    3. ccc
    4. [用户手动填入]

    推荐答案: 2
    推荐理由: ...
    决策影响: ...
    前置依赖: ...
    ```
  - Human 选择或手动输入后，Main Agent 必须把待确认点、需求影响、候选方案、推荐答案、推荐理由、前置依赖、探索依据、用户选择、最终取值保留在当前 clarify 工作集中；不要每轮写入 `brief.md`。
  - 如果已回答澄清轮次少于三轮，必须继续提出下一轮同样格式的有意义确认问题。
  - 如果已达到三轮但仍有关键不确定点，继续提出下一轮同样格式的确认问题，直到关键歧义消失。
  - 如果 Main Agent 找不到新的有意义待确认点，不得提出无意义问题；应说明当前无法继续澄清的原因，并请 Human 提供补充或确认是否存在遗漏需求。
- 支持方式：
  - 直接澄清：围绕缺失字段和歧义提问。
  - 发散探索：提供多个目标拆分、范围取舍、风险优先、极简/完整方案、反例边界等思考方向。
  - 用户自定义输入：Human 可在对话中给出任意补充；如果 Human 明确要求直接编辑 `brief.md`，该文件可作为人工例外提前创建或修改，Main Agent 后续必须把它视为外部编辑并在完成前合并。
- 完成条件：Main Agent 确认当前 clarify 工作集中至少有三条已回答的有意义且 grill-me 合规的交互式确认记录，且核心目标、范围、约束和关键歧义状态足以进入设计；随后一次性写入 `.dm/tasks/[task-id]/brief.md`，并只做必要的目标性检查。满足后不再等待 Human 确认，直接进入 Step 4。

### Step 4 - 输出设计方案

- 执行者：Main Agent
- 操作：Main Agent 必须先读取最新 `brief.md`，再读取完成设计所需的项目文件、相关 spec 和既有 `.dm` 产物，自主生成 `.dm/design/[task-id]/design.md`。
- 设计要求：
  - `design.md` 必须包含目标、需求映射、范围、非目标、推荐方案、备选方案取舍、实施计划、预期变更文件、验证计划、验收标准和风险。
  - Main Agent 不向 Human 提出设计待确认点，不要求三轮设计确认记录，也不要求 Human 在 design 阶段输入 continue。
  - 如设计所需事实可通过代码库或 `.dm` 文件查明，Main Agent 必须自行读取；如无法完成设计，应进入 `blocked` 或在 `summary.md` 记录明确缺口，而不是开启互动式设计确认。
  - Human 可在 `design_review` 期间直接编辑 `design.md` 或通过反馈要求修改。
- 完成条件：`.dm/design/[task-id]/design.md` 存在，Main Agent 判断当前文件内容已形成完整设计草案，更新 phase 为 `design_review`，提醒 Human 审批设计并在同意后输入对应平台的 continue 命令。

### Step 5 - 用户确认方案

- 执行者：Human
- 操作：审阅并可直接编辑 `.dm/design/[task-id]/design.md`；同意时输入对应平台的 continue 命令，修改意见使用 feedback 命令或直接编辑设计文件后要求重做设计。
- 完成条件：Human 输入对应平台的 continue 命令。

### Step 6 - 确认当前设计文件

- 执行者：Main Agent
- 操作：重新读取 `.dm/design/[task-id]/design.md`，将当前文件内容视为被确认的设计，写入或更新 `.dm/design/[task-id]/decisions.md` 与 `.dm/design/[task-id]/revisions.md`。
- 完成条件：`design.md` 和 `decisions.md` 存在，自动进入 `working` 并继续执行后续阶段。

确认前允许写入和修改 `design.md` 作为设计草案。进入 `working` 后，Worker/Test/Accept 必须以 `design.md` 当前确认版本为准。修改已确认设计时，必须回到 `design_review` 并重新等待对应平台的 continue 命令。

### Step 7 - Worker 执行任务

- 执行者：Worker Sub Agent
- 操作：依据确认后的设计执行任务。
- 完成条件：生成 `.dm/tasks/[task-id]/worker-result-[n].md`。

### Step 8 - Test / Accept / Feedback 循环

- 执行者：Main Agent 调度，Sub Agent 顺序执行。

流程：

1. Worker 完成后进入 `testing`。
2. Test Sub Agent 只读验证，生成 `.dm/tasks/[task-id]/test-report-[n].md`。
3. Test pass 后进入 `accepting`。
4. Accept Sub Agent 只读做交付前检查并尽可能模拟人类验收，生成 `.dm/tasks/[task-id]/accept-report-[n].md`。
5. Test 或 Accept fail 时，生成 `.dm/tasks/[task-id]/feedback-[n].md`，状态回流到 `working`。

约束：

- Test / Accept 只读业务代码。
- Test / Accept 只允许写对应 report 和 feedback 文件。
- 多 agent 第一阶段顺序执行，不做真实并行。

### Step 9 - 汇总变更

- 执行者：Main Agent
- 操作：生成 `.dm/session/[task-id]/summary.md`。
- 完成条件：summary 至少包含文件新增/修改/删除列表和设计决策变化。

人类可阅读 `summary.md` 文本内容进行事后审阅。`manifest.json` 第一阶段不强制生成。

### Step 10 - 自动完成或反馈回流

- 执行者：Main Agent / Human
- 通过：当 Worker/Test/Accept 都通过且 summary 已生成时，Main Agent 自动将任务置为 `done`。
- 未通过：Human 提供反馈，Main Agent 写入 `.dm/tasks/[task-id]/feedback-[n].md`，状态回流到 `working`。

## 持久化约定

每个任务目录：

```text
.dm/tasks/[task-id]/
├── state.json
├── events.jsonl
├── brief.md
├── summary.md
├── command-log.md
├── feedback-001.md
├── worker-result-001.md
├── test-report-001.md
└── accept-report-001.md
```

设计目录：

```text
.dm/design/[task-id]/
├── design.md
├── decisions.md
└── revisions.md
```

会话目录：

```text
.dm/session/[task-id]/
└── summary.md
```

## 目录约定

| 路径 | 用途 |
|------|------|
| `.dm/agents/` | 平台无关 agent 角色定义 |
| `.dm/specs/` | 已确认的原则与运行规范 |
| `.dm/skills/` | 可复用技能说明 |
| `.dm/hooks/` | 未来 hook 接入点 |
| `.dm/tasks/` | 任务状态、反馈与角色报告 |
| `.dm/design/` | 已确认设计方案和设计决策 |
| `.dm/session/` | 任务变更汇总，供人类审阅 |
| `.dm/guardrails/` | Guardrail 策略占位，第一阶段 disabled |
| `specs/` | 当前项目模块 Spec |

## Guardrail 状态

第一阶段 Guardrail Engine 为 `v0-deferred`：

- 不启用命令拦截。
- 不改变 Claude Code / Codex 原生权限策略。
- 不新增拦截命令执行的 hook。
- 其他模块不得依赖 Guardrail 的实际阻断能力。

## 完成标准

- 设计方案已持久化至 `.dm/design/[task-id]/`。
- `test` Sub Agent 报告通过。
- `accept` Sub Agent 报告通过。
- 变更摘要已写入 `.dm/session/[task-id]/summary.md`。
- 任务状态已自动进入 `done`，或失败/反馈已回流到 `working`。

## 相关资源

- [Phase Controller Spec](../specs/phase-controller.spec.md)
- [Persistence Spec](../specs/persistence.spec.md)
- [Adapters Spec](../specs/adapters.spec.md)
- [Guardrail Engine Spec](../specs/guardrail-engine.spec.md)
- [Grill-Me Discussion Spec](../specs/grill-me-discussion.spec.md)
