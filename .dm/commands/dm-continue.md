# DM Command: continue

Logical command: `/dm:continue [task-id]`

Platform entrypoints:

- Codex: `$dm continue [task-id]`
- Claude Code: `/dm-continue [task-id]`

Claude Code equivalent project command: `/dm-continue [task-id]`

Purpose: advance the current Harness-DM task by at most one phase.

This is a platform-neutral command template. Claude Code and Codex adapters may copy or reference this file. Do not invoke a self-developed CLI.

## Inputs

- Optional task id.
- Current conversation context.
- Current task state.

If task id is omitted, load the most recently updated active task from `.dm/tasks/*/state.json`.

If multiple active tasks have the same latest `updated_at`, stop and ask the user to specify a task id.

## Read

- `AGENTS.md`
- `.dm/workflow.md`
- `specs/phase-controller.spec.md`
- `specs/persistence.spec.md`
- `.dm/tasks/[task-id]/state.json`
- `.dm/tasks/[task-id]/summary.md`
- Phase-specific required artifacts

For `clarifying`, always read `.dm/tasks/[task-id]/brief.md` immediately before evaluating the transition.

For `designing` and later design-related transitions, always read `.dm/design/[task-id]/design.md` immediately before evaluating the transition.

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

Advance at most one phase per invocation.

| Current Phase | Required Condition | Next Phase |
|---|---|---|
| `clarifying` | latest `brief.md` exists, contains at least three answered meaningful interactive confirmation records, and Main Agent judges its file content has no key ambiguity remaining | `designing` |
| `designing` | latest `design.md` exists, contains at least three answered meaningful interactive design confirmation records, and Main Agent judges its file content is a complete design draft | `design_review` |
| `design_review` | Human invoked the platform continue command to confirm the current `design.md` content | `design_persisted` |
| `design_persisted` | `design.md` and `decisions.md` exist | `working` |
| `working` | Latest `worker-result-[n].md` exists | `testing` |
| `testing` | Latest `test-report-[n].md` exists and result is pass | `accepting` |
| `accepting` | Latest `accept-report-[n].md` exists and result is pass | `human_acceptance` |
| `human_acceptance` | Human invoked the platform continue command for final acceptance | `done` |

Failure transitions:

- `testing` fail -> write `feedback-[n].md` -> `working`
- `accepting` fail -> write `feedback-[n].md` -> `working`
- `human_acceptance` feedback -> write `feedback-[n].md` -> `working`
- `design_review` requested changes -> return to `designing` or stay in `design_review` depending on feedback

## Clarifying Artifact Rule

`brief.md` is the formal output of the clarifying phase.

Main Agent must create and revise `brief.md` through CLI-visible multi-turn discussion with the human. It must complete at least three meaningful interactive clarification rounds before clarifying can complete.

Each interactive confirmation prompt must have this structure:

```text
对于待确认点A，有多个方案:
1. aaa
2. bbb
3. ccc
4. [用户手动填入]
```

Rules:

- Replace `待确认点A` with a concrete unresolved requirement point.
- Provide at least 3 meaningful options before `[用户手动填入]`.
- Accept either a numbered choice or free-form human input.
- Record the point, requirement impact, options, selected answer, final value, and status in `brief.md`.
- A round is meaningful only if it changes or confirms goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case.
- If fewer than three answered meaningful rounds are recorded, ask the next confirmation prompt in the same format.
- After three answered meaningful rounds, keep asking only while key ambiguity remains.
- Do not ask filler questions solely to increase the count. If no meaningful next question can be identified, do not advance silently; ask the human for missing context or confirmation of omitted requirements.

Main Agent should use divergent thinking modes such as goal decomposition, scope tradeoffs, risk-first questioning, minimal vs complete versions, boundary examples, and user-provided free-form input.

The human may directly edit `brief.md` before invoking continue. On continue, Main Agent must read the latest file and use that file content as the source of truth for the next phase.

If `brief.md` has fewer than three answered meaningful interactive confirmation records, `$dm continue` / `/dm-continue` must not advance from `clarifying`; Main Agent must ask the next required meaningful confirmation prompt instead.

## Design Artifact Rule

`design.md` is the formal output of the designing phase.

Main Agent may create and revise `design.md` during `designing` through multi-turn discussion with the human. It should support divergent thinking modes such as alternative designs, architecture tradeoffs, implementation slicing, risk-first planning, validation-backward planning, and user-provided free-form input.

Main Agent must complete at least three meaningful interactive design confirmation rounds before `designing` can complete.

Each interactive design confirmation prompt must have this structure:

```text
对于设计待确认点A，有多个方案:
1. aaa
2. bbb
3. ccc
4. [用户手动填入]
```

Rules:

- Replace `设计待确认点A` with a concrete unresolved design point.
- Provide at least 3 meaningful options before `[用户手动填入]`.
- Accept either a numbered choice or free-form human input.
- Record the point, design impact, options, selected answer, final value, and status in `design.md`.
- A design round is meaningful only if it changes or confirms architecture, approach, scope slicing, tradeoffs, validation plan, acceptance criteria, rollout, rollback, or risk handling.
- If fewer than three answered meaningful design rounds are recorded, ask the next design confirmation prompt in the same format.
- After three answered meaningful design rounds, keep asking only while key design ambiguity remains.
- Do not ask filler questions solely to increase the count. If no meaningful next design question can be identified, do not advance silently; ask the human for missing context or confirmation of omitted design constraints.

The human may directly edit `design.md` before invoking continue. On continue, Main Agent must read the latest file and use that file content as the source of truth for design review and implementation handoff.

If `design.md` has fewer than three answered meaningful interactive design confirmation records, `$dm continue` / `/dm-continue` must not advance from `designing`; Main Agent must ask the next required meaningful design confirmation prompt instead.

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
- If current phase is `clarifying` and `brief.md` has fewer than three answered meaningful interactive confirmation records, do not advance; ask a required meaningful confirmation prompt.
- If current phase is `designing` and `design.md` has fewer than three answered meaningful interactive design confirmation records, do not advance; ask a required meaningful design confirmation prompt.
- If a report is present but its result cannot be determined, do not advance; ask for clarification or a corrected report.
- If current phase is `blocked`, do not advance until the blocking reason has human feedback.
- If current phase is `done`, do not advance.
- If `state.json` is missing or malformed, stop and do not infer transition from Markdown alone.
- Never skip a phase.
- Never advance more than one phase in one invocation.

## Event Format

Append one JSON line to `events.jsonl`:

```json
{"time":"{{TIMESTAMP}}","type":"phase_transition","actor":"main","from":"{{FROM_PHASE}}","to":"{{TO_PHASE}}","reason":"{{REASON}}"}
```

## User Response

Report:

- task id
- previous phase
- new phase, or missing requirements if not advanced
- files written
- next action

## Acceptance Criteria

- The platform continue command advances at most one phase.
- Missing required artifacts prevent advancement.
- Every phase transition appends one event.
- `brief.md` is the source of truth when leaving `clarifying`.
- `clarifying` cannot complete unless `brief.md` records at least three answered meaningful interactive confirmation prompts with multiple options and a manual input option.
- `design.md` is the source of truth when leaving `designing`, confirming design, and handing off to Worker/Test/Accept.
- `designing` cannot complete unless `design.md` records at least three answered meaningful interactive design confirmation prompts with multiple options and a manual input option.
- `decisions.md` is created or updated only when the current `design.md` is human-confirmed.
- Failed test/accept creates feedback and returns to `working`.
- Entering `human_acceptance` creates `.dm/session/[task-id]/summary.md`.
