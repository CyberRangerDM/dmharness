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
- `/dm-status 202605121200-sample01`
- `/dm-feedback 202605121200-sample01 Please fix the missing validation`

Codex trigger examples:

- `$dm start Add sample setting`
- `$dm continue 202605121200-sample01`
- `$dm status 202605121200-sample01`
- `$dm feedback 202605121200-sample01 Please fix the missing validation`

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
- Logical `/dm:status` is read-only and must not write `command-log.md`.
- Codex `$dm status` is not the same as Codex built-in `/status`.
- `brief.md` and `design.md` are the phase handoff files; later phases should read them rather than relying on conversation history.
- Done tasks must not advance again.
