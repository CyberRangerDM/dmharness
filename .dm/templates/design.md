# Design: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Drafted At: `{{DRAFTED_AT}}`
- Source Brief: `.dm/tasks/{{TASK_ID}}/brief.md`
- Confirmation Status: `{{DRAFT_OR_CONFIRMED}}`
- Confirmed At: `{{CONFIRMED_AT_OR_PENDING}}`
- Confirmed By: Human via platform continue command when status is `confirmed`

## DM Compact Summary

- Gate Status: `{{DRAFT_BLOCKED_READY_FOR_REVIEW_OR_CONFIRMED}}`
- Key Design Decisions:
  - {{KEY_DESIGN_DECISION}}
- Validation Summary: {{VALIDATION_SUMMARY}}
- Open Risks:
  - {{OPEN_RISK_OR_NONE}}
- Source Freshness: `{{UPDATED_AT}}`

## How This Design Was Shaped

- Design mode: `autonomous`
- Source brief summary: {{SOURCE_BRIEF_SUMMARY}}
- Project files/specs inspected:
  - {{FILES_OR_SPECS_INSPECTED}}
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

## Design Review Notes

- Review status: `{{DRAFT_READY_FOR_HUMAN_APPROVAL_OR_CONFIRMED}}`
- Human approval command: `{{PLATFORM_CONTINUE_COMMAND}}`
- Human edits considered: `{{YES_OR_NO}}`

`designing` is autonomous and does not require interactive design confirmation records.

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
