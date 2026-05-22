# Guardrail Engine Spec

## 1. 范围

Guardrail Engine 用于描述危险操作拦截的未来方向。当前版本为 `v0-deferred`，不实现实际拦截。

## 2. Current Policy

- 不启用命令拦截。
- 不安装 Claude Code / Codex hook。
- 不改变 Claude Code / Codex 原生权限与审批行为。
- 不实现独立 CLI。
- 其他模块不得依赖 Guardrail Engine 的实际阻断能力。

## 3. Existing Files

```text
.dm/
├── specs/
│   ├── guardrail-engine.spec.md
│   └── guardrails.md
└── guardrails/
    ├── policy.json
    └── README.md
```

`policy.json` 在 phase 1 中应保持 disabled/noop 语义。

## 4. Acceptance Criteria

- `.dm/specs/guardrail-engine.spec.md` 明确标注当前不实现实际围栏。
- 不新增会拦截命令的 hook、脚本或独立 CLI。
- 不改变平台原生权限策略。
- Worker/Test/Accept 的读写约束由工作流指令和 phase 规则执行，不由 Guardrail Engine 执行。
