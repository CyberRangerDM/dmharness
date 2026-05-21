# DM Command: continue

Logical command: `/dm:continue [task-id]`

Platform entrypoints:

- Codex: `$dm continue [task-id]`
- Claude Code: `/dm-continue [task-id]`

Claude Code equivalent project command: `/dm-continue [task-id]`

Purpose: advance the current Harness-DM task through the next autonomous workflow segment.

This is a platform-neutral command template. Claude Code and Codex adapters may copy or reference this file. Do not invoke a self-developed CLI.

## Inputs

- Optional task id.
- Current conversation context.
- Current task state.

If task id is omitted, load the most recently updated active task from `.dm/tasks/*/state.json`.

If multiple active tasks have the same latest `updated_at`, stop and ask the user to specify a task id.

## Read

- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- Phase-specific required artifacts

Do not read the complete workflow, all specs, all command files, all role files, or all templates during a normal continue. This command file plus `state.json`, `summary.md`, and the current phase artifact are the default context set. Read broader reference files only when a required rule is ambiguous, a compact summary is missing or stale, a human-edited artifact cannot be validated from markers, or an exception path requires the full rule text.

For `clarifying`, `brief.md` is normally written once after the interactive clarification rounds are complete. If `.dm/tasks/[task-id]/brief.md` already exists because clarify was finalized or the human edited it early, inspect it before evaluating the transition. Prefer the `DM Compact Summary` and `Interactive Confirmation Records` sections; read the full file if either section is missing, inconsistent, or recently edited in a way that cannot be validated by targeted checks. If `brief.md` does not exist yet, do not treat that as an error during ongoing clarification; continue the next meaningful prompt unless the current conversation contains enough answered rounds to finalize and write it once.

For `designing` and later design-related transitions, always inspect `.dm/design/[task-id]/design.md` immediately before evaluating the transition. Prefer the `DM Compact Summary` and decision/status markers for normal status checks; read the full file when confirming the exact design content or handing off to Worker/Test/Accept.

## Phase-Specific Reading

Use this table as the normal minimum read set:

| Current Phase | Required Reads | Usually Avoid |
|---|---|---|
| `clarifying` | `state.json`, `summary.md`, existing `brief.md` compact summary and confirmation records if present; otherwise current conversation clarify working set | Full specs/templates and repeated `brief.md` reads/writes unless gate fields are unclear |
| `designing` | `state.json`, `summary.md`, `brief.md` compact summary, `design.md` compact summary if present | Full `brief.md` after compact summary is sufficient |
| `design_review` | `state.json`, `summary.md`, current `design.md` full content before confirmation | Unrelated command, agent, and template files |
| `design_persisted` | `state.json`, `summary.md`, `design.md`, `decisions.md` existence and compact summaries | Full workflow/spec rereads |
| `working` | `state.json`, `summary.md`, confirmed `design.md`, `decisions.md`, latest feedback if present | Test/accept role files unless dispatching those roles |
| `testing` | `state.json`, `summary.md`, latest worker report compact summary, confirmed design compact summary | Full worker report unless result/details are unclear |
| `accepting` | `state.json`, `summary.md`, latest worker/test report compact summaries, confirmed design compact summary | Full implementation files unless needed for acceptance |
| `human_acceptance` | `state.json`, `summary.md`, session summary compact summary or file lists | Full historical reports unless feedback references them |

Use targeted commands such as `jq` for `state.json` and `rg` for headings, status markers, result lines, and confirmation IDs. Avoid `cat` or broad `sed` ranges for large Markdown or implementation files unless the full text is necessary for the current gate.

## Write

Depending on phase:

- Update `.dm/tasks/[task-id]/state.json`
- Append `.dm/tasks/[task-id]/events.jsonl`
- Update `.dm/tasks/[task-id]/summary.md`
- Create or update `.dm/design/[task-id]/design.md` while drafting in `designing`
- Create or update `.dm/design/[task-id]/decisions.md` only when confirming the current design file
- Create `.dm/tasks/[task-id]/feedback-[n].md` on failed test/accept/human feedback
- Create `.dm/session/[task-id]/summary.md` when entering `human_acceptance`

## Phase Transition Rules

`$dm continue` / `/dm-continue` advances by workflow segment, not by a single phase:

- From `clarifying`, if the brief gate passes, Main Agent enters `designing`, completes the design autonomously, writes `design.md`, and stops at `design_review` for human approval.
- From `design_review`, the command means Human approves the current `design.md`; Main Agent then runs `design_persisted`, `working`, `testing`, `accepting`, `human_acceptance`, and `done` automatically until a failure, blocker, or required feedback stops the run.
- From operational phases after approval, Main Agent continues autonomously until `done`, failure feedback, or `blocked`.

| Current Phase | Required Condition | Next Phase |
|---|---|---|
| `clarifying` | final `brief.md` has been written once, contains at least three answered meaningful and grill-me-compliant interactive confirmation records, and Main Agent judges its file content has no key ambiguity remaining | `designing`, then automatically `design_review` after design is written |
| `designing` | latest `brief.md` is sufficient and Main Agent can write a complete `design.md` without more human input | `design_review` |
| `design_review` | Human invoked the platform continue command to confirm the current `design.md` content | `design_persisted` |
| `design_persisted` | `design.md` and `decisions.md` exist | `working` |
| `working` | Latest `worker-result-[n].md` exists | `testing` |
| `testing` | Latest `test-report-[n].md` exists and result is pass | `accepting` |
| `accepting` | Latest `accept-report-[n].md` exists and result is pass | `human_acceptance` |
| `human_acceptance` | `.dm/session/[task-id]/summary.md` exists after Worker/Test/Accept pass | `done` |

Failure transitions:

- `testing` fail -> write `feedback-[n].md` -> `working`
- `accepting` fail -> write `feedback-[n].md` -> `working`
- `human_acceptance` feedback -> write `feedback-[n].md` -> `working`
- `design_review` requested changes -> return to `designing` or stay in `design_review` depending on feedback

## Clarifying Artifact Rule

`brief.md` is the formal output of the clarifying phase.

Main Agent must complete CLI-visible multi-turn discussion with the human before writing the final `brief.md`. It must complete at least three meaningful interactive clarification rounds before clarifying can complete.

During ongoing `clarifying`, do not revise `brief.md` after each human answer. Keep answered rounds in the current clarify working set. When at least three meaningful rounds have been answered and no key ambiguity remains, read `.dm/templates/task-brief.md` if needed and write `.dm/tasks/[task-id]/brief.md` once with the final requirement brief and all confirmation records. If a human directly edits an existing `brief.md` before finalization, treat it as an external source to merge, not as a reason to rewrite the file every round.

Each interactive confirmation prompt must have this structure:

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

Rules:

- Replace `待确认点A` with a concrete unresolved requirement point.
- Provide at least 3 meaningful options before `[用户手动填入]`.
- Ask exactly one main pending point per round.
- Provide the Main Agent recommended answer and recommendation reason.
- State the decision impact and any upstream dependency.
- Explore available project files and `.dm` artifacts before asking; do not ask the human for facts that can be discovered locally.
- Accept either a numbered choice or free-form human input.
- Record the point, requirement impact, options, recommended answer, recommendation reason, upstream dependency, exploration evidence, selected answer, final value, and status in the clarify working set until the final one-shot `brief.md` write.
- A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case.
- If fewer than three answered meaningful rounds are recorded, ask the next confirmation prompt in the same format.
- After three answered meaningful rounds, keep asking only while key ambiguity remains.
- Do not ask filler questions solely to increase the count. If no meaningful next question can be identified, do not advance silently; ask the human for missing context or confirmation of omitted requirements.

Main Agent should use divergent thinking modes such as goal decomposition, scope tradeoffs, risk-first questioning, minimal vs complete versions, boundary examples, and user-provided free-form input.

The human may directly edit `brief.md` before invoking continue. On continue, Main Agent must read the latest file and use that file content as the source of truth for the next phase.

If the final `brief.md` has fewer than three answered meaningful and grill-me-compliant interactive confirmation records, `$dm continue` / `/dm-continue` must not advance from `clarifying`; Main Agent must ask the next required meaningful confirmation prompt instead. If `brief.md` is absent but the current clarify working set is complete, Main Agent must write `brief.md` once before advancing.

## Design Artifact Rule

`design.md` is the formal output of the designing phase.

Main Agent creates and revises `design.md` during `designing` autonomously from the latest `brief.md`, relevant project files, and `.dm` artifacts. It should still do real design work: compare alternatives, make architecture tradeoffs, slice implementation, plan validation backward from acceptance criteria, and document risks.

Rules:

- Do not ask interactive design confirmation questions.
- Do not require three design confirmation records.
- Do not ask the human for facts that can be discovered locally.
- If a design-critical fact is genuinely unavailable, write the blocker to `summary.md` and move to `blocked` instead of starting an interactive design round.
- `design.md` must include enough concrete implementation guidance for Worker/Test/Accept: goal, requirements mapping, scope, non-goals, proposed approach, options considered, implementation plan, expected file changes, validation plan, acceptance criteria, risks, and source of truth.
- After writing a complete draft, set phase to `design_review` and ask the human to approve by invoking the platform continue command.

The human may directly edit `design.md` during `design_review`. On continue, Main Agent must read the latest file and use that file content as the source of truth for design confirmation and implementation handoff.

## Design Confirmation Rule

When moving from `design_review` to `design_persisted`:

1. Read the current `.dm/design/[task-id]/design.md`.
2. Treat that exact file content as the human-confirmed design.
3. Write design decisions to `decisions.md`.
4. Create or update `revisions.md`.
5. Append phase transition event.

`design.md` may exist before human confirmation as a draft. Do not proceed to implementation from conversation-only design content.

If an already confirmed design changes, move back to `design_review` and wait for another platform continue command.

## Session Summary Rule

When moving to `human_acceptance`, create `.dm/session/[task-id]/summary.md`.

The summary must include at least:

- added files
- modified files
- deleted files
- design decision changes

`manifest.json` is optional and not required in phase 1.

## Failure Rules

- If required artifacts are missing, do not advance.
- If current phase is `clarifying` and the final `brief.md` is absent while the current clarify working set is incomplete, do not advance; ask a required meaningful confirmation prompt.
- If current phase is `clarifying` and the final `brief.md` has fewer than three answered meaningful and grill-me-compliant interactive confirmation records, do not advance; ask a required meaningful confirmation prompt.
- If the required clarifying records lack grill-me fields such as recommended answer, recommendation reason, upstream dependency, or exploration evidence, do not advance; ask or repair the next meaningful confirmation record according to `specs/grill-me-discussion.spec.md`.
- If current phase is `designing` and Main Agent cannot produce a complete design from existing files, move to `blocked` with a concrete missing item rather than asking interactive design questions.
- If a report is present but its result cannot be determined, do not advance; record a blocker or corrected-report requirement instead of starting an interactive discussion.
- If current phase is `blocked`, do not advance until the blocking reason has human feedback.
- If current phase is `done`, do not advance.
- If `state.json` is missing or malformed, stop and do not infer transition from Markdown alone.
- Do not skip required artifacts, even when multiple phases are completed in one invocation.

## Event Format

Append one JSON line to `events.jsonl`:

```json
{"time":"{{TIMESTAMP}}","type":"phase_transition","actor":"main","from":"{{FROM_PHASE}}","to":"{{TO_PHASE}}","reason":"{{REASON}}"}
```

## User Response

Report:

- task id
- previous phase
- new phase or final status, or missing requirements if not advanced
- files written
- next action

## Acceptance Criteria

- The platform continue command advances by workflow segment: `clarifying` can produce design and stop at `design_review`; approved `design_review` can run through remaining phases automatically.
- Normal continue handling uses phase-specific minimum reads and avoids reloading unrelated workflow/spec/template files.
- Missing required artifacts prevent advancement.
- Every phase transition appends one event, including transitions completed in the same invocation.
- `brief.md` is the source of truth when leaving `clarifying`.
- `clarifying` cannot complete unless the final one-shot `brief.md` records at least three answered meaningful interactive confirmation prompts with multiple options and a manual input option.
- `clarifying` cannot complete unless those records also include recommended answers, recommendation reasons, dependency notes, and exploration evidence.
- `design.md` is the source of truth when leaving `designing`, confirming design, and handing off to Worker/Test/Accept.
- `designing` completes autonomously when `design.md` is complete enough for review and implementation handoff; it must not require interactive design confirmation records.
- `decisions.md` is created or updated only when the current `design.md` is human-confirmed.
- Failed test/accept creates feedback and returns to `working`.
- Entering `human_acceptance` creates `.dm/session/[task-id]/summary.md`.
- If Worker, Test, Accept, and session summary all pass after design approval, Main Agent marks the task `done` without requiring another human continue.
