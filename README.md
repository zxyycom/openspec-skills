# OpenSpec Skills

这个仓库维护一组面向 OpenSpec workflow 的 Codex skills。它存在的原因，是把 OpenSpec change 从零散命令调用整理成可复用的 agent 工作流：探索、提案、实现推进和归档各自有明确入口、边界和完成条件。

这个仓库不是 OpenSpec CLI 的替代品。它的价值在于让 agent 在使用 OpenSpec 时稳定读取项目事实、维护 artifacts、尊重开放问题门禁，并把用户确认的决策沉淀到正确位置。

## 项目意义

OpenSpec change 往往跨越需求澄清、方案设计、任务执行和最终归档。缺少明确分工时，agent 容易提前实现、跳过开放问题、把临时讨论写进长期 spec，或在归档前遗漏验证。

这个集合型 skill 仓库把流程拆成四类长期能力：

1. Explore：在实现前澄清问题、调查事实、比较方案和沉淀决策。
2. Propose：创建可进入实现阶段的 proposal、design、tasks 和 delta artifacts。
3. Apply Change：按任务清单推进实现，并同步任务状态、验证结果和阻塞说明。
4. Archive Change：在实现和验收完成后归档 change，并输出可审计摘要。

拆分后的目标不是增加步骤，而是让每个阶段都有清晰的进入条件、输出边界和暂停条件。

## 发展方向

这个仓库后续应继续强化四类能力：

1. 更稳定的 CLI 协作：优先通过 OpenSpec CLI 获取结构化状态、instructions、delta 和验证结果。
2. 更严格的阶段门禁：开放问题、artifact 缺失、任务未完成和验证失败都应有明确暂停或确认路径。
3. 更清楚的 artifact owner：proposal、design、tasks、spec delta 和主 spec 各自只承接适合长期维护的信息。
4. 更可审计的决策流：用户确认的范围、方案、边界、依赖和验证取舍应进入对应 change 的决策记录。

新增 OpenSpec skill 时，应先判断它是否代表独立阶段或稳定工作姿态。只是现有阶段中的步骤、引用规则或输出格式，应优先放回对应 skill。

## 与主仓库的关系

本仓库只维护 OpenSpec workflow skills 的本体和随 skill 分发的资料。共享校验、打包、CI、发布和多 skill 维护文档由 `skills` 主仓库承接。

这种拆分让 OpenSpec 相关技能可以作为一个集合独立演进，同时保持主仓库统一发现、校验和发布。
