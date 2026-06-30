---
name: openspec-propose
description: 为新的 OpenSpec change 生成可进入实现阶段的 proposal、design、tasks 等 artifacts。用于用户给出 change name 或需求描述，并希望一次性完成提案材料。
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

# OpenSpec Propose

核心：新增 change 只在 `openspec/changes/<name>/` 内生成中文、未审核的临时 artifacts，用一句话锚定目标，并以阻塞级审计任务作为实现前置门禁。

## 目标

根据用户给出的 change name 或需求描述，创建一个新的 OpenSpec change，并生成进入实现阶段所需的 artifacts。完成后，该 change 应满足 `openspec status --change "<name>" --json` 中 `applyRequires` 指向的 artifacts 全部为 `done`；但在阻塞级审计任务完成前，该 change 只达到 artifact 准备状态，不可进入实现执行。

## 输入

用户可以提供以下任一形式：

1. 明确的 kebab-case change name，例如 `add-user-auth`。
2. 自然语言需求描述，例如“给文档导航增加 Markdown 章节搜索”。

当用户只提供需求描述时，从描述中派生简短、稳定、语义明确的 kebab-case 名称。名称应使用英文小写、数字和连字符，并表达 change 的核心行为。

当需求目标、范围或名称无法可靠判断时，直接向用户提问，要求补充“要构建或修复什么”。在理解目标前先暂停，不创建 change。

## CLI 使用策略

这些命令按创建进度选择，不是每次全量执行：

1. 必跑命令：
   - `openspec list --json`：先读取当前 active changes，避免重复创建或忽略正在进行的 change。
   - `openspec list --specs --json`：写 proposal 的 Capabilities 前读取现有主 spec id，避免把 change 名称误当成 capability。
   - `openspec new change "<name>" --description "<one-line goal>" --schema "<schema>"`：创建 change。使用默认 schema 时可以省略 `--schema`。
   - `openspec status --change "<name>" --json`：读取 `applyRequires`、artifact 状态和依赖。
   - `openspec instructions <artifact-id> --change "<name>" --json`：为当前可生成 artifact 获取 `template`、`instruction`、`outputPath`、`dependencies`、`context` 和 `rules`。
   - `openspec validate "<name>" --type change --json --strict --no-interactive`：artifact 生成完成后验证 change。
2. 条件命令：
   - `openspec schemas --json`：需要确认可用 schema，或用户指定 schema 时运行。
   - `openspec templates --schema "<schema>" --json`：需要查看模板来源或排查 artifact 结构时运行。
   - `openspec show "<spec>" --type spec --json --no-interactive`：需要读取已有主 spec 时运行。
   - `openspec show "<change>" --type change --json --no-interactive`：需要读取已有 change delta 时运行。
3. 兜底读取：
   - CLI 不可用、命令失败或输出不足时，再读取目标文件原文。
   - 需要参考改写前行为时，只读同目录 `reference-original.md`。

## Capability ID 命名规则

OpenSpec change name 和 capability ID 是不同概念：

1. Change name 表达本次要完成的变化，可以是动词短语，例如 `implement-docnav-adapter-management`。
2. Capability ID 表达长期主 spec 所有权，必须是稳定名词短语，例如 `adapter-management`。
3. Delta spec 的目录名就是归档目标：`openspec/changes/<change>/specs/<capability>/spec.md` 会合并到 `openspec/specs/<capability>/spec.md`。

写 proposal 的 Capabilities 前必须选择 capability ID：

1. 运行 `openspec list --specs --json`，读取现有 capability ID。
2. 如果需求改变已有能力的 requirement，使用现有 capability ID，不能创建同义新 ID。
3. 如果需求确实引入新的长期能力，创建新的 capability ID。
4. 如果一个 change 同时改变多个长期能力，列出多个 capability；不要为整个 change 创建一个总括 capability。
5. 如果无法判断应该复用哪个 capability，先向用户问一个具体问题，不继续生成 specs。

新 capability ID 必须满足：

1. 使用 kebab-case，小写英文、数字和连字符。
2. 使用名词或名词短语，表达长期能力或稳定责任边界。
3. 优先按产品/接口/制品所有权命名，例如 `core-cli`、`adapter-routing`、`readable-output`、`adapter-management`、`markdown-navigation`。
4. 不默认复用 change name。
5. 不包含 `implement`、`implementation`、`change`、`task`、日期或一次性迁移阶段。
6. 不用 `v0`、`v1` 等版本阶段表达长期能力；版本范围写入 requirement、design 或 tasks。

示例：

1. `replace-text-with-readable-view` -> capability `readable-output`。
2. `implement-docnav-adapter-management` -> capability `adapter-management`。

## 决策记录

创建和更新 change 时，把用户已确认的范围、方案、边界、依赖、验证和兼容性取舍记录到 `## Decisions`。

1. 使用连续编号：`### Decision 1: <short title>`、`### Decision 2: <short title>`；已有编号不重排，新增决策追加。
2. 决策正文只写决定和影响，避免重复解释理由。
3. 用户回答 `## Open Questions` 后，先把答案落到持久 owner：新增 Decision、更新已有 Decision，或修正 artifact 正文。
4. 已进入新增或已有 Decision 的问题，从 `## Open Questions` 删除；仅由措辞或误解引起的问题，改为 `已收敛：<位置> 已调整，无待确认项`。
5. 未回答问题保留在 `## Open Questions`；没有未回答问题且已收敛条目无歧义时，写明“无未回答开放问题，可以进入实现前审计”。

## 工作流程

1. 确定 change 名称、schema 和目标
   - 先运行：

     ```text
     openspec list --json
     ```

   - 用 active change 列表检查名称冲突和相关在途工作。
   - 从用户输入识别现成的 kebab-case 名称，或从需求描述派生名称。
   - 将用户需求压缩成一句目标说明，作为后续 artifact 写作的主线。
   - 运行 `openspec list --specs --json`，记录现有 capability ID，并按“Capability ID 命名规则”初步判断本 change 应修改或新增哪些 capability。
   - 如果同名 change 已存在，先确认用户要继续该 change，还是改用新名称；得到选择后再继续。
   - 需要确认可用 workflow schema 时，运行：

     ```text
     openspec schemas --json
     ```

   - 用户未指定 schema 时，使用项目默认 schema；需要查看模板来源时运行 `openspec templates --schema "<schema>" --json`。

2. 创建 change
   - 运行：

     ```text
     openspec new change "<name>" --description "<one-line goal>" --schema "<schema>"
     ```

   - 如果使用默认 schema 且 CLI 或项目约定不要求显式传入，可以省略 `--schema "<schema>"`。
   - 如果没有可用的一句话描述，可以省略 `--description`，但应优先传入用户目标摘要。
   - 预期生成 `openspec/changes/<name>/` 和 `.openspec.yaml`。
   - 创建后确认 change 目录存在。
   - 创建阶段只写入当前 change 目录；除用户明确要求外，不修改现有 specs、docs、schemas、examples 或其它 change。

3. 读取 artifact 状态
   - 运行：

     ```text
     openspec status --change "<name>" --json
     ```

   - 从 JSON 中读取：
     - `applyRequires`：实现前必须完成的 artifact ID。
     - `artifacts`：每个 artifact 的状态、依赖和可写条件。
   - 记录 artifact 进度，按依赖关系选择当前可生成的 artifact。

4. 生成 apply-ready artifacts
   - 持续处理 artifact，直到 `applyRequires` 中所有 artifact 的状态都是 `done`。
   - 每次只处理状态允许生成、依赖已经满足的 artifact。
   - 对每个 artifact 运行：

     ```text
     openspec instructions <artifact-id> --change "<name>" --json
     ```

   - 按 instructions JSON 执行：
     - `template`：输出文件结构。
     - `instruction`：artifact 写作要求。
     - `outputPath`：写入位置。
     - `dependencies`：写作前读取的已完成 artifact。
     - `context` 和 `rules`：约束和判断依据。
   - 在 proposal 和 specs artifact 中应用“Capability ID 命名规则”；delta spec 目录必须与 proposal 中的 capability ID 完全一致。
   - 依赖 artifact 正文优先通过 instructions 返回的 `dependencies` 和 `outputPath` 定位；CLI 未提供正文时再读取对应文件。
   - 涉及已有主 spec 或 change delta 时，使用 CLI 使用策略中的 `openspec show` 命令获取结构化内容。
   - artifact 内容应服务于用户需求和 change 目标，避免把平台说明、内部流程、上下文块或规则块写成正文。
   - 生成或更新 design 时，按“决策记录”维护 `## Decisions` 和 `## Open Questions`。
   - 生成 tasks artifact 时，必须在实现任务前加入阻塞级审计任务，写清“审计未完成前不得执行任何实现任务”。
   - 阻塞级审计任务必须检查：proposal、design、specs 和 tasks 是否围绕开头核心句；capability ID 是否符合命名规则；当前 change 是否只包含未审核临时 artifacts；是否没有修改或影响现有其它文档；`## Open Questions` 是否没有未回答问题或已收敛歧义。
   - 如果某个 artifact 的关键决策无法从用户需求、依赖 artifact 或 instructions 中确定，直接向用户提一个具体问题；得到答案后继续生成。

5. 每个 artifact 写完后验证
   - 确认 `outputPath` 对应文件存在。
   - 重新运行：

     ```text
     openspec status --change "<name>" --json
     ```

   - 检查刚写入 artifact 的状态是否更新，并据此选择下一个可生成 artifact。
   - 若状态没有按预期变化，读取 instructions 和已写文件，修正缺失内容后再次检查。

6. 输出最终状态
   - 运行：

     ```text
     openspec status --change "<name>"
     ```

   - 然后运行：

     ```text
     openspec validate "<name>" --type change --json --strict --no-interactive
     ```

   - 汇报 change 是否已经达到可进入实现的状态。

## Artifact 写作要求

1. 严格使用 `template` 给出的结构，填充真实内容，不保留空模板说明。
2. 遵循 `instruction` 中的 schema 和写作规则。
3. 写作前读取已完成依赖 artifact，保持 proposal、design、tasks 等文件之间目标一致。
4. `context`、`rules`、`project_context` 等内容只用于约束判断，不作为 artifact 的段落、引用块或清单输出。
5. 内容应具体到可执行和可验收：proposal 写清 what 与 why，design 写清关键方案和取舍，tasks 写成可逐项完成的实现步骤。
6. 对不影响范围、协议、架构边界或验收标准的细节，选择与项目现有规范一致的默认；对会改变这些边界的缺口，先问用户。
7. 需要读取现有 specs 时，优先使用 CLI 使用策略中的主 spec 命令；CLI 输出不足时再读取 `openspec/specs/<spec>/spec.md`。
8. 除用户明确要求其它语言外，artifact 正文使用中文。
9. 每个 artifact 文件正文开头必须写一句核心句，说明本 change 的目标和当前文档性质，防止后续内容偏离范围。
10. 每个 artifact 必须标注：当前 change 只在 `openspec/changes/<name>/` 下形成未审核临时文档，不影响现有其它文档或主规范。
11. tasks artifact 必须把阻塞级审计任务放在所有实现任务之前；后续实现任务必须以该审计完成为前置条件。

## 完成标准

满足以下条件时才算完成：

1. change 目录存在，并包含 `.openspec.yaml`。
2. `openspec status --change "<name>" --json` 可读取。
3. `applyRequires` 中列出的每个 artifact 都已写入文件，且状态为 `done`。
4. 每个已生成 artifact 的文件路径来自对应 instructions 的 `outputPath`，并已验证存在。
5. artifact 正文没有复制 `context`、`rules` 或内部流程说明。
6. `openspec validate "<name>" --type change --json --strict --no-interactive` 通过；无法运行时说明失败原因和影响。
7. tasks artifact 包含阻塞级审计任务，并明确审计未完成前不可执行实现任务。
8. proposal Capabilities 和 specs 目录使用的 capability ID 符合命名规则，且已有能力修改使用现有主 spec ID。
9. 最终回复包含 change 名称、change 路径、已创建 artifacts、最终状态、审计门禁状态和下一步入口。

## 最终回复格式

完成后简要说明：

1. Change：`<name>`，位置 `openspec/changes/<name>/`。
2. Artifacts：列出创建或更新的 artifact 文件及作用。
3. 状态：说明是否已满足 apply-ready；如未满足，列出阻塞原因和需要用户补充的问题。
4. 审计门禁：说明阻塞级审计任务是否已完成；未完成时明确不能进入实现执行。
5. 下一步：审计未完成、存在未回答问题或已收敛歧义时先完成确认；确认完成后再进入实现流程。
