# Task Summary: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Status: `{{STATUS}}`
- Current Phase: `{{PHASE}}`
- Iteration: `{{ITERATION}}`
- Updated At: `{{UPDATED_AT}}`

## DM Compact Summary

- Current Phase: `{{PHASE}}`
- Gate Status: `{{GATE_STATUS}}`
- Latest Worker Result: `{{WORKER_STATUS}}`
- Latest Test Result: `{{TEST_STATUS}}`
- Latest Accept Result: `{{ACCEPT_STATUS}}`
- Next Action: {{NEXT_ACTION}}

## Task Goal

{{TASK_GOAL}}

## Confirmed Scope

{{CONFIRMED_SCOPE}}

Source: `.dm/tasks/{{TASK_ID}}/brief.md` (`deferred` until clarify finalization when not yet written)

## Confirmed Design Summary

{{DESIGN_SUMMARY}}

Source: `.dm/design/{{TASK_ID}}/design.md`

## Latest Role Results

| Role | Status | Latest Report |
|---|---|---|
| worker | {{WORKER_STATUS}} | {{WORKER_REPORT}} |
| test | {{TEST_STATUS}} | {{TEST_REPORT}} |
| accept | {{ACCEPT_STATUS}} | {{ACCEPT_REPORT}} |

## Latest Feedback

{{LATEST_FEEDBACK}}

## Next Action

{{NEXT_ACTION}}

## Recovery Notes

To resume this task, read the minimum set first:

1. `.dm/tasks/{{TASK_ID}}/state.json`
2. `.dm/tasks/{{TASK_ID}}/summary.md`
3. Current phase artifact compact summary and required records
4. Latest worker/test/accept report compact summary when relevant

Read full workflow/spec/artifact files only when compact summaries are missing, stale, or insufficient for the current phase gate.
