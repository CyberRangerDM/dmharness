# Task Brief: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Created At: `{{CREATED_AT}}`
- Platform: `{{PLATFORM}}`
- Current Phase: `clarifying`

## DM Compact Summary

- Brief Status: `{{DRAFT_OR_ADJUSTMENT_REQUESTED_OR_CONFIRMED}}`
- Human Adjustment Decision: `{{NOT_ASKED_OR_NEEDS_ADJUSTMENT_OR_NO_ADJUSTMENT_NEEDED_OR_ADJUSTMENT_COMPLETE}}`
- Gate Status: `{{CLARIFYING_OR_READY_FOR_DESIGN}}`
- Key Decisions:
  - {{KEY_DECISION}}
- Open Ambiguities:
  - {{OPEN_AMBIGUITY_OR_NONE}}
- Source Freshness: `{{UPDATED_AT}}`

## Original Request

{{ORIGINAL_REQUEST}}

## How This Brief Was Shaped

- Clarification skill: `.dm/skills/grill-me.md`
- Human free-form input: `{{HUMAN_INPUT_SUMMARY}}`
- Direct file edits considered: `{{YES_OR_NO}}`
- Adjustment status: `{{ADJUSTMENT_STATUS}}`

## Confirmed Goal

{{CONFIRMED_GOAL}}

## Scope

### In Scope

- {{IN_SCOPE_ITEM}}

### Out of Scope

- {{OUT_OF_SCOPE_ITEM}}

## Constraints

- Runs inside Claude Code / Codex native CLI.
- No self-developed CLI is required.
- Guardrail Engine is deferred in phase 1.
- Project-specific constraints: {{PROJECT_SPECIFIC_CONSTRAINTS}}

## Acceptance Or Success Signals

| ID | Signal / Check | Expected Result | Source |
|---|---|---|---|
| AS-001 | {{SIGNAL_OR_CHECK}} | {{EXPECTED_RESULT}} | {{HUMAN_OR_INFERRED_SOURCE}} |

## Clarifications

| Question | Answer | Status |
|---|---|---|
| {{QUESTION}} | {{ANSWER}} | confirmed |

## Alternatives Considered

| Option | Why Considered | Outcome |
|---|---|---|
| {{OPTION}} | {{WHY_CONSIDERED}} | {{OUTCOME}} |

## Brief Adjustment

- Asked Human whether this brief needs adjustment: `{{YES_OR_NO}}`
- Human response: `{{NO_ADJUSTMENT_NEEDED_OR_NEEDS_ADJUSTMENT_OR_ADJUSTMENT_COMPLETE_OR_PENDING}}`
- Latest adjustment summary: `{{ADJUSTMENT_SUMMARY_OR_NONE}}`
- Main Agent recommendation: `{{CONTINUE_TO_DESIGN_OR_KEEP_CLARIFYING}}`

## Source Of Truth

This file is the clarifying phase output, written from `.dm/skills/grill-me.md` discussion and any direct human edits. The design phase must read the latest version of this file instead of relying only on conversation history.
