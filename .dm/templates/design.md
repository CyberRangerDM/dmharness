# Design: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Drafted At: `{{DRAFTED_AT}}`
- Source Brief: `.dm/tasks/{{TASK_ID}}/brief.md`
- Confirmation Status: `{{DRAFT_OR_CONFIRMED}}`
- Confirmed At: `{{CONFIRMED_AT_OR_PENDING}}`
- Confirmed By: Human via platform continue command when status is `confirmed`

## How This Design Was Shaped

- Discussion mode: `{{DISCUSSION_MODE}}`
- Human free-form input: `{{HUMAN_INPUT_SUMMARY}}`
- Direct file edits considered: `{{YES_OR_NO}}`

## Goal

{{GOAL}}

## Requirements

{{REQUIREMENTS}}

## Scope

### In Scope

- {{IN_SCOPE_ITEM}}

### Out of Scope

- {{OUT_OF_SCOPE_ITEM}}

## Non-Goals

{{NON_GOALS}}

## Proposed Approach

{{PROPOSED_APPROACH}}

## Options Considered

| Option | Tradeoff | Outcome |
|---|---|---|
| {{OPTION}} | {{TRADEOFF}} | {{OUTCOME}} |

## Interactive Design Confirmation Records

At least three answered meaningful design records are required before leaving `designing`.

| ID | Design Pending Point | Design Impact | Options Presented | Human Choice | Final Value | Status |
|---|---|---|---|---|---|---|
| DC-001 | {{DESIGN_PENDING_POINT}} | {{ARCHITECTURE_APPROACH_SLICING_TRADEOFF_VALIDATION_ACCEPTANCE_ROLLOUT_RISK}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |
| DC-002 | {{DESIGN_PENDING_POINT}} | {{ARCHITECTURE_APPROACH_SLICING_TRADEOFF_VALIDATION_ACCEPTANCE_ROLLOUT_RISK}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |
| DC-003 | {{DESIGN_PENDING_POINT}} | {{ARCHITECTURE_APPROACH_SLICING_TRADEOFF_VALIDATION_ACCEPTANCE_ROLLOUT_RISK}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |

Meaningful design record rule: each record must affect or confirm architecture, approach, scope slicing, tradeoffs, validation plan, acceptance criteria, rollout, rollback, or risk handling. Filler questions do not count.

## Implementation Plan

1. {{STEP_1}}
2. {{STEP_2}}
3. {{STEP_3}}

## Files Expected To Change

| Path | Expected Change |
|---|---|
| `{{PATH}}` | {{EXPECTED_CHANGE}} |

## Validation Plan

{{VALIDATION_PLAN}}

## Acceptance Criteria

- {{ACCEPTANCE_CRITERION}}

## Risks

| Risk | Mitigation |
|---|---|
| {{RISK}} | {{MITIGATION}} |

## Source Of Truth

This file is the design phase output. Worker, Test, and Accept must read this file instead of relying only on conversation history.
