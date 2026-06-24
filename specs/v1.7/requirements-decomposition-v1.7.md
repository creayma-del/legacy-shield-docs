# legacy-shield v1.7 需求分解文档

> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应需求文档：requirements-v1.7.md
> 对应会议纪要：v1.7/meetings/requirements-alignment-v1.7-20260624.md

## 1. 功能分解

| 编号 | 名称 | 描述 | 对应需求 | 验收标准 | 优先级 |
|------|------|------|---------|---------|--------|
| T1 | 创建归档仓库 | 创建独立仓库 `legacy-shield-docs`，初始化 `package.json`（name: `@legacy-shield/docs`），初始化目录结构 | REQ-1-1 | 归档仓库存在，package.json name 正确，目录结构就绪 | P0 |
| T2 | 迁移历史 spec | 将主仓库 `docs/specs/` 下全部历史 spec（v1.1~v1.6、早期无版本号文档、common/）迁移到归档仓库对应目录 | REQ-1-2 | 迁移范围完整，主仓库对应目录已删除，归档仓库目录结构完整 | P0 |
| T3 | 配置 pnpm workspace | 主仓库 `pnpm-workspace.yaml` 增加 `../legacy-shield-docs`，`package.json` 增加 `@legacy-shield/docs` 依赖，`pnpm install` 成功 | REQ-1-3 | workspace 配置正确，pnpm install 成功，归档仓库可被引用 | P0 |
| T4 | 创建 INDEX.md | 创建 `docs/INDEX.md`，列出所有文档位置与状态（活文档在主仓库，历史 spec 在归档仓库） | REQ-1-5 | INDEX.md 存在，内容完整覆盖所有文档，位置与状态准确 | P0 |
| T5 | 评估 spec-guardian 路径引用 | 评估 spec-guardian skill 中引用 `docs/specs/` 路径的地方，迁移后路径变化是否需同步调整 | REQ-4-2 | 输出评估结论，若需调整则同步调整 | P1 |
| T6 | 引入 TypeDoc | 引入 TypeDoc 作为 devDependency，创建 `typedoc.json` 配置 | REQ-2-1 | package.json 含 typedoc，typedoc.json 配置正确，pnpm install 成功 | P0 |
| T7 | 迁移 api.md 为 TSDoc | 将 `docs/api.md` 的内容迁移为 `lib/` 下公开 API 的 TSDoc 注释 | REQ-2-2 | 源码公开 API 均有 TSDoc 注释，内容覆盖原 api.md 描述 | P0 |
| T8 | 配置 docs:gen 生成 | 配置 `docs:gen` npm script，TypeDoc 生成产物输出到 `docs/api/` 目录 | REQ-2-3, REQ-2-5 | pnpm docs:gen 执行成功，docs/api/ 下生成多文件 API 文档 | P0 |
| T9 | 改写 api.md 为入口索引 | 原 `docs/api.md` 改为入口索引，指向 `docs/api/` 目录 | REQ-2-4 | api.md 内容改为索引说明，指向 docs/api/ | P0 |
| T10 | 实现 docs-lint 脚本 | 实现独立 docs-lint 脚本，校验活文档中引用的代码符号、CLI 命令、配置项、文件路径是否真实存在 | REQ-3-1, REQ-3-2, REQ-3-3, REQ-3-4 | 脚本实现完整，校验四类引用，不一致时输出位置与原因，退出码非 0 | P0 |
| T11 | 配置 docs:lint script | 配置 `docs:lint` npm script | REQ-3-1 | package.json scripts 含 docs:lint，执行脚本 | P0 |
| T12 | 评估测试路径引用 | 评估测试中是否有引用 `docs/` 路径的断言，迁移后是否需同步调整 | REQ-4-3 | 输出评估结论，若需调整则同步调整 | P1 |
| T13 | 全量回归测试 | 执行 typecheck、build、test 全量回归，确保迁移不破坏现有功能 | REQ-4-1 | pnpm typecheck、pnpm build、pnpm test 全部通过 | P0 |

## 2. 任务依赖关系

| 任务 | 依赖 | 可并行任务 |
|------|------|-----------|
| T1 | 无 | T6（TypeDoc 引入与归档仓库创建无依赖） |
| T2 | T1 | T6、T7（T7 依赖 T6，但可与 T2 并行） |
| T3 | T2 | T6、T7、T10（T10 实现可与 workspace 配置并行） |
| T4 | T3（需确定归档文档路径） | T6、T7、T10 |
| T5 | T3（路径变化后评估） | T6、T7、T10、T12 |
| T6 | 无 | T1、T2、T3 |
| T7 | T6 | T1、T2、T3、T10 |
| T8 | T7 | T4、T5、T10、T11、T12 |
| T9 | T8（需生成 docs/api/ 后改写索引） | T4、T5、T10、T12 |
| T10 | 无 | T1~T9 |
| T11 | T10 | T4、T5、T9、T12 |
| T12 | T3（路径变化后评估） | T6、T7、T8、T9、T10、T11 |
| T13 | T4、T9、T11、T5、T12（所有变更完成） | 无 |

### 依赖关系图

```
T1 ──→ T2 ──→ T3 ──→ T4 ──→ T13
                │       │
                ├──→ T5 ─┘
                │
T6 ──→ T7 ──→ T8 ──→ T9 ──→ T13
                        │
T10 ──→ T11 ────────────┘
        │
T3 ──→ T12 ──→ T13
```

## 3. 资源分配估算

| 任务 | 工作量 | 角色 | 外部资源 |
|------|--------|------|---------|
| T1 | 小 | 开发 | 无 |
| T2 | 中 | 开发 | 无 |
| T3 | 小 | 开发 | 无 |
| T4 | 小 | 开发 | 无 |
| T5 | 中 | 开发 + 评审 | 无 |
| T6 | 小 | 开发 | TypeDoc 文档 |
| T7 | 大 | 开发 | 原 api.md 内容 |
| T8 | 小 | 开发 | TypeDoc 配置文档 |
| T9 | 小 | 开发 | 无 |
| T10 | 大 | 开发 | Babel AST、现有源码 |
| T11 | 小 | 开发 | 无 |
| T12 | 中 | 开发 + 评审 | 无 |
| T13 | 中 | 开发 + 测试 | 无 |

## 4. 时间线里程碑

| 里程碑 | 预计交付物 |
|--------|-----------|
| M1: 归档仓库就绪 | T1、T2、T3 完成，归档仓库创建并引入主仓库 |
| M2: API 文档代码化就绪 | T6、T7、T8、T9 完成，TypeDoc 生成 docs/api/ |
| M3: 契约校验就绪 | T10、T11 完成，docs-lint 可执行 |
| M4: 兼容性评估完成 | T5、T12 完成，路径引用评估与调整完成 |
| M5: 全量回归通过 | T13 完成，typecheck/build/test 全部通过 |
| M6: INDEX.md 完成 | T4 完成，文档总入口就绪 |

## 5. 风险与假设

| 风险 / 假设 | 说明 | 应对措施 |
|------------|------|---------|
| 假设：归档仓库为本地目录 `../legacy-shield-docs` | pnpm workspace 通过相对路径引入 | T1 创建时确认目录结构 |
| 风险：TypeDoc 生成的文档结构与原 api.md 差异较大 | 用户可能需要适应新文档结构 | T8 生成后与用户确认，必要时调整 TypeDoc 配置 |
| 风险：docs-lint 误报 | 正则提取代码符号可能不准确 | T10 优先用 AST 提取，设计阶段细化规则 |
| 风险：spec-guardian skill 引用 docs/specs/ 路径失效 | 迁移后路径变为 node_modules/@legacy-shield/docs/specs/ | T5 评估并同步调整 |
| 风险：测试中引用 docs/ 路径失效 | 迁移后测试断言可能失败 | T12 评估并同步调整 |
| 风险：T7 工作量大 | api.md 内容迁移为 TSDoc 需覆盖所有公开 API | 分批迁移，优先核心 API |
| 假设：v1.7 spec 文档保留在主仓库 | 本次 v1.7 的 spec 不迁移到归档 | docs/specs/v1.7/ 保留在主仓库 |

## 6. 任务并行策略

为提高效率，以下任务可并行执行：

- **并行组 A**（归档迁移线）：T1 → T2 → T3 → T4
- **并行组 B**（API 代码化线）：T6 → T7 → T8 → T9
- **并行组 C**（契约校验线）：T10 → T11

三条线在 T13（全量回归）处汇合。T5、T12 为评估类任务，可在 T3 完成后并行启动。
