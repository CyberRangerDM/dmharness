# Task Brief: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Created At: `{{CREATED_AT}}`
- Platform: `{{PLATFORM}}`
- Current Phase: `clarifying`

## DM Compact Summary

- Confirmation Count: `{{CONFIRMATION_COUNT}}`
- Gate Status: `{{BLOCKED_OR_READY_FOR_DESIGN}}`
- Key Decisions:
  - {{KEY_DECISION}}
- Open Ambiguities:
  - {{OPEN_AMBIGUITY_OR_NONE}}
- Source Freshness: `{{UPDATED_AT}}`

## Original Request

{{ORIGINAL_REQUEST}}

## How This Brief Was Shaped

- Write mode: final one-shot after interactive clarification
- Discussion mode: `{{DISCUSSION_MODE}}`
- Human free-form input: `{{HUMAN_INPUT_SUMMARY}}`
- Direct file edits considered: `{{YES_OR_NO}}`

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

## Clarifications

| Question | Answer | Status |
|---|---|---|
| {{QUESTION}} | {{ANSWER}} | confirmed |

## Interactive Confirmation Records

At least three answered meaningful records are required before leaving `clarifying`.

| ID | Pending Point | Requirement Impact | Options Presented | Recommended Answer | Recommendation Reason | Upstream Dependency | Exploration Evidence | Human Choice | Final Value | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| IC-001 | {{PENDING_POINT}} | {{GOAL_SCOPE_CONSTRAINT_ACCEPTANCE_RISK_PRIORITY_OR_BOUNDARY}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{RECOMMENDED_ANSWER}} | {{RECOMMENDATION_REASON}} | {{UPSTREAM_DEPENDENCY_OR_NONE}} | {{FILES_OR_FACTS_CHECKED}} | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |
| IC-002 | {{PENDING_POINT}} | {{GOAL_SCOPE_CONSTRAINT_ACCEPTANCE_RISK_PRIORITY_OR_BOUNDARY}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{RECOMMENDED_ANSWER}} | {{RECOMMENDATION_REASON}} | {{UPSTREAM_DEPENDENCY_OR_NONE}} | {{FILES_OR_FACTS_CHECKED}} | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |
| IC-003 | {{PENDING_POINT}} | {{GOAL_SCOPE_CONSTRAINT_ACCEPTANCE_RISK_PRIORITY_OR_BOUNDARY}} | 1. {{OPTION_A}}<br>2. {{OPTION_B}}<br>3. {{OPTION_C}}<br>4. [用户手动填入] | {{RECOMMENDED_ANSWER}} | {{RECOMMENDATION_REASON}} | {{UPSTREAM_DEPENDENCY_OR_NONE}} | {{FILES_OR_FACTS_CHECKED}} | {{HUMAN_CHOICE}} | {{FINAL_VALUE}} | answered |

Meaningful record rule: each record must affect or confirm goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case. Filler questions do not count. Each record must follow `specs/grill-me-discussion.spec.md`: one main question, recommended answer, recommendation reason, upstream dependency, and exploration evidence.

## Alternatives Considered

| Option | Why Considered | Outcome |
|---|---|---|
| {{OPTION}} | {{WHY_CONSIDERED}} | {{OUTCOME}} |

## Ready For Design

- Key ambiguity remaining: `{{YES_OR_NO}}`
- Main Agent recommendation: `{{CONTINUE_OR_CLARIFY}}`

## Source Of Truth

This file is the clarifying phase output, written once after interactive clarification is complete. The design phase must read this file instead of relying only on conversation history.
