# Task Summary: {{TASK_TITLE}}

- Task ID: `{{TASK_ID}}`
- Status: `{{STATUS}}`
- Current Phase: `{{PHASE}}`
- Iteration: `{{ITERATION}}`
- Updated At: `{{UPDATED_AT}}`

## Task Goal

{{TASK_GOAL}}

## Confirmed Scope

{{CONFIRMED_SCOPE}}

Source: `.dm/tasks/{{TASK_ID}}/brief.md`

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

To resume this task, read:

1. `AGENTS.md`
2. `.dm/workflow.md`
3. `.dm/tasks/{{TASK_ID}}/state.json`
4. `.dm/tasks/{{TASK_ID}}/summary.md`
5. `.dm/tasks/{{TASK_ID}}/brief.md`
6. `.dm/design/{{TASK_ID}}/design.md`
7. Latest worker/test/accept report
