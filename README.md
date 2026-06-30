# OpenSpec Skills

这个仓库用于维护 OpenSpec 相关 Codex skills。

## 项目定位

`openspec-skills` 是一个集合型 skill 仓库，维护 OpenSpec change 生命周期中常用的四个 workflow skill：

1. `openspec-propose`: 创建并补齐可进入实现阶段的 OpenSpec change artifacts。
2. `openspec-apply-change`: 根据 OpenSpec change 任务清单推进实现。
3. `openspec-archive-change`: 归档已完成的 OpenSpec change。
4. `openspec-explore`: 在 change 前后澄清问题、调查事实和比较方案。

本仓库不是应用项目。主要维护对象是可安装的 skill 包，以及支撑这些 skill 长期演进的项目级配置。

## 目录结构

- `.github/workflows/`: GitHub CI，用于校验、打包并发布 skill 制品。
- `AGENTS.md`: Codex agent 维护本仓库时的项目级指令。
- `docs/tooling.md`: 脚本、安装、校验、打包和 CI 的 owner 文档。
- `package.json`: 本地校验、打包和交付准备脚本入口。
- `scripts/`: 项目级自动化脚本。
- `skill/openspec-*/`: 可安装的 OpenSpec skill 包。
- `skill/openspec-*/SKILL.md`: 各 skill 入口和主执行流程。
- `skill/openspec-*/reference-original.md`: 需要回看原始行为时使用的参考材料。

## 工具链

脚本、依赖安装、校验、打包和 CI 的具体标准由 [docs/tooling.md](docs/tooling.md) 承接。

## 维护约定

skill 本体保持在 `skill/openspec-*/` 下。仓库根目录、`docs/`、`scripts/` 和 `.github/` 只放项目级说明、agent 指令、工具链说明、自动化脚本、CI 和仓库元数据。

项目文档只引用本仓库内的路径。需要说明外部来源、临时调查路径或一次性上下文时，放在对话或提交说明中，不写入长期项目文档。
