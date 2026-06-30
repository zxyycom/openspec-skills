# 项目工具链

本文档是本仓库脚本、依赖安装、校验、打包和 CI 的 owner 文档。README 只保留入口说明，AGENTS 只引用本文档，具体标准在这里维护。

## 工具分工

1. pnpm 管安装：`packageManager` 使用 pnpm，锁文件使用 `pnpm-lock.yaml`，CI 安装依赖使用 `pnpm install --frozen-lockfile`。
2. Bun 管脚本调度和执行：本地和 CI 通过 `bun run <script>` 调用 package scripts，package scripts 内部直接调用 Bun 执行 TypeScript 脚本。
3. package scripts 是统一入口：本地和 CI 都通过 package scripts 调用脚本，不维护另一套命令。

## 脚本入口

当前脚本入口：

1. `bun run validate`: 校验四个 OpenSpec skill 入口、内部链接、包目录和项目级配置。
2. `bun run pack:skills`: 将 `skill/openspec-*/` 分别打包为 `dist/<skill-name>.zip`。
3. `bun run check`: 先校验，再打包。
4. `bun run deploy:package`: 生成本地可交付 zip 制品，不写入仓库外目录；CI 发布由 workflow 负责。

## 脚本标准

1. 脚本使用 TypeScript 编写，放在 `scripts/`。
2. 脚本应优先使用 Bun 和 Node 兼容的标准 API，避免为简单校验和打包引入依赖。
3. 脚本默认只读写仓库内路径；需要生成产物时输出到 `dist/`。
4. 打包脚本只打包 `skill/openspec-*/`，不把项目文档、CI、脚本或仓库元数据放进 skill zip。

## CI 标准

GitHub CI 复用本地入口：

1. 安装 Bun，用于执行 TypeScript 脚本。
2. 安装 pnpm，用于依赖安装。
3. 运行 `pnpm install --frozen-lockfile`。
4. 运行 `bun run check`。
5. 上传 `dist/*.zip` 作为 workflow artifact。
6. 对 `main` 分支的 `push` 和 `workflow_dispatch`，发布或更新 GitHub Release `openspec-skills-latest`，并上传全部 `dist/*.zip`。
