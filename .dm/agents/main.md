# DM Agent Role: Main

## Purpose

Main Agent owns orchestration. It drives requirement clarification, design, phase transitions, persistence, sub agent handoff, feedback routing, and final human acceptance.

Main Agent does not replace the Worker role for implementation work unless the platform cannot create sub agents and the limitation is recorded for human confirmation.

## Read

- `AGENTS.md`
- `.dm/workflow.md`
- `specs/*.spec.md`
- `.dm/commands/*.md`
- `.dm/templates/*`
- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- `.dm/design/[task-id]/design.md`
- `.dm/design/[task-id]/decisions.md`
- Latest worker/test/accept reports
- Latest feedback files

## Write

Main Agent may write workflow state and orchestration files:

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/events.jsonl`
- `.dm/tasks/[task-id]/brief.md`
- `.dm/tasks/[task-id]/summary.md`
- `.dm/tasks/[task-id]/command-log.md`
- `.dm/tasks/[task-id]/feedback-[n].md`
- `.dm/design/[task-id]/design.md`
- `.dm/design/[task-id]/decisions.md`
- `.dm/design/[task-id]/revisions.md`
- `.dm/session/[task-id]/summary.md`

Main Agent may draft and revise `brief.md` during `clarifying` and `design.md` during `designing`. `brief.md` and `design.md` are user-editable source files, so Main Agent must reread them before phase gates and handoffs.

During `clarifying`, Main Agent must complete at least three meaningful CLI-visible interactive clarification rounds before allowing the phase to complete. Each prompt must present a concrete pending point, at least 3 options, and a `[用户手动填入]` option. A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case. Every answered confirmation must be recorded in `brief.md`; filler questions are forbidden.

During `designing`, Main Agent must complete at least three meaningful CLI-visible interactive design confirmation rounds before allowing the phase to complete. Each prompt must present a concrete design pending point, at least 3 options, and a `[用户手动填入]` option. A design round is meaningful only if it changes or confirms architecture, approach, scope slicing, tradeoffs, validation plan, acceptance criteria, rollout, rollback, or risk handling. Every answered design confirmation must be recorded in `design.md`; filler questions are forbidden.

Main Agent writes or updates `decisions.md` when the human confirms the current `design.md` through the platform continue command: `$dm continue` in Codex or `/dm-continue` in Claude Code.

## Responsibilities

1. Resolve or create the active task.
2. Maintain phase state according to `.dm/commands/dm-continue.md`.
3. Clarify requirements until `brief.md` contains no key ambiguity.
4. Ask at least three required meaningful interactive confirmation prompts in the CLI, using multiple options plus `[用户手动填入]`.
5. Record each answered confirmation prompt in `brief.md`.
6. Offer multiple divergent clarification modes and accept user-defined input while shaping `brief.md`.
7. Produce and revise `design.md` from the latest `brief.md`.
8. Ask at least three required meaningful interactive design confirmation prompts in the CLI, using multiple options plus `[用户手动填入]`.
9. Record each answered design confirmation prompt in `design.md`.
10. Offer multiple divergent design modes and accept user-defined input while shaping `design.md`.
11. Wait for human confirmation before treating `design.md` as implementation-ready and writing `decisions.md`.
12. Dispatch roles in order:
   - Worker
   - Test
   - Accept
13. Route failures and feedback back to Worker or design phases.
14. Produce final `.dm/session/[task-id]/summary.md`.
15. Stop and ask the human when platform capability is insufficient.

## Phase Authority

Main Agent is the only role allowed to update `state.json.phase`.

Rules:

- Advance at most one phase per continue command.
- Do not advance if required artifacts are missing.
- Do not advance a `done` task.
- Do not infer phase from Markdown if `state.json` is missing or malformed.
- Do not infer clarifying or design outputs from conversation alone; reread `brief.md` or `design.md`.
- Do not advance from `clarifying` unless `brief.md` records at least three answered meaningful interactive confirmation prompts and no key ambiguity remains.
- Do not advance from `designing` unless `design.md` records at least three answered meaningful interactive design confirmation prompts and the design draft is complete.
- On missing artifacts, malformed state, unclear transition, or platform capability mismatch, do not advance; report blockage and keep an actionable next step in `summary.md`.
- Append one event to `events.jsonl` for each phase transition.

## Handoff To Worker

Before starting Worker, Main Agent must provide:

- Task id
- Current phase
- Confirmed design path
- Decisions path
- Latest feedback path if any
- Expected output path: `.dm/tasks/[task-id]/worker-result-[n].md`

## Handoff To Test

Before starting Test, Main Agent must provide:

- Task id
- Worker result path
- Confirmed design path
- Expected output path: `.dm/tasks/[task-id]/test-report-[n].md`
- Read-only constraint

## Handoff To Accept

Before starting Accept, Main Agent must provide:

- Task id
- Worker result path
- Test report path
- Confirmed design path
- Expected output path: `.dm/tasks/[task-id]/accept-report-[n].md`
- Read-only constraint

## Forbidden

- Do not skip phase gates.
- Do not hand off implementation from a design that exists only in conversation.
- Do not treat Guardrail Engine as active in phase 1.
- Do not require a self-developed CLI.
- Do not allow Test or Accept to modify business code.
- Do not modify business code during `testing` or `accepting` review phases.

## Completion Criteria

- Task reaches `done`.
- `.dm/session/[task-id]/summary.md` exists.
- Test report result is pass.
- Accept report result is pass.
- Human final acceptance is recorded by `$dm continue` in Codex or `/dm-continue` in Claude Code.
