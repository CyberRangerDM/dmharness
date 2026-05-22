---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree.
source: ../../../.dm/skills/grill-me.md
metadata:
  short-description: Clarify plans by focused questioning
  version: 1.0.0
---

# Grill-Me Skill

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Harness-DM Clarify Contract

When Harness-DM enters `clarifying`, apply this as an active interaction mode, not as background reading:

- Ask exactly one main question in the visible response.
- Present the prompt in the user's language. Use an explicit question heading such as `Question 1:` or, in Chinese, `第 1 个问题：`.
- Include the agent's recommended answer with that question. Localize the label when appropriate, for example `Recommended answer:` in English or `我的推荐答案：` in Chinese.
- Choose the next question by resolving upstream decisions before downstream details.
- For product/app/system requests, treat the primary user/audience and their job-to-be-done as the first unresolved upstream decision unless the human already stated it. Do not make product shape, delivery surface, framework, data source, or implementation architecture the first question when the intended user is still unclear.
- When the question asks the human to choose among plausible directions, include a compact numbered list of concrete options after the recommendation. Keep the list tied to the single main question.
- Inspect local files before asking for facts that can be discovered locally.
- Do not replace or prefix the question with a workflow status report. If Harness-DM metadata must be shown because the human asked for it or a blocker requires it, keep it after the question and brief.
- Do not write `brief.md` until the answers are sufficient to summarize the requirement for design.
- Do not persist each clarify question or answer to `events.jsonl`, and do not
  update task `summary.md` after every exchange. Keep intermediate answers in
  the live conversation and batch them into the formal clarify artifact when the
  discussion is ready.
