# 软件任务语言选择原则 Spec

## 1. 范围

本规范约束使用 harness-dm 执行软件类任务时的编程语言选择。它适用于需求澄清、设计、实现和返工阶段中涉及新增代码、重写模块、引入工具链或创建新项目的决策。

本规范不强制改变已有项目的主语言，也不覆盖特定平台、运行时或生态的硬性要求。

## 2. 原则

- 软件类任务默认优先使用 Golang、TypeScript 或 Python。
- 当已有代码库、框架、构建系统或团队约定已经明确主语言时，优先保持与现有项目一致。
- 当目标平台存在事实上的语言要求或强约束时，可以使用平台所需语言，例如 Android 的 Java/Kotlin、iOS 的 Objective-C/Swift、系统扩展或嵌入式平台要求的语言。
- 如果任务需要选择 Golang、TypeScript、Python 以及平台必要语言之外的其他编程语言，Agent 必须先向 Human 说明原因、影响和替代方案，并取得 Human 同意后再采用。
- 不得仅因为 Agent 熟悉、示例较多或临时方便而引入其他语言。

## 3. Phase 要求

- `clarifying`：如果语言选择会影响项目形态、部署、运行环境、维护成本或 Human 预期，必须作为有意义约束澄清或记录到最终 `brief.md`。
- `designing`：`design.md` 必须记录所选语言、选择依据、是否遵循本规范，以及是否需要 Human 同意。
- `working`：Worker 不得偏离已确认设计中的语言选择；如发现必须改用其他语言，应报告 blocker 或反馈给 Main Agent 重新设计。
- `testing` / `accepting`：检查实现语言是否符合 `design.md` 和本规范；不符合时应报告失败或风险。

## 4. Acceptance Criteria

- 新增或重写的软件实现默认落在 Golang、TypeScript 或 Python，除非已有项目或平台要求另有约束。
- 使用其他语言前已取得 Human 明确同意，或设计中记录了平台强制原因。
- `design.md` 中能追溯语言选择和取舍依据。
