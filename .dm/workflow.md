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

由 Main Agent 驱动，通过与用户协作确认需求、按阶段持久化状态、顺序调度 Worker / Test / Accept 角色，最终产出经人类验收通过的交付物。

本工作流的唯一共享事实来源是项目内文件协议：

- `.dm/tasks/`
- `.dm/design/`
- `.dm/session/`
- `.dm/specs/`
- `specs/`

## Phase 状态机

| Phase | 含义 | 执行者 | 推进方式 |
|---|---|---|---|
| `idle` | 尚未创建任务 | Human / Main Agent | 逻辑 `/dm:start`；Codex 用 `$dm start`；Claude Code 用 `/dm-start` |
| `clarifying` | 通过讨论和文件编辑形成需求简报 | Main Agent + Human | 逻辑 `/dm:continue`；Codex 用 `$dm continue`；Claude Code 用 `/dm-continue` |
| `designing` | 通过讨论和文件编辑形成设计草案 | Main Agent + Human | 逻辑 `/dm:continue`；Codex 用 `$dm continue`；Claude Code 用 `/dm-continue` |
| `design_review` | 等待人类确认当前 `design.md` | Human | 逻辑 `/dm:continue`；Codex 用 `$dm continue`；Claude Code 用 `/dm-continue` |
| `design_persisted` | 当前 `design.md` 已被确认 | Main Agent | 自动进入 `working` |
| `working` | Worker 执行任务 | Worker Sub Agent | worker report |
| `testing` | Test 只读验证 | Test Sub Agent | test report |
| `accepting` | Accept 只读交付前检查 | Accept Sub Agent | accept report |
| `human_acceptance` | 汇总变更并等待人类最终验收 | Human | 逻辑 `/dm:continue` 或反馈；Codex 用 `$dm continue` / `$dm feedback` |
| `done` | 任务完成 | Main Agent | 无 |
| `blocked` | 缺少决策、能力不支持或外部阻塞 | Main Agent / Human | 人类反馈 |

说明：

- Claude Code 第一阶段接受 `/dm-continue` 作为 `/dm:continue` 的等价命令。
- Codex 第一阶段使用 `$dm continue` 作为 `/dm:continue` 的等价入口；不要在交互式 Codex 中输入 `/dm:*`，因为当前 CLI 会在消息到达 Agent 前拒绝未知 slash command。
- Main Agent 必须在 `clarifying` 阶段完成至少三轮有意义的 CLI 内交互式澄清，确认记录写入 `brief.md` 后，才可以判断 `brief.md` 已足够清晰并提示用户输入对应平台的 continue 命令。
- “有意义的澄清轮次”必须影响需求事实，例如目标、范围、非目标、约束、验收标准、风险、优先级或关键边界；禁止为了凑轮次提出无实质影响的问题。
- Main Agent 必须在 `designing` 阶段完成至少三轮有意义的 CLI 内交互式设计确认，确认记录写入 `design.md` 后，才可以判断设计草案已完整并提示用户输入对应平台的 continue 命令。
- “有意义的设计确认轮次”必须影响设计事实，例如架构、方案、范围切片、取舍、验证计划、验收标准、发布/回滚或风险；禁止为了凑轮次提出无实质影响的问题。
- `brief.md` 和 `design.md` 是跨阶段交接的源文件。后续 phase 必须重新读取文件内容，不得只依赖对话上下文。

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
- 操作：Main Agent 通过多轮对话和文件写入维护 `.dm/tasks/[task-id]/brief.md`，双方对齐任务边界与预期。
- 强制交互：
  - Main Agent 必须在 Claude Code 或 Codex 的 CLI 对话中完成至少三个“待确认点”的有意义澄清轮次。
  - 每个待确认点必须提供 3 个或以上候选方案，并提供一个用户手动填入选项。
  - 对话格式必须类似：
    ```text
    对于待确认点A，有多个方案:
    1. aaa
    2. bbb
    3. ccc
    4. [用户手动填入]
    ```
  - Human 选择或手动输入后，Main Agent 必须把待确认点、需求影响、候选方案、用户选择、最终取值写入 `brief.md`。
  - 如果已回答澄清轮次少于三轮，必须继续提出下一轮同样格式的有意义确认问题。
  - 如果已达到三轮但仍有关键不确定点，继续提出下一轮同样格式的确认问题，直到关键歧义消失。
  - 如果 Main Agent 找不到新的有意义待确认点，不得提出无意义问题；应说明当前无法继续澄清的原因，并请 Human 提供补充或确认是否存在遗漏需求。
- 支持方式：
  - 直接澄清：围绕缺失字段和歧义提问。
  - 发散探索：提供多个目标拆分、范围取舍、风险优先、极简/完整方案、反例边界等思考方向。
  - 用户自定义输入：Human 可在对话中给出任意补充，也可直接编辑 `brief.md`。
- 完成条件：Main Agent 在收到 continue 命令后重新读取 `.dm/tasks/[task-id]/brief.md`，确认文件中至少有三条已回答的有意义交互式确认记录，且核心目标、范围、约束和关键歧义状态足以进入设计。

### Step 4 - 输出设计方案

- 执行者：Main Agent + Human
- 操作：Main Agent 必须先读取最新 `brief.md`，再通过多轮对话和文件写入维护 `.dm/design/[task-id]/design.md`。
- 强制交互：
  - Main Agent 必须在 Claude Code 或 Codex 的 CLI 对话中完成至少三个“设计待确认点”的有意义确认轮次。
  - 每个设计待确认点必须提供 3 个或以上候选方案，并提供一个用户手动填入选项。
  - 对话格式必须类似：
    ```text
    对于设计待确认点A，有多个方案:
    1. aaa
    2. bbb
    3. ccc
    4. [用户手动填入]
    ```
  - Human 选择或手动输入后，Main Agent 必须把设计待确认点、设计影响、候选方案、用户选择、最终取值写入 `design.md`。
  - 如果已回答设计确认轮次少于三轮，必须继续提出下一轮同样格式的有意义设计确认问题。
  - 如果已达到三轮但仍有关键设计不确定点，继续提出下一轮同样格式的确认问题，直到关键歧义消失。
  - 如果 Main Agent 找不到新的有意义设计待确认点，不得提出无意义问题；应说明当前无法继续设计确认的原因，并请 Human 提供补充或确认是否存在遗漏设计约束。
- 支持方式：
  - 方案发散：提供多方案对比、架构取舍、实现切片、风险优先、验收倒推、渐进交付等思考方向。
  - 用户自定义输入：Human 可在对话中指定方案、约束、取舍或直接编辑 `design.md`。
- 完成条件：`.dm/design/[task-id]/design.md` 存在，且 Main Agent 在收到 continue 命令后重新读取该文件，确认文件中至少有三条已回答的有意义设计确认记录，并判断当前文件内容已形成完整设计草案，进入 `design_review`。

### Step 5 - 用户确认方案

- 执行者：Human
- 操作：审阅并可直接编辑 `.dm/design/[task-id]/design.md`；明确同意或提出修改意见。
- 完成条件：Human 输入对应平台的 continue 命令。

### Step 6 - 确认当前设计文件

- 执行者：Main Agent
- 操作：重新读取 `.dm/design/[task-id]/design.md`，将当前文件内容视为被确认的设计，写入或更新 `.dm/design/[task-id]/decisions.md` 与 `.dm/design/[task-id]/revisions.md`。
- 完成条件：`design.md` 和 `decisions.md` 存在，进入 `working`。

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

### Step 9 - 汇总变更，移交人类验收

- 执行者：Main Agent
- 操作：生成 `.dm/session/[task-id]/summary.md`。
- 完成条件：summary 至少包含文件新增/修改/删除列表和设计决策变化。

人类主要阅读 `summary.md` 文本内容。`manifest.json` 第一阶段不强制生成。

### Step 10 - 人类最终验收

- 执行者：Human
- 通过：Human 输入 `/dm:continue`（Claude Code）或 `$dm continue`（Codex），任务进入 `done`。
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
| `.dm/session/` | 任务变更汇总，供人类验收 |
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
- Human 最终验收通过。

## 相关资源

- [Phase Controller Spec](../specs/phase-controller.spec.md)
- [Persistence Spec](../specs/persistence.spec.md)
- [Adapters Spec](../specs/adapters.spec.md)
- [Guardrail Engine Spec](../specs/guardrail-engine.spec.md)
