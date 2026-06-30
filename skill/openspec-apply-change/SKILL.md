---
name: openspec-apply-change
description: 实施 OpenSpec change 中的待办任务。适用于用户要求开始实现、继续实现或推进某个 change 的任务清单。
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

# OpenSpec Apply Change

## 目标

根据 OpenSpec change 的任务清单实施改动，并同步维护任务勾选、验证结果和阻塞说明。该 skill 面向“开始实现、继续实现、推进任务”的请求。

## Change 选择

1. 使用用户明确给出的 change 名称。
2. 用户未给名称时，从当前对话、最近正在处理的 change、或唯一活跃 change 推断目标。
3. 当存在多个候选或指代不唯一时，先列出候选并直接向用户确认。
4. 选定后说明 `Using change: <name>`；用户后续指定新名称时重新选择。

## CLI 使用策略

这些命令按场景选择，不是每次全量执行：

1. 必跑命令：
   - `openspec list --json`：先读取 active changes，确认目标 change 存在并辅助处理未命名请求。
   - `openspec status --change "<name>" --json`：确认 schema、artifact 状态和完成度。
   - `openspec instructions apply --change "<name>" --json`：获取 `state`、`progress`、`tasks`、`contextFiles` 和本轮实施指令。
2. 条件命令：
   - `openspec show "<name>" --type change --json --no-interactive`：需要理解 delta、capability 或 requirement 时运行。
   - `openspec validate "<name>" --type change --json --strict --no-interactive`：本轮改动完成后验证 change。
   - `openspec validate --specs --json --strict --no-interactive`：本轮改动触及主 specs 时运行。
3. 兜底读取：
   - CLI 不可用、命令失败或 JSON 中缺少完成任务所需正文时，再读取 `contextFiles` 指向的文件。
   - 需要参考改写前行为时，只读同目录 `reference-original.md`。

## 执行前开放问题门禁

执行任何实现任务前必须确认 change 没有未回答开放问题：

1. 按 `contextFiles` 读取包含 `## Open Questions` 的 artifact。
2. 发现未回答问题时立即暂停，不改代码、不勾选任务、不把问题当作实现假设。
3. 对 `已收敛` 条目检查是否仍有待选择、待确认或影响实现的歧义；存在歧义时暂停。
4. 用户回答后，先更新 artifact，并按归宿删除开放问题或标记为 `已收敛`，再重新进入 apply 流程。

## 流程

1. 运行 `openspec list --json`，确认 active changes 并锁定目标 change。
2. 运行 `openspec status --change "<name>" --json`，确认 `schemaName`、artifact 状态和当前完成度。
3. 运行 `openspec instructions apply --change "<name>" --json`，以其中的 `state`、`progress`、`tasks`、`contextFiles` 和 `instruction` 作为本轮实施入口。
4. 按状态处理：
   - `blocked`：说明缺失 artifact 或阻塞原因，并建议先补齐 change 材料。
   - `all_done`：说明任务已完成，并提示可进入归档。
   - 其他可执行状态：继续处理未完成任务。
5. 运行 `openspec show "<name>" --type change --json --no-interactive`，用结构化 delta 理解 capability、operation 和 requirement 变化；只需要 delta 时加 `--deltas-only`。
6. 对 CLI 未覆盖的 proposal、design、tasks 原文细节，按 `contextFiles` 精确读取对应文件。
7. 按“执行前开放问题门禁”检查 `## Open Questions`；存在未回答问题或已收敛歧义时停止在询问阶段。
8. 逐项实施未完成任务：
   - 说明当前任务。
   - 做与任务直接相关的最小必要改动。
   - 完成后立刻在 tasks 文件中把对应 checkbox 标为完成。
   - 长任务分段推进时持续报告当前任务编号和进度。
9. 运行与改动范围匹配的验证：
   - OpenSpec change 自身先运行 change 验证。
   - 涉及主 specs 时运行 specs 验证。
   - 涉及代码时运行相应格式化、静态检查、单元或集成测试。
10. 收尾时输出 change、schema、完成任务、总进度、验证结果、剩余项和阻塞点。

## 边界

1. 任务描述不清时，先提出一个具体问题或列出可选解释让用户确认。
2. 实现暴露设计或规格缺口时，说明缺口、影响 artifact 和可继续路径。
3. 命令失败或环境阻塞时，报告失败命令、关键错误和下一步可执行选项。
4. 改动范围保持与当前任务对齐；额外问题先记录为后续任务或待确认事项。

## 完成条件

满足以下任一条件即可结束本次执行：

1. 本轮可处理任务已完成，checkbox 已同步，验证已运行或原因已说明。
2. 所有任务已完成，并提示可进入 archive。
3. 存在开放问题、必须由用户确认的歧义、设计取舍或环境阻塞，且已暂停并给出具体下一步。
