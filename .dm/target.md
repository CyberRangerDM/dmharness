# My Harness Engineering — Product Requirements

## 目标用户
 目前用户就我, 我希望定制开发出我自己的 Harness Engineering. 当我提出一个任务, 一个想法, 一个需求时, 它能让 AI Agent 尽可能的与我讨论细节, 而后将讨论的结果实现.
 在写代码场景, 我希望通过这套 Harness Engineering, 我不再手动介入修改代码, 我只需要看结果并验收成果. 

## 功能特点清单
1. 对于任务执行, 尽量稳定, 有一个固定且可拓展的流程: clarify 阶段需要和我互动澄清; clarify 完成后自动进入 design 并产出设计; design_review 由 Main Agent 自动校验和持久化, 后续 worker/test/accept/summary/done 自动完成, 不需要我输入 continue 推进
2. 危险操作有围栏机制阻挡, 除非获得同意, 且围栏要由代码驱动, 而非提示词LLM驱动
3. 设计决策需求持久化在项目的 .dm/design 目录下
4. 任务状态持久化, 避免下次重新打开会话, 全靠记忆. 就算完全推导重来, Agent 也能根据这些持久化数据恢复记忆. 任务数据保存在项目的 .dm/tasks 目录下
5. 同时支持 claude code && codex
6. 对于一个任务, 使用多 agent, 主 agent 负责调度, 其他 agent 类型为 worker / test / accept, 其中 accept 覆盖 review 与尽可能模拟人类验收
7. 我的定制 Harness Engineering 要做到代码驱动结合 LLM 驱动, 而哪些逻辑由代码驱动, 哪些由 LLM 驱动, 实现时与人类讨论, 并由人类来决定判断
8. 每次任务完成后, 我可以知道发生了哪些变化, 代码, 文件, 上下文
9. 将一些多次出现的原则类条款, 持续沉淀, 并将这些条款持久化在目录 .dm/specs 中
10. 在当前会话上下文即将耗尽时, 主动与人类沟通, 进而得知是压缩, 还是重新开始新会话, 还是给当前会话的上下文瘦身


## 参考项目及借鉴点
你可以借鉴这几个开源项目:
1.https://github.com/tim-hub/powerball-harness 
2.https://github.com/obra/superpowers
3.https://github.com/mindfold-ai/trellis

## 约束条件
只支持 Claude Code + Codex，个人使用，轻量优先, 极简风格, 指令明确. 第一阶段不开发独立 CLI, 在 Claude Code / Codex 原生 CLI 内通过 adapter 和文件协议运行. Guardrail Engine 第一阶段允许 deferred.
