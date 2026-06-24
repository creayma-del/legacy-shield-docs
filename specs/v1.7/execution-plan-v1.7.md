# legacy-shield v1.7 执行计划

> 版本：v1.7
> 对应设计文档：[design-v1.7.md](design-v1.7.md)
> 对应需求分解文档：[requirements-decomposition-v1.7.md](requirements-decomposition-v1.7.md)
> 状态：已通过

---

## 1. 任务清单

| 任务编号 | 任务名称 | 优先级 | 依赖 | 预计耗时 | 负责人 |
|---|---|---|---|---|---|
| T1 | 创建归档仓库 | P0 | 无 | 小 | 开发专家 |
| T2 | 迁移历史 spec | P0 | T1 | 中 | 开发专家 |
| T3 | 配置 pnpm workspace | P0 | T2 | 小 | 开发专家 |
| T4 | 创建 INDEX.md | P0 | T3 | 小 | 开发专家 |
| T5 | 评估 spec-guardian 路径引用 | P1 | T3 | 中 | 开发专家 + 评审 |
| T6 | 引入 TypeDoc | P0 | 无 | 小 | 开发专家 |
| T7 | 迁移 api.md 为 TSDoc | P0 | T6 | 大 | 开发专家 |
| T8 | 配置 docs:gen 生成 | P0 | T7 | 小 | 开发专家 |
| T9 | 改写 api.md 为入口索引 | P0 | T8 | 小 | 开发专家 |
| T10 | 实现 docs-lint 脚本 | P0 | 无 | 大 | 开发专家 |
| T11 | 配置 docs:lint script | P0 | T10 | 小 | 开发专家 |
| T12 | 评估测试路径引用 | P1 | T3 | 中 | 开发专家 + 评审 |
| T13 | 全量回归测试 | P0 | T4, T5, T9, T11, T12 | 中 | 开发专家 + 测试 |

### 任务详细说明

#### T1：创建归档仓库

- **目标**：创建独立仓库 `legacy-shield-docs`（位于主仓库同级目录 `../legacy-shield-docs`），初始化 `package.json`（name: `@legacy-shield/docs`，version: `1.0.0`，private: true），创建 `specs/` 空目录与 `README.md`
- **涉及文件**：`../legacy-shield-docs/package.json`（新建）、`../legacy-shield-docs/README.md`（新建）、`../legacy-shield-docs/specs/`（新建空目录）
- **关键实现**：按设计文档 §2.1.1 创建归档仓库；仅创建 `specs/` 空目录（不创建 common/、phases/、v1.1/~v1.6/ 等子目录，子目录由 T2 迁移时自动创建，避免 `cp -r` 产生嵌套目录）；README.md 说明归档仓库用途与引用方式
- **验收标准**：归档仓库存在，package.json name 为 `@legacy-shield/docs`，specs/ 空目录就绪

#### T2：迁移历史 spec

- **目标**：将主仓库 `docs/specs/` 下全部历史 spec 迁移到归档仓库对应目录
- **涉及文件**：`docs/specs/common/`、`docs/specs/phases/`、`docs/specs/v1.1/`~`docs/specs/v1.6/`、`docs/specs/design.md`、`docs/specs/requirements.md`、`docs/specs/execution-plan.md`、`docs/specs/acceptance-report.md`（共 4 个根目录早期文档）
- **关键实现**：使用 `cp + git rm` 方式迁移（`git mv` 无法跨仓库移动文件）。迁移方法以本执行计划为准，设计文档 §2.1.1 已同步修正为 `cp + git rm`。迁移范围按设计文档 §2.1.1 与需求对齐会议确认的清单执行，执行前输出完整文件清单并经评审确认。迁移步骤：
  ```bash
  # 1. 复制文件到归档仓库
  cp -r docs/specs/v1.1 ../legacy-shield-docs/specs/v1.1
  # 2. 在归档仓库提交（commit message 记录主仓库原始 commit hash 以便追溯）
  cd ../legacy-shield-docs && git add . && git commit -m "migrate v1.1 specs from main repo (original commit: <hash>)"
  # 3. 在主仓库删除并提交
  cd ../legacy-shield && git rm -r docs/specs/v1.1 && git commit -m "migrate v1.1 specs to archive repo"
  ```
  对每个版本目录（v1.1~v1.6）、common/、phases/、根目录早期文档重复上述步骤。迁移后主仓库 `docs/specs/` 仅保留 `v1.7/`、`design-v1.7.md`、`requirements-v1.7.md`、`requirements-decomposition-v1.7.md`、`execution-plan-v1.7.md` 等当前版本文档
- **验收标准**：迁移范围完整，主仓库对应目录已删除，归档仓库目录结构完整，文件数与迁移前一致

#### T3：配置 pnpm workspace

- **目标**：主仓库新增 `pnpm-workspace.yaml` 引入归档仓库，`package.json` 增加 `@legacy-shield/docs` 依赖，`pnpm install` 成功
- **涉及文件**：`pnpm-workspace.yaml`（新建）、`package.json`（修改 devDependencies）
- **关键实现**：`pnpm-workspace.yaml` 内容为 `packages: ['../legacy-shield-docs']`；`package.json` devDependencies 增加 `"@legacy-shield/docs": "workspace:*"`；执行 `pnpm install` 验证
- **验收标准**：workspace 配置正确，pnpm install 成功，`node_modules/@legacy-shield/docs/specs/` 可访问

#### T4：创建 INDEX.md

- **目标**：创建 `docs/INDEX.md` 文档总入口，列出所有文档位置与状态
- **涉及文件**：`docs/INDEX.md`（新建）
- **关键实现**：按设计文档 §2.1.4 结构创建，包含活文档表、当前版本 Spec 表、历史归档表、历史文档路径映射表；包含 onboarding 说明：克隆主仓库后需额外克隆归档仓库至 `../legacy-shield-docs`，否则 `pnpm install` 失败
- **验收标准**：INDEX.md 存在，内容完整覆盖所有文档，位置与状态准确，路径映射表完整

#### T5：评估 spec-guardian 路径引用

- **目标**：评估 spec-guardian skill 中引用 `docs/specs/` 路径的地方，迁移后路径变化是否需同步调整
- **涉及文件**：`.trae/skills/spec-guardian/` 下所有 skill 文件
- **关键实现**：按设计文档 §8.1 评估结论，skill 中的 `docs/specs/` 路径引用是命名规范（新文档创建），v1.7+ spec 仍在主仓库，无需调整；历史文档读取通过 INDEX.md 路径映射表定位
- **验收标准**：输出评估结论文档至 `docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md`，结论包含是否需调整 skill 流程规则及具体调整方案（若需）

#### T6：引入 TypeDoc

- **目标**：引入 TypeDoc `^0.28.18` 作为 devDependency，创建 `typedoc.json` 配置
- **涉及文件**：`package.json`（修改 devDependencies）、`typedoc.json`（新建）
- **关键实现**：按设计文档 §2.2.2 与 §5.1，typedoc 版本 `^0.28.18`（兼容 TypeScript 5.9.3）；`typedoc.json` entryPoints 为 lib/api.ts、lib/code-quality/index.ts、lib/knowledge-graph/index.ts、lib/custom-rules/index.ts、lib/types.ts（不含 cli.ts）
- **验收标准**：package.json 含 typedoc ^0.28.18，typedoc.json 配置正确，pnpm install 成功

#### T7：迁移 api.md 为 TSDoc

- **目标**：将 `docs/api.md` 的内容迁移为 `lib/` 下公开 API 的 TSDoc 注释
- **涉及文件**：`lib/api.ts`、`lib/code-quality/index.ts`、`lib/knowledge-graph/index.ts`、`lib/custom-rules/index.ts`、`lib/types.ts`
- **关键实现**：按设计文档 §2.2.5，将原 api.md 中的端点说明、参数说明、响应示例迁移为源码 TSDoc；`lib/api.ts` 的 `startApiServer` 加 TSDoc 描述所有端点；`lib/code-quality/index.ts` 的 `runAll`/`runModule`/`runDiff`/`runWatch`/`loadLocalLLMConfig`/`createCLI` 加 TSDoc；`lib/knowledge-graph/index.ts` 的 `runKnowledgeGraph` 补全 TSDoc；`lib/custom-rules/index.ts` 的 `runCustomRules`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` 加 TSDoc；`lib/types.ts` 的公开类型加 TSDoc
- **验收标准**：源码公开 API 均有 TSDoc 注释（覆盖设计文档 §2.2.1 全部公开 API），内容覆盖原 api.md 描述

#### T8：配置 docs:gen 生成

- **目标**：配置 `docs:gen` npm script，TypeDoc 生成产物输出到 `docs/api/` 目录
- **涉及文件**：`package.json`（修改 scripts）
- **关键实现**：scripts 增加 `"docs:gen": "typedoc"`；执行 `pnpm docs:gen` 验证生成产物
- **验收标准**：pnpm docs:gen 执行成功，docs/api/ 下生成多文件 API 文档

#### T9：改写 api.md 为入口索引

- **目标**：原 `docs/api.md` 改为入口索引，指向 `docs/api/` 目录
- **涉及文件**：`docs/api.md`（改写）
- **关键实现**：按设计文档 §2.2.4，api.md 改为索引说明，包含在线文档链接、更新方式、源码位置
- **验收标准**：api.md 内容改为索引说明，指向 docs/api/

#### T10：实现 docs-lint 脚本

- **目标**：实现独立 docs-lint 脚本，校验活文档中引用的代码符号、CLI 命令、配置项、文件路径是否真实存在
- **涉及文件**：`scripts/docs-lint.ts`（新建）、`tsconfig.json`（修改 include 增加 `scripts/**/*.ts`）、`tsconfig.test.json`（同步增加 `scripts/**/*.ts`）
- **关键实现**：按设计文档 §2.3 与 §4.1 实现；使用 Babel AST（`@babel/parser` plugins: ['typescript']、`@babel/traverse`、`@babel/types`）收集 lib/ 下导出符号 + cli.ts 中子命令与选项（含 `.option()` 与 `.requiredOption()`）；白名单匹配策略校验四类引用；CLI 入口使用 `fileURLToPath(import.meta.url) === resolve(process.argv[1])` 判定；`--no-` 前缀选项提取去掉 no- 的实际选项名
- **验收标准**：脚本实现完整，校验四类引用，不一致时输出位置与原因，退出码非 0；tsconfig.json include 含 `scripts/**/*.ts`

#### T11：配置 docs:lint script

- **目标**：配置 `docs:lint` npm script
- **涉及文件**：`package.json`（修改 scripts）
- **关键实现**：scripts 增加 `"docs:lint": "node ./dist/scripts/docs-lint.js"`；先执行 `pnpm build` 编译，再执行 `pnpm docs:lint` 验证
- **验收标准**：package.json scripts 含 docs:lint，执行脚本输出校验结果

#### T12：评估测试路径引用

- **目标**：评估测试中是否有引用 `docs/` 路径的断言，迁移后是否需同步调整
- **涉及文件**：`tests/` 下所有文件
- **关键实现**：按设计文档 §8.2 评估结论，测试中无引用 `docs/` 文档路径的断言（仅 vendor 第三方库 JS 文件匹配），无需调整
- **验收标准**：输出评估结论文档至 `docs/specs/v1.7/evaluations/t12-test-path-evaluation.md`，结论包含是否需调整测试及具体调整方案（若需）

#### T13：全量回归测试

- **目标**：执行 typecheck、build、test 全量回归，确保迁移不破坏现有功能
- **涉及文件**：全项目
- **关键实现**：执行 `pnpm typecheck`、`pnpm build`、`pnpm test`；验证 docs:gen 与 docs:lint 可正常执行；验证 pnpm install 含 workspace 依赖
- **验收标准**：pnpm typecheck、pnpm build、pnpm test 全部通过；pnpm docs:gen 生成成功；pnpm docs:lint 校验通过；`docs/` 仅含 INDEX.md、api.md、api/、usage.md、custom-rules.md、specs/；`docs/specs/` 仅含 v1.7/ 目录及 v1.7 根级 spec 文件（design-v1.7.md、requirements-v1.7.md、requirements-decomposition-v1.7.md、execution-plan-v1.7.md）

---

## 2. 里程碑

| 里程碑 | 预计交付物 |
|---|---|
| M1: 归档仓库就绪 | T1、T2、T3 完成，归档仓库创建并引入主仓库 |
| M6: INDEX.md 完成 | T4 完成，文档总入口就绪（含 onboarding 说明） |
| M2: API 文档代码化就绪 | T6、T7、T8、T9 完成，TypeDoc 生成 docs/api/ |
| M3: 契约校验就绪 | T10、T11 完成，docs-lint 可执行 |
| M4: 兼容性评估完成 | T5、T12 完成，路径引用评估完成 |
| M5: 全量回归通过 | T13 完成，typecheck/build/test 全部通过 |

### 并行执行策略

三条并行线在 T13（全量回归）处汇合：

- **并行组 A**（归档迁移线）：T1 → T2 → T3 → T4
- **并行组 B**（API 代码化线）：T6 → T7 → T8 → T9
- **并行组 C**（契约校验线）：T10 → T11

T5、T12 为评估类任务，可在 T3 完成后并行启动。

---

## 3. 资源与角色

- **开发专家**：负责 T1-T12 全部代码开发与配置变更
- **质量管控专家**：负责代码评审与功能测试（T13 回归测试阶段并行调用）
- **SOLO Coder**：负责 Spec 文档创建、流程编排、验收归档

---

## 4. 风险与应对

| 风险 | 影响 | 应对措施 |
|---|---|---|
| 跨仓库迁移文件丢失或损坏 | T2 历史 spec 内容不完整 | 迁移前 git 备份；使用 `cp + git rm` 方式迁移（`git mv` 无法跨仓库）；迁移后校验文件数与内容完整性（diff 对比） |
| pnpm workspace 与现有依赖冲突 | T3 pnpm install 失败 | T3 验证依赖解析；失败时移除 workspace 配置回滚 |
| TypeDoc 生成失败 | T8 docs:gen 报错 | T6 验证 typedoc.json 配置；T8 生成后检查产物完整性 |
| TSDoc 迁移遗漏 | T7 API 文档不完整 | T7 分批迁移，优先核心 API（startApiServer、runAll、runKnowledgeGraph）；T8 生成后对照原 api.md 检查 |
| docs-lint 误报 | T10 校验失败 | 白名单匹配策略 + TypeScript 关键字 denylist + CLI 命令白名单 + 配置项有效 CLI 上下文校验（设计文档 §2.3.5） |
| tsconfig.json 变更影响现有编译 | T10 编译失败 | 仅新增 `scripts/**/*.ts` 到 include，不修改现有配置；T13 回归验证 |
| spec-guardian skill 路径失效 | T5 历史文档无法定位 | INDEX.md 路径映射表提供迁移前后路径对应；skill 命名规范不变 |
| 归档仓库路径错误 | T3 pnpm install 失败 | T1 创建时确认目录结构；T3 验证 workspace 配置 |

> **整体回滚策略**：详见设计文档 §7，包含 5 步回滚步骤：1. 删除 pnpm-workspace.yaml，移除 @legacy-shield/docs 依赖；2. 从 git 历史恢复 docs/specs/ 下的历史 spec；3. 移除 typedoc 依赖与 docs:gen script，恢复手写 api.md；4. 移除 scripts/docs-lint.ts 与 docs:lint script；5. 删除 docs/api/、docs/INDEX.md。

---

## 5. 任务 Spec 拆解说明

### 5.1 阶段 Spec 创建与评审

执行计划评审通过后，必须先创建阶段 Spec 并通过评审，才能拆解任务 Spec：

1. 创建阶段 Spec：`docs/specs/v1.7/phases/phase-v1.7-spec.md`，汇总本阶段全部任务清单、验收标准、整体测试计划。
2. 阶段 Spec 提交评审专家评审，状态从「草稿」→「评审中」→「已通过」。
3. 阶段 Spec 评审通过后，方可拆解任务 Spec。

### 5.2 任务 Spec 拆解与评审

- 每个任务拆解为独立的任务 Spec。
- 任务 Spec 文件命名：`docs/specs/v1.7/phases/phase-v1.7-t{n}-spec.md`（与 v1.1~v1.6 既有惯例一致，位于 `v1.7/phases/` 子目录下）。
- 任务 Spec 引用阶段 Spec、设计文档、执行计划、任务编号和依赖关系。
- 任务 Spec 文件头包含任务编号、对应阶段 Spec、依赖任务等字段，正文包含任务目标、实现步骤、测试计划、验收标准等章节。
- 所有任务 Spec 评审通过后，方可进入代码开发阶段。
