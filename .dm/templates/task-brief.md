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
- Intent Confidence: `{{LOW_MEDIUM_HIGH}}`
- Misunderstanding Check: `{{PASSED_OR_BLOCKED}}`
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

## Human Intent Model

- Pain Point / Trigger: {{PAIN_POINT_OR_TRIGGER}}
- Expected Final Artifact: {{EXPECTED_FINAL_ARTIFACT}}
- Primary User / Audience: {{PRIMARY_USER_OR_AUDIENCE}}
- Usage Scenario: {{USAGE_SCENARIO}}
- Success Standard: {{SUCCESS_STANDARD}}
- Preferred Tradeoff: {{PREFERRED_TRADEOFF}}

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

## Acceptance Examples

At least two concrete examples, checks, or observable signals are required before leaving `clarifying`. If examples are not suitable for the task, record the replacement acceptance signals.

| ID | Example / Check | Expected Result | Source |
|---|---|---|---|
| AE-001 | {{EXAMPLE_OR_CHECK}} | {{EXPECTED_RESULT}} | {{HUMAN_OR_INFERRED_SOURCE}} |
| AE-002 | {{EXAMPLE_OR_CHECK}} | {{EXPECTED_RESULT}} | {{HUMAN_OR_INFERRED_SOURCE}} |

## Misunderstanding Checks

List likely wrong interpretations before design starts. Key open misunderstandings block `designing`.

| ID | Possible Wrong Interpretation | Status | Resolution / Evidence |
|---|---|---|---|
| MC-001 | {{POSSIBLE_WRONG_INTERPRETATION}} | {{excluded_or_accepted_or_open}} | {{RESOLUTION_OR_EVIDENCE}} |
| MC-002 | {{POSSIBLE_WRONG_INTERPRETATION}} | {{excluded_or_accepted_or_open}} | {{RESOLUTION_OR_EVIDENCE}} |
| MC-003 | {{POSSIBLE_WRONG_INTERPRETATION}} | {{excluded_or_accepted_or_open}} | {{RESOLUTION_OR_EVIDENCE}} |

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

Meaningful record rule: each record must affect or confirm goal, scope, non-goals, constraints, acceptance criteria, risk, priority, or a key boundary case. Filler questions do not count. Each record must follow `.dm/specs/grill-me-discussion.spec.md`: one main question, recommended answer, recommendation reason, upstream dependency, and exploration evidence.

Intent-quality rule: the final record set must also make the Human intent model coherent enough for design. Before marking ready, Main Agent must verify that expected artifact, success standard, non-goals, misunderstanding checks, and acceptance examples are populated with concrete content rather than generic placeholders.

## Alternatives Considered

| Option | Why Considered | Outcome |
|---|---|---|
| {{OPTION}} | {{WHY_CONSIDERED}} | {{OUTCOME}} |

## Ready For Design

- Key ambiguity remaining: `{{YES_OR_NO}}`
- Key misunderstanding remaining: `{{YES_OR_NO}}`
- Acceptance examples ready: `{{YES_OR_NO}}`
- Intent model complete: `{{YES_OR_NO}}`
- Main Agent recommendation: `{{CONTINUE_OR_CLARIFY}}`

## Source Of Truth

This file is the clarifying phase output, written once after interactive clarification is complete. The design phase must read this file instead of relying only on conversation history.
