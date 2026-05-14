# 围栏模块（Guardrail Engine）Spec

## 1. 范围

Guardrail Engine 负责识别、记录和阻止危险操作。根据当前决策，本模块第一阶段不实现实际拦截，只保留规范、文件结构和未来接口，以便后续接入 Claude Code / Codex 原生 hook 或权限机制。

## 2. 当前版本策略

版本：`v0-deferred`

- 不启用命令拦截。
- 不改变 Claude Code / Codex 原生权限与审批行为。
- 不实现独立 CLI。
- 仅定义策略文件、事件格式、适配接口和验收标准。
- 这是已确认的阶段性取舍；第一阶段可以不满足原始需求中“危险操作由代码驱动阻挡”的目标。

## 3. 输入定义

### 未来 hook 输入

```ts
type ToolIntent = {
  platform: "claude-code" | "codex";
  task_id?: string;
  actor: "main" | "worker" | "test" | "accept";
  tool: "bash" | "edit" | "write" | "read" | "web" | "unknown";
  command?: string;
  paths?: string[];
  cwd: string;
  raw: unknown;
};
```

### 策略输入

- `.dm/specs/guardrails.md`
- `.dm/guardrails/policy.json`
- `.dm/tasks/[task-id]/state.json`

### 人类审批输入

当前阶段不启用。未来格式：

```text
/dm:approve [task-id] [approval-id]
/dm:deny [task-id] [approval-id]
```

## 4. 输出定义

### 判定输出

```ts
type GuardrailDecision = {
  decision: "allow" | "ask_human" | "deny" | "noop";
  severity: "none" | "low" | "medium" | "high";
  reason: string;
  matched_rule?: string;
  approval_id?: string;
};
```

当前 `v0-deferred` 固定输出：

```json
{
  "decision": "noop",
  "severity": "none",
  "reason": "guardrail engine deferred"
}
```

### 事件输出

未来写入：

- `.dm/session/[task-id]/guardrail-events.jsonl`
- `.dm/session/[task-id]/approvals.md`

当前版本不强制生成。

## 5. 文件结构

```text
.dm/
├── specs/
│   └── guardrails.md
├── guardrails/
│   ├── policy.json
│   └── README.md
└── session/
    └── [task-id]/
        ├── guardrail-events.jsonl
        └── approvals.md
```

`policy.json` 建议格式：

```json
{
  "version": 1,
  "mode": "disabled",
  "rules": [
    {
      "id": "dangerous-git-reset",
      "match": {
        "tool": "bash",
        "command_prefix": ["git reset", "git checkout --"]
      },
      "decision": "ask_human",
      "severity": "high"
    }
  ]
}
```

## 6. 接口设计

本模块不提供独立 CLI。接口以 hook handler 或 Agent 内部函数形式实现。

```ts
function loadPolicy(root: string): GuardrailPolicy;
function evaluateIntent(intent: ToolIntent, policy: GuardrailPolicy): GuardrailDecision;
function recordGuardrailEvent(taskId: string, intent: ToolIntent, decision: GuardrailDecision): void;
function requestApproval(taskId: string, intent: ToolIntent, decision: GuardrailDecision): ApprovalRequest;
function resolveApproval(taskId: string, approvalId: string, approved: boolean): ApprovalResult;
```

### Claude Code 未来接入点

- `PreToolUse`：执行前判断是否 allow/ask/deny。
- `PostToolUse`：记录执行结果。
- `PermissionRequest`：承接 Claude Code 原生权限请求。

### Codex 未来接入点

- 优先使用 Codex 原生 permissions / sandbox / hooks 能力。
- 若当前版本 hook 能力不足，则只使用 Codex 原生审批模式，不强行实现额外拦截。

## 7. 默认危险操作分类（未来启用）

| 类别 | 例子 | 默认决策 |
|---|---|---|
| 删除文件 | `rm`, `find -delete` | `ask_human` |
| 丢弃修改 | `git reset`, `git checkout --` | `ask_human` |
| 历史重写 | `rebase`, `push --force` | `ask_human` |
| 远程脚本 | `curl ... | sh` | `ask_human` |
| 仓库外写入 | 写入非工作区路径 | `ask_human` |
| 权限修改 | `chmod`, `chown` | `ask_human` |
| 网络访问 | 下载依赖、访问外部 API | `ask_human` |

当前版本不启用上述规则。

## 8. 验收标准

### v0-deferred 验收

- `specs/guardrail-engine.spec.md` 存在，并明确标注当前不实现实际围栏。
- 不新增会拦截命令的 hook、脚本或独立 CLI。
- 不改变 Claude Code / Codex 原生权限策略。
- 其他模块不得依赖 Guardrail Engine 的实际阻断能力。

### 未来启用验收

- 对同一 `ToolIntent`，`evaluateIntent` 必须产生确定性结果。
- 命中 `ask_human` 或 `deny` 的规则时，工具调用不得继续执行，除非有有效审批。
- 每次拦截或审批必须写入 `guardrail-events.jsonl`。
- 审批记录必须包含 task id、命令摘要、审批结果、审批时间。
- `test` 与 `accept` agent 触发写操作时，默认判定为 `ask_human` 或 `deny`。

## 9. 已确认限制与后续决策

- 第一阶段不实现实际围栏，只保留规范和未来接口。
- 后续如果恢复“危险操作必须由代码驱动阻挡”，再决策审批粒度采用单条命令、命令前缀、任务级还是会话级。
- 后续如果 Codex hook 能力不足，再决策是否只依赖 Codex 原生审批模式。
