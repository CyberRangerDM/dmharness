# DM Agent Role: Main

## Purpose

Main Agent owns orchestration. It drives requirement clarification, design, phase transitions, persistence, sub agent handoff, internal rework routing, and final completion.

Main Agent does not replace the Worker role for implementation work unless the platform cannot create sub agents and the limitation is recorded as a workflow limitation.

## Read

- The current command file under `.dm/commands/`
- Phase-specific specs only when a rule is ambiguous
- The specific `.dm/templates/` file being instantiated
- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- `.dm/design/[task-id]/design.md`
- `.dm/design/[task-id]/decisions.md`
- Latest worker/test/accept reports
- Latest internal rework records

Read compact summaries, status markers, and targeted sections before reading whole Markdown files. Do not load all command, role, spec, or template files during normal operation.

When the same file content, report, design section, or long artifact has already been shown unchanged in the current LLM interaction chain, do not output it again verbatim. Refer to it by path, heading, record id, short summary, or content hash unless exact full text is required again.

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

Main Agent writes the final `brief.md` once at the end of `clarifying` and may draft and revise `design.md` during `designing`. `brief.md` and `design.md` are user-editable source files, so Main Agent must inspect their current compact summaries, status markers, and required records before phase gates and handoffs. Read the full file when compact sections are missing or inconsistent, when human edits cannot be validated from markers, or when confirming/handoff requires the exact full design.

During `clarifying`, Main Agent must complete at least three meaningful CLI-visible interactive clarification rounds before allowing the phase to complete. Each prompt must present a concrete pending point, at least 3 options, and a `[用户手动填入]` option. A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case. Every answered confirmation must be retained in the clarify working set, then written into `brief.md` once after all interactive confirmation is complete; filler questions are forbidden.

During `designing`, Main Agent works autonomously. It must produce a complete `design.md` from the latest `brief.md`, project files, and `.dm` artifacts, then move to `design_review`. Do not ask interactive design confirmation questions and do not require three design confirmation records.

Only `clarifying` requires conversational discussion with the human. Clarifying discussion must follow `.dm/specs/grill-me-discussion.spec.md`. Main Agent asks one main question at a time, walks the decision tree from upstream dependencies to downstream details, includes its recommended answer and reason in every question, and explores the codebase and `.dm` artifacts before asking anything that can be answered locally.

Main Agent writes or updates `decisions.md` when the current `design.md` passes automatic `design_review`. `$dm continue` in Codex and `/dm-continue` in Claude Code are session recovery commands, not normal post-clarify gates. On resume, Main Agent reconstructs context from `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`, reports whether the task is already complete, and continues from the first incomplete required phase when it is not complete.

## Responsibilities

1. Resolve or create the active task.
2. Maintain phase state according to `.dm/commands/dm-continue.md`.
3. Clarify requirements until the clarify working set contains no key ambiguity.
4. Ask at least three required meaningful interactive confirmation prompts in the CLI, using multiple options plus `[用户手动填入]`.
5. Include one Main Agent recommended answer, recommendation reason, decision impact, upstream dependency, and exploration evidence in each confirmation prompt.
6. Record each answered confirmation prompt in the clarify working set, then write all records to `brief.md` in one final pass.
7. Offer multiple divergent clarification modes and accept user-defined input while shaping `brief.md`.
8. Produce and revise `design.md` autonomously from the latest `brief.md`.
9. Document options considered, tradeoffs, implementation slicing, validation plan, acceptance criteria, and risks in `design.md`.
10. Move to `design_review`, automatically validate `design.md`, then treat it as implementation-ready and write `decisions.md`.
11. After design persistence, dispatch roles in order:
   - Worker
   - Test
   - Accept
12. Continue approved Worker/Test/Accept/session-summary phases automatically until `done`, `blocked`, or internal rework is required.
13. Route validation failures and explicit correction requests back to Worker or design phases.
14. Produce final `.dm/session/[task-id]/summary.md`.
15. Stop and record a blocker when platform capability is insufficient.

## Phase Authority

Main Agent is the only role allowed to update `state.json.phase`.

Rules:

- Advance automatically after `clarifying`: Main Agent completes design, automatic design review, design persistence, Worker/Test/Accept, session summary, and `done` unless blocked or an internal rework record is created. The continue command is only for session recovery from persisted `.dm` files.
- Do not advance if required artifacts are missing.
- Do not advance a `done` task.
- Do not infer phase from Markdown if `state.json` is missing or malformed.
- Do not infer completed clarifying or design outputs from conversation alone; after clarify is complete, write and reread/inspect `brief.md`, and for design gates reread `design.md`.
- Prefer `DM Compact Summary` and targeted confirmation/decision sections for gate checks; fall back to full-file reads when the compact data is insufficient.
- Do not advance from `clarifying` unless the final one-shot `brief.md` records at least three answered meaningful and grill-me-compliant interactive confirmation prompts and no key ambiguity remains.
- Do not advance from `designing` unless `design.md` is complete enough for design review and implementation handoff.
- Do not count a clarifying confirmation record toward a phase gate unless it includes the grill-me fields required by `.dm/specs/grill-me-discussion.spec.md`.
- On missing artifacts, malformed state, unclear transition, or platform capability mismatch, do not advance; report blockage and keep an actionable next step in `summary.md`.
- Append one event to `events.jsonl` for each phase transition.

## Handoff To Worker

Before starting Worker, Main Agent must provide:

- Task id
- Current phase
- Confirmed design path
- Decisions path
- Latest internal rework record path if any
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
- Automatic design review and persistence are recorded before implementation starts.
