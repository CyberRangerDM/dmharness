# 流程控制模块 Spec

## 1. 范围

Phase Controller 负责把任务从需求输入推进到最终验收完成。它不作为独立 CLI 存在，而是在 Claude Code / OpenAI Codex 原生 CLI 内通过项目命令、skill invocation、项目指令和 `.dm` 文件协议执行。

本模块只控制阶段、状态、角色顺序和阶段门禁；不负责具体业务代码实现，也不负责危险命令拦截。

## 2. 约束

- task id 格式为 `YYYYMMDDHHmm-xxxxxxxx`。
- 多 agent 顺序执行。
- `review` 与 `accept` 合并为 `accept` 角色。
- `test` 与 `accept` 只读业务代码；发现问题只输出报告和反馈。
- 只有 `clarifying` 默认需要 Human 互动；后续设计、设计校验、实现、测试、验收和收尾默认自动推进。
- 修改已持久化设计时，必须回到 `designing` 或 `design_review` 重新生成、校验并持久化。
- Codex 使用 `$dm` skill invocation，不依赖项目级 `/dm:*` slash command。
- `brief.md` 和 `design.md` 是 phase 交接的文件级事实来源；后续 phase 不得只依赖对话上下文。

## 3. Phase 定义

| Phase | 含义 | 执行者 | 下一步 |
|---|---|---|---|
| `idle` | 尚未创建任务 | Human / Main | `$dm start` 或 `/dm-start` |
| `clarifying` | 按 `.dm/skills/grill-me.md` 提问、写入 `brief.md` 并确认是否调整 | Main + Human | brief 调整确认后自动进入 `designing` |
| `designing` | 根据 `brief.md` 生成 `design.md` | Main | 自动进入 `design_review` |
| `design_review` | 自动校验当前 `design.md` | Main | 自动进入 `design_persisted` |
| `design_persisted` | 当前设计已校验并持久化 | Main | 自动进入 `working` |
| `working` | Worker 执行实现 | Worker | worker report |
| `testing` | Test 只读验证 | Test | test report |
| `accepting` | Accept 只读交付检查 | Accept | accept report |
| `human_acceptance` | 生成 session summary 并自动完成 | Main / Human | `done` |
| `done` | 任务完成 | Main | 无 |
| `blocked` | 缺失信息、平台能力不足或外部阻塞 | Main / Human | 人工解除阻塞或新证据 |

主路径：

```text
idle -> clarifying -> designing -> design_review -> design_persisted
  -> working -> testing -> accepting -> human_acceptance -> done
```

Failure loops:

- `testing` fail -> `feedback-[n].md` -> `working`
- `accepting` fail -> `feedback-[n].md` -> `working`
- any phase -> `blocked` when required evidence or capability is missing

## 4. Command Mapping

| 逻辑命令 | Codex 入口 | Claude Code 入口 | 含义 |
|---|---|---|---|
| `/dm:start` | `$dm start [title]` | `/dm-start [title]` | 创建任务并进入 `clarifying` |
| `/dm:continue` | `$dm continue [task-id]` | `/dm-continue [task-id]` | 会话恢复时从 `.dm` 文件判断完成状态并继续未完成任务 |

## 5. Phase Artifacts

| Phase | Required artifact |
|---|---|
| `clarifying` | `.dm/tasks/[task-id]/brief.md` |
| `designing` | `.dm/design/[task-id]/design.md` |
| `design_review` | current `.dm/design/[task-id]/design.md` |
| `design_persisted` | `.dm/design/[task-id]/design.md`, `.dm/design/[task-id]/decisions.md` |
| `working` | `.dm/tasks/[task-id]/worker-result-[n].md` |
| `testing` | `.dm/tasks/[task-id]/test-report-[n].md` |
| `accepting` | `.dm/tasks/[task-id]/accept-report-[n].md` |
| `human_acceptance` | `.dm/session/[task-id]/summary.md` |
| `done` | `state.json.status = "done"` |

Every phase transition must update `.dm/tasks/[task-id]/state.json` and append one JSON line to `.dm/tasks/[task-id]/events.jsonl`.

## 6. Phase Gates

| Current phase | Gate |
|---|---|
| `clarifying` | `brief.md` exists, human has said no adjustment is needed or adjustment is complete, and Main has reread the latest `brief.md` |
| `designing` | latest `brief.md` is sufficient and complete `design.md` exists |
| `design_review` | Main rereads `design.md` and validates brief coverage, constraints, acceptance criteria, implementation plan, validation plan, and risks |
| `design_persisted` | `design.md` and `decisions.md` exist |
| `working` | latest worker report exists |
| `testing` | latest test report exists and result is pass |
| `accepting` | latest accept report exists and result is pass |
| `human_acceptance` | session summary exists and Worker/Test/Accept have passed |

Missing required artifacts prevent advancement. `state.json` missing or malformed prevents advancement.

## 7. Clarifying

- Main Agent must ask questions according to `.dm/skills/grill-me.md`.
- Ask one main question at a time.
- Include Main Agent's recommended answer in each question.
- Resolve upstream dependencies before downstream details.
- Explore codebase, project files, and `.dm` artifacts before asking the human for facts.
- When the clarified content is sufficient, write the summarized requirement to `brief.md`.
- After writing or updating `brief.md`, ask the human whether it still needs adjustment.
- If the human says no adjustment is needed or adjustment is complete, Main rereads latest `brief.md` and leaves `clarifying`.
- If the human says adjustment is needed, Main remains in `clarifying` until the feedback is applied or direct human edits are complete.

## 8. Designing And Review

- `designing` is autonomous. Main Agent writes `design.md` from latest `brief.md`, project files, and `.dm` artifacts.
- Do not ask interactive design confirmation questions.
- `design.md` must record options considered, tradeoffs, implementation plan, expected file changes, validation plan, acceptance criteria, and risks.
- `design_review` is an automatic validation/persistence step. Human approval through continue is not required.
- `decisions.md` is written or updated after the current `design.md` passes automatic review.

## 9. `/dm:continue` Recovery Behavior

`$dm continue` / `/dm-continue` is only for session recovery.

Behavior:

1. Resolve task id.
2. Read `.dm/tasks/[task-id]/state.json`; stop if missing or malformed.
3. Read task summary, events, current task artifacts, design artifacts, and session artifacts needed to reconstruct context.
4. Reconstruct phase/status, latest iteration, latest feedback, latest role reports, design persistence state, session summary state, blockers, and missing artifacts.
5. If `state.json.status = "done"` or `state.json.phase = "done"`, report completion and do not advance.
6. If Worker/Test/Accept pass reports and session summary exist but state is stale, append safe transition event(s), mark task `done`, and report recovery finalization.
7. If incomplete, continue from the first incomplete required phase without rerunning phases already proven complete by files.

Persisted files are the source of truth during recovery. Conversation context may supplement but must not override newer files.

## 10. Acceptance Criteria

- Normal post-clarify flow runs automatically through design, review, implementation, testing, accept, session summary, and done unless blocked or internal rework is required.
- `$dm continue` / `/dm-continue` reconstructs context from `.dm/tasks`, `.dm/design`, and `.dm/session` before resuming.
- Required artifacts are never skipped.
- Failed test or accept creates feedback and returns to `working`.
- Test and Accept do not modify business code.
- A task is not marked `done` until session summary exists and Worker/Test/Accept pass.
