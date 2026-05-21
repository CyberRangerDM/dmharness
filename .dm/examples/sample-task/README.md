# Sample Task: End-to-End Harness-DM Flow

This sample demonstrates one complete Harness-DM task using the shared `.dm` file protocol.

Task id:

```text
202605121200-sample01
```

## Purpose

The sample verifies that Claude Code and Codex can share one task state without a self-developed CLI.

Claude trigger examples:

- `/dm-start Add sample setting`
- `/dm-continue 202605121200-sample01`

Codex trigger examples:

- `$dm start Add sample setting`
- `$dm continue 202605121200-sample01`

## Demonstrated Flow

```text
idle
  -> clarifying
  -> designing
  -> design_review
  -> design_persisted
  -> working
  -> testing
  -> working
  -> testing
  -> accepting
  -> human_acceptance
  -> done
```

The first test attempt fails, creates `feedback-001.md`, and returns to `working`. The second worker/test pass succeeds, then accept succeeds, then human acceptance completes the task.

`/dm-continue` and `$dm continue` are session recovery commands for interrupted tasks. They read `.dm/tasks/[task-id]`, `.dm/design/[task-id]`, and `.dm/session/[task-id]`, report completion if the task is done, and resume from the first incomplete required phase otherwise. They are not normal triggers in this sample flow.

## Recovery

To recover this task in a new session, read:

1. `AGENTS.md`
2. `.dm/workflow.md`
3. `.dm/examples/sample-task/tasks/202605121200-sample01/state.json`
4. `.dm/examples/sample-task/tasks/202605121200-sample01/summary.md`
5. `.dm/examples/sample-task/tasks/202605121200-sample01/brief.md`
6. `.dm/examples/sample-task/design/202605121200-sample01/design.md`
7. Latest role reports in `.dm/examples/sample-task/tasks/202605121200-sample01/`

## Notes

- No self-developed CLI is used.
- Guardrail Engine remains disabled/deferred in this sample.
- `brief.md` and `design.md` are the phase handoff files; later phases should read them rather than relying on conversation history.
- Done tasks must not advance again.
