---
name: openspec-archive-change
description: 归档已完成的 OpenSpec change。用于用户要求在实现、验收或同步评估完成后 finalize、archive 或归档某个 change。
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

# OpenSpec Change 归档

## 目标

使用 OpenSpec CLI 归档一个 active change，并在归档前确认 artifact、task、delta spec 和验证状态。归档后输出可审计摘要。

## Change 选择

1. 使用用户明确给出的 change 名称。
2. 用户未给名称时，从当前对话或最近处理的 change 推断目标。
3. 当多个候选都合理或上下文不足时，先向用户确认。
4. 已归档 change 只作为历史参考，不进入本流程。

## CLI 使用策略

这些命令按归档检查需要选择，不是清单式全量执行：

1. 必跑命令：
   - `openspec list --json`：先读取 active changes，确认目标 change 仍处于可归档状态。
   - `openspec status --change "<name>" --json`：读取 schema 和 artifact 状态。
   - `openspec archive "<name>" --yes`：执行实际归档并跳过 CLI 确认提示；`--yes` 不跳过 spec 更新或验证。
2. 条件命令：
   - `openspec instructions apply --change "<name>" --json`：需要检查任务完成度时运行。
   - `openspec show "<name>" --type change --json --no-interactive`：需要评估 delta spec 同步状态时运行。
   - `openspec show "<spec>" --type spec --json --no-interactive`：需要对照主 spec 当前内容时运行。
3. 归档参数：
   - `--yes`：默认必须追加，避免确认提示卡住非交互执行。
   - `--skip-specs`：仅在用户明确跳过 spec 更新时追加。
   - `--no-validate`：仅在用户明确跳过验证并接受风险时追加。
4. 兜底读取：
   - OpenSpec CLI 负责归档移动和 spec 更新；常规流程中不手动创建 archive 目录或移动 change 目录。
   - CLI 不可用或失败时，读取 `reference-original.md` 和目标 change 文件，报告失败命令、错误摘要和用户可确认的兜底方案。

## 流程

1. 运行 `openspec list --json`，确认 active changes。
2. 确认目标 change，并在目标不唯一时向用户确认。
3. 运行 `openspec status --change "<name>" --json`，读取 `schemaName` 和 artifact 状态。
4. 运行 `openspec instructions apply --change "<name>" --json`，用 `progress` 和 `tasks` 判断任务完成状态；命令未返回任务信息时再读取 tasks 文件。
5. 对未完成 artifact 或 task，列出风险并取得用户明确确认后继续。
6. 运行 `openspec show "<name>" --type change --json --no-interactive`：
   - 无 delta 时记录 `No delta specs`。
   - 有 delta 时展示 capability、operation 和 requirement 摘要。
   - 需要对照主 spec 时运行 `openspec show "<spec>" --type spec --json --no-interactive`。
7. 选择归档命令：
   - 默认：`openspec archive "<name>" --yes`
   - 用户明确跳过 spec 更新：`openspec archive "<name>" --skip-specs --yes`
   - 用户明确跳过验证并接受风险：`openspec archive "<name>" --no-validate --yes`
8. 执行归档命令。CLI 报告目标冲突、验证失败或其它错误时停止，并给出下一步选项。

## 边界

1. 归档会改变 active change 状态；执行前完成状态检查、风险说明和必要确认。
2. Artifact 或 task 未完成时，用户确认是继续归档的前置条件。
3. `--skip-specs` 和 `--no-validate` 只在用户明确选择对应取舍时使用。
4. CLI 失败时不模拟目录移动，先报告失败并等待用户选择兜底方案。

## 输出

归档完成后输出：

```text
Change: <change-name>
Schema: <schema-name>
Archived to: <path from CLI>
Specs: <synced | no delta specs | skipped | warning>
Artifacts: <complete | incomplete confirmed>
Tasks: <complete | incomplete confirmed | no tasks>
Warnings: <none | summary>
```
