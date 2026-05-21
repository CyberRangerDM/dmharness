# 持久化模块 Spec

## 1. 范围

Persistence 模块负责把任务状态、设计决策、反馈、报告和会话摘要持久化到 `.dm` 目录。它不依赖独立 CLI，所有读写由 Claude Code / Codex Agent 按文件协议完成。

## 2. 原则

- 机器状态与人类可读内容分离。
- `state.json` 表示当前事实；`events.jsonl` 记录历史。
- `brief.md` 是 clarifying 的正式产物，通常在澄清完成后一次性写入。
- `design.md` 是 designing 的正式产物，通过自动 `design_review` 后成为实现依据。
- 阶段产物和报告应提供 `DM Compact Summary`，支持低上下文读取和跨会话恢复。
- Compact summary 是派生摘要，不替代正文事实；冲突时以正文为准并修复摘要。

## 3. File Layout

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
└── session/
    └── [task-id]/
        ├── summary.md
        └── manifest.json  # optional
```

## 4. `state.json`

Required fields:

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

Rules:

- Every task must have exactly one valid `state.json`.
- `schema_version` is `1` in phase 1.
- `state.json` may be overwritten to reflect current truth.
- Historical detail belongs in `events.jsonl`, phase artifacts, and reports.

## 5. Events And Summaries

`events.jsonl` is append-only. Each phase transition appends one JSON line:

```json
{"time":"2026-05-12T16:20:00+08:00","type":"phase_transition","from":"working","to":"testing","actor":"main","reason":"worker report created"}
```

`.dm/tasks/[task-id]/summary.md` supports agent recovery and should include:

- current phase
- task goal
- confirmed requirement boundary
- design summary
- latest worker/test/accept result
- next action

## 6. Compact Summary

These files should start with `## DM Compact Summary`:

- `brief.md`
- `design.md`
- `worker-result-[n].md`
- `test-report-[n].md`
- `accept-report-[n].md`
- `.dm/session/[task-id]/summary.md`

Compact summaries should expose gate fields such as confirmation count, gate status, result, blocking issues, key decisions, open ambiguity, and latest report paths.

Agent should read compact summaries, status markers, and targeted anchors first. Read full files when summaries are missing, stale, inconsistent, human-edited in an unverifiable way, or exact content is required for handoff.

## 7. Session Summary

`.dm/session/[task-id]/summary.md` is required before `done` and must contain:

- added files
- modified files
- deleted files
- design decision changes

`manifest.json` is optional in phase 1.

## 8. Recovery Flow

New session recovery must read, in order:

1. `AGENTS.md`
2. `.dm/workflow.md`
3. `.dm/tasks/[task-id]/state.json`
4. `.dm/tasks/[task-id]/summary.md`
5. `.dm/tasks/[task-id]/events.jsonl`
6. `.dm/tasks/[task-id]/brief.md` compact summary and confirmation records; read full text if needed
7. `.dm/design/[task-id]/design.md` compact summary and design records; read full text for validation or handoff
8. `.dm/design/[task-id]/decisions.md` and `revisions.md` status
9. latest `feedback-[n].md`, `worker-result-[n].md`, `test-report-[n].md`, `accept-report-[n].md`
10. `.dm/session/[task-id]/summary.md` and optional `manifest.json`

If task id is omitted, choose the latest `status != done` task by `updated_at`. If tied, ask the user to specify a task id.

Recovery must first decide completion:

- If `state.json.status = "done"` or `state.json.phase = "done"`, report completion only.
- If Worker/Test/Accept all passed and session summary exists but state is stale, append safe transition event(s), mark the task `done`, and report finalization.
- Otherwise resume from the first incomplete required phase.

## 9. Write Rules

- `events.jsonl` is append-only.
- Numbered files such as `feedback-[n].md`, `worker-result-[n].md`, `test-report-[n].md`, and `accept-report-[n].md` must never be overwritten.
- `brief.md` is written once after at least three meaningful grill-me-compliant clarification records exist and key ambiguity is resolved.
- `brief.md` records must include pending point, requirement impact, options, recommended answer, recommendation reason, upstream dependency, exploration evidence, user selection, final value, and status.
- `design.md` must record key design decisions, options/tradeoffs, implementation plan, expected file changes, validation plan, acceptance criteria, and risks.
- `decisions.md` is written or updated only after current `design.md` passes automatic `design_review`.
- Modified persisted design must be recorded in `revisions.md` and returned to `designing` or `design_review`.
- Test and Accept reports are read-only with respect to business code.
- `command-log.md` records meaningful commands when command logging is required by the current command protocol.

## 10. Acceptance Criteria

- State can be recovered from `.dm` files and `AGENTS.md` without conversation history.
- Required phase artifacts exist before phase advancement.
- Every phase transition appends an event.
- Session summary exists before `done`.
- Numbered artifacts are not overwritten.
