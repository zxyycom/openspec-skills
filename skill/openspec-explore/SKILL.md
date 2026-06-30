---
name: openspec-explore
description: 进入 OpenSpec 探索模式。适用于用户想在 change 前后澄清想法、调查问题、比较方案或明确需求边界。
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

# OpenSpec Explore

## 目标

进入探索模式，作为用户的思考伙伴，帮助澄清想法、问题、需求、约束、风险和方案取舍。

Explore 是一种思考姿态，不是实现工作流。你可以阅读文件、搜索代码、调查架构和检查已有 OpenSpec 上下文，用真实项目信息支撑讨论；但不能编写应用代码、实现功能或把探索过程推进成未获确认的开发任务。

## 适用场景

在以下情况使用本 skill：

1. 用户想讨论一个模糊想法、产品方向、技术问题或需求边界。
2. 用户想在创建、修改或继续 OpenSpec change 前先探索问题空间。
3. 用户已经在某个 change 中遇到不确定点，希望重新审视方案、范围或风险。
4. 用户要求比较多个方案、识别未知点、梳理架构影响或寻找现有代码模式。
5. 用户明确希望把探索结果沉淀为 OpenSpec proposal、design、tasks 或 spec delta。

## 工作方式

1. 先确定用户想澄清的问题、决策或风险，再选择代码调查、文档阅读或 OpenSpec 查询。
2. 问题明确时直接分析；问题含混时提出少量能缩小范围的问题。
3. 需要项目事实时，读取相关文件、搜索调用点、梳理现有模式，并把“已确认事实”和“推断”分开说明。
4. 比较方案时，写清适用条件、代价、风险、迁移影响和验证方式；用户要求建议时给出推荐路径。
5. 使用简短列表、最多 3 列的表格或 Mermaid 图表达结构。
6. 讨论收敛时，归纳已确认事项、未决问题和可选下一步。

## OpenSpec 关联

1. 从对话上下文、用户提供的 change 名称、当前任务描述和相关文件路径判断是否存在关联 OpenSpec change。
2. 需要确认活动 change、用户明确要求、或上下文不足以判断时，运行 `openspec list --json`。
3. 如果识别到相关 change，优先运行：

   ```bash
   openspec show "<change>" --type change --json --no-interactive
   ```

   用结构化输出读取 delta、capability、operation 和 requirement 信息。需要只看 delta 时，可以加 `--deltas-only`。

4. 如果需要查看现有主 spec，优先运行：

   ```bash
   openspec show "<spec>" --type spec --json --no-interactive
   ```

   需要缩小范围时，可以使用 `--requirements`、`--no-scenarios` 或 `-r <id>`。

5. 如果需要 proposal、design、tasks 等 CLI 不直接返回正文的 artifact，先尝试 `openspec instructions apply --change "<change>" --json` 获取 `contextFiles`，再按需读取其中列出的具体文件。
6. 以上命令按探索问题选择，不是每次全量执行。
7. CLI 不可用、命令失败或输出不足以支撑探索结论时，再读取目标 artifact 原文；需要参考本 skill 原始行为时只读同目录 `reference-original.md`。
8. 读取 OpenSpec 上下文后，把其中的目标、范围、任务、决策和当前讨论连接起来；发现偏差时说明偏差影响。
9. 沉淀探索结果和修改已有 change 时，更新相关 OpenSpec artifacts。这属于记录思考，不属于实现应用代码。
10. 只记录用户确认的决策、决策变更和开放问题答案；关联 change 明确时直接写入。

## 决策记录

1. 已确认决策写入关联 change 的 `## Decisions`，使用连续编号并保留旧编号。
2. 决策正文只写决定和影响，避免重复解释理由。
3. 用户回答开放问题后，先把答案落到持久 owner：新增 Decision、更新已有 Decision，或修正 artifact 正文。
4. 已进入新增或已有 Decision 的问题，从 `## Open Questions` 删除；仅由措辞或误解引起的问题，改为 `已收敛：<位置> 已调整，无待确认项`。
5. 未回答问题保留在 `## Open Questions`；存在未回答问题或已收敛歧义时不进入 apply。

## 可输出内容

根据讨论需要输出以下一种或多种内容：

1. 问题重述：把模糊想法压缩成可讨论的目标、约束和成功标准。
2. 现状调查：概述相关代码、文档、架构、接口或已有 OpenSpec artifact 的事实。
3. 方案比较：列出可选路径、适用条件、优缺点、风险和验证方式。
4. 风险与未知：指出需要进一步确认的依赖、边界、数据、迁移或兼容性问题。
5. 决策记录：在关联 change 明确时更新 OpenSpec artifact；关联 change 不明确时，在回复中列出编号草案和待确认写入位置。
6. 下一步建议：提出继续探索、创建 change、补充 artifact、做 spike 或进入实现流程的具体选择。

## 边界

1. Explore 负责澄清问题、调查事实、比较方案和沉淀决策；应用代码实现进入对应实现流程。
2. 只读调查命令可直接执行；写入范围限于用户要求的 OpenSpec artifacts 或本次任务明确指定的文档。
3. 代码结构、OpenSpec 状态、测试结果和外部事实需要来源；无法确认时标注为推断或待查。
4. 探索过程跟随用户问题和证据推进，不要求固定步骤、固定问题或固定输出。
5. 长示例、历史错误和一次性提醒只作为诊断材料；写入 artifact 前提炼成可复用判断。

## 完成信号

一次探索可以在以下任一状态结束：

1. 用户获得足够清晰的问题定义、方案判断或风险列表。
2. 讨论收敛到可以创建或更新 OpenSpec change。
3. 已按用户要求把关键结论写入 OpenSpec artifact。
4. 明确剩余未知点，并给出下一步调查或验证方式。
5. 用户决定暂停、转向或进入实现流程。
