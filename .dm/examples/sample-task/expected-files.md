# Expected Files

## Required

- `.dm/examples/sample-task/README.md`
- `.dm/examples/sample-task/expected-files.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/state.json`
- `.dm/examples/sample-task/tasks/202605121200-sample01/events.jsonl`
- `.dm/examples/sample-task/tasks/202605121200-sample01/brief.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/summary.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/command-log.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/worker-result-001.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/test-report-001.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/feedback-001.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/worker-result-002.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/test-report-002.md`
- `.dm/examples/sample-task/tasks/202605121200-sample01/accept-report-001.md`
- `.dm/examples/sample-task/design/202605121200-sample01/design.md`
- `.dm/examples/sample-task/design/202605121200-sample01/decisions.md`
- `.dm/examples/sample-task/session/202605121200-sample01/summary.md`

## Acceptance Checks

- `state.json` is valid JSON.
- `events.jsonl` is valid JSONL.
- Final fixture has `status = done` and `phase = done`.
- Phase transitions do not skip required phases.
- Failed test creates `feedback-001.md` and returns to `working`.
- Numbered reports use `001`, `002`; no overwrite is implied.
- Session summary includes added, modified, deleted files and design decision changes.
- Claude and Codex examples share the same `.dm` protocol.
- Fixture is a final-state sample. `brief.md` records the clarified requirement, and `design.md` confirmation metadata demonstrates that the current design file was confirmed before implementation.
