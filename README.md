# OpenSpec Skills

`openspec-skills` 是对 OpenSpec skills 的二次开发。它的起点不是重新发明 OpenSpec，而是原有 OpenSpec skill 文本在实际使用中还不够理想：阶段边界不够清楚，指令脉络不够稳定，很多要求需要进一步改写，才能更好地引导 agent 完成 OpenSpec 工作流。

OpenSpec 本身提供的是规格和变更流程；这个项目关注的是 agent 如何理解并执行这些流程。它要优化的是 skill 文本质量、任务进入姿态、artifact 写作边界、开放问题门禁和阶段之间的交接方式。

## 为什么需要它

OpenSpec 的工作流天然分阶段：先探索问题，再形成 proposal、design、tasks 和 spec delta，然后执行任务，最后归档 change。对人类来说，这些阶段可以通过经验判断切换；对 agent 来说，如果 skill 文本没有清楚地区分阶段目标、暂停条件和输出位置，就容易提前实现、跳过问题确认、写错 artifact owner，或者把临时讨论带进长期规范。

`openspec-skills` 想解决的正是这一层问题。它把 OpenSpec 的流程重新写成更适合 agent 的工作方式，让每个阶段都有明确的任务姿态：什么时候只探索，什么时候可以生成 change，什么时候可以实现，什么时候可以归档。

## 希望形成的能力

这个项目希望形成一组更高质量的 OpenSpec agent skills：

1. Explore 负责澄清问题、调查事实和比较方案，不提前进入实现。
2. Propose 负责把需求整理成可进入实现阶段的 artifacts。
3. Apply Change 负责按任务清单推进实现，并同步任务状态和验证结果。
4. Archive Change 负责在实现和验收完成后归档，并保留可审计摘要。

这些 skill 的重点不是包装 CLI 命令，而是把 OpenSpec 的流程语义转成 agent 可以稳定遵守的文本指令。

## 发展方向

后续的发展重点会放在 OpenSpec skill 文本本身的质量提升上：更清楚的阶段边界，更少的歧义入口，更可靠的开放问题处理，更准确的 artifact owner，以及更稳定的 CLI 和文档协作方式。

长期来看，这个项目希望成为一组适合真实项目使用的 OpenSpec agent 工作流技能。它应该让 agent 更少凭感觉推进流程，更多依据清楚的阶段目标、已确认决策和可验证状态行动。
