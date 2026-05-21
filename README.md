# harness-dm

Lightweight delivery-management workflow for Claude Code and OpenAI Codex.

`harness-dm` turns an AI coding session into a file-backed workflow: clarify the task with the human, then automatically design, implement, test, accept, and hand back the result.

## Why

AI agents are good at doing work, but long tasks need checkpoints, memory, and review.

`harness-dm` keeps those in the repo:

- human-controlled clarification and feedback
- persistent task state under `.dm/tasks/`
- persistent design decisions under `.dm/design/`
- Worker / Test / Accept role separation
- shared protocol for Claude Code and Codex

## Workflow

```text
clarifying
  -> designing
  -> design_review
  -> working
  -> testing
  -> accepting
  -> human_acceptance
  -> done
```

The important rule: agents do not move past requirements silently. After the clarify rounds produce `brief.md`, the remaining phases run automatically unless a blocker or feedback loop is recorded. `brief.md` and `design.md` are the handoff files.

## Usage

Claude Code:

```text
/dm-start [title]
/dm-status [task-id]
/dm-feedback [task-id] [text]
```

Codex:

```text
$dm start [title]
$dm status [task-id]
$dm feedback [task-id] [text]
```

`/dm-continue` and `$dm continue` are legacy/recovery commands only; normal tasks started with `dm start` do not need continue after clarify.

There is no standalone `harness-dm` CLI in phase 1. Commands are executed by Claude Code / Codex through the `.dm` file protocol.

## Files

```text
.dm/workflow.md              # workflow definition
.dm/commands/                # command behavior
.dm/agents/                  # Worker / Test / Accept roles
.dm/tasks/[task-id]/         # task state, brief, reports, feedback
.dm/design/[task-id]/        # design, decisions, revisions
.dm/session/[task-id]/       # final human-facing summary
AGENTS.md                    # Codex entrypoint
CLAUDE.md                    # Claude Code entrypoint
```

## Current Status

Phase 1 is file-protocol based:

- Claude Code and Codex are supported.
- Worker / Test / Accept run sequentially.
- Guardrail Engine is specified but not enabled.
- Custom blocking hooks are not installed.

For the full protocol, read `.dm/workflow.md`.
