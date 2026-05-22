# Guardrails

Guardrail Engine is deferred in phase 1.

Current behavior:

- No command interception is enabled.
- Claude Code and Codex native permission behavior is unchanged.
- No hook is installed to block tool execution.
- Other workflow modules must not depend on Guardrail Engine enforcement.

Future behavior, if enabled, will follow `.dm/specs/guardrail-engine.spec.md`.
