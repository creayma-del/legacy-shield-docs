# legacy-shield v1.7 阶段 Spec

> 版本：v1.7
> 对应需求文档：[requirements-v1.7.md](../../requirements-v1.7.md)
> 对应需求分解文档：[requirements-decomposition-v1.7.md](../../requirements-decomposition-v1.7.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md)
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md)
> 状态：已通过
> 创建日期：2026-06-24

---

## 1. 阶段目标

v1.7 作为 MINOR 版本，目标为建立可持续的文档维护机制，解决文档数量增长导致的版本漂移与文档-代码不一致问题：

1. **分层归档**：将 v1.1~v1.6 历史 spec 迁移至独立归档仓库 `legacy-shield-docs`，通过 pnpm workspace 引入主仓库，主仓库轻量化。
2. **API 文档代码化**：将手写 `docs/api.md` 迁移为源码 TSDoc 注释，通过 TypeDoc 自动生成 `docs/api/` 目录，消除 API 文档与代码的版本漂移。
3. **契约校验**：实现 docs-lint 脚本，校验活文档中引用的代码符号、CLI 命令、配置项、文件路径是否与源码一致，不一致即报错。
4. **零破坏迁移**：不修改现有代码行为，不破坏现有测试。

---

## 2. 总体目标

| 目标编号 | 描述 | 对应需求 | 验收标准 |
|---|---|---|---|
| G-1 | 归档仓库创建与历史 spec 迁移 | REQ-1-1, REQ-1-2 | 归档仓库存在，历史 spec 完整迁移，主仓库 docs/specs/ 仅保留 v1.7 |
| G-2 | pnpm workspace 引入 | REQ-1-3 | workspace 配置正确，pnpm install 成功，归档仓库可被引用 |
| G-3 | 主仓库 docs/ 重组 | REQ-1-4, REQ-1-5 | docs/ 仅含活文档 + specs/v1.7/ + 根级 v1.7 spec；INDEX.md 存在且内容完整 |
| G-4 | TypeDoc 引入与 API 文档生成 | REQ-2-1, REQ-2-3, REQ-2-5 | typedoc ^0.28.18 引入，pnpm docs:gen 生成 docs/api/ |
| G-5 | TSDoc 迁移 | REQ-2-2 | 源码公开 API 均有 TSDoc 注释，覆盖设计文档 §2.2.1 全部公开 API |
| G-6 | api.md 改写为入口索引 | REQ-2-4 | api.md 内容改为索引说明，指向 docs/api/ |
| G-7 | docs-lint 契约校验 | REQ-3-1, REQ-3-2, REQ-3-3, REQ-3-4 | 脚本实现完整，校验四类引用，不一致时输出位置与原因，退出码非 0 |
| G-8 | 兼容性评估 | REQ-4-2, REQ-4-3 | spec-guardian skill 路径评估完成，测试路径评估完成 |
| G-9 | 全量回归 | REQ-4-1 | pnpm typecheck、pnpm build、pnpm test 全部通过 |

---

## 3. 任务清单

| 任务编号 | 任务名称 | 优先级 | 依赖 | 对应需求 |
|---|---|---|---|---|
| T1 | 创建归档仓库 | P0 | 无 | REQ-1-1 |
| T2 | 迁移历史 spec | P0 | T1 | REQ-1-2 |
| T3 | 配置 pnpm workspace | P0 | T2 | REQ-1-3 |
| T4 | 创建 INDEX.md | P0 | T3 | REQ-1-5 |
| T5 | 评估 spec-guardian 路径引用 | P1 | T3 | REQ-4-2 |
| T6 | 引入 TypeDoc | P0 | 无 | REQ-2-1 |
| T7 | 迁移 api.md 为 TSDoc | P0 | T6 | REQ-2-2 |
| T8 | 配置 docs:gen 生成 | P0 | T7 | REQ-2-3, REQ-2-5 |
| T9 | 改写 api.md 为入口索引 | P0 | T8 | REQ-2-4 |
| T10 | 实现 docs-lint 脚本 | P0 | 无 | REQ-3-1, REQ-3-2, REQ-3-3, REQ-3-4 |
| T11 | 配置 docs:lint script | P0 | T10 | REQ-3-1 |
| T12 | 评估测试路径引用 | P1 | T3 | REQ-4-3 |
| T13 | 全量回归测试 | P0 | T4, T5, T9, T11, T12 | REQ-4-1 |

### 并行执行批次

| 批次 | 任务 | 说明 |
|---|---|---|
| 并行组 A | T1 → T2 → T3 → T4 | 归档迁移线 |
| 并行组 B | T6 → T7 → T8 → T9 | API 代码化线 |
| 并行组 C | T10 → T11 | 契约校验线 |
| 评估组 | T5, T12 | T3 完成后并行启动 |
| 汇合 | T13 | 依赖 T4, T5, T9, T11, T12 |

---

## 4. 验收标准

### 4.1 功能验收

| 编号 | 验收项 | 验证方式 |
|---|---|---|
| AC-1 | 归档仓库 `legacy-shield-docs` 存在，package.json name 为 `@legacy-shield/docs` | T1 验收 |
| AC-2 | 历史 spec 完整迁移，主仓库 docs/specs/ 仅保留 v1.7 相关文档 | T2 验收 + T13 目录结构校验 |
| AC-3 | pnpm workspace 配置正确，pnpm install 成功 | T3 验收 |
| AC-4 | INDEX.md 存在，内容完整覆盖所有文档，含路径映射表与 onboarding 说明 | T4 验收 |
| AC-5 | TypeDoc ^0.28.18 引入，typedoc.json 配置正确 | T6 验收 |
| AC-6 | 源码公开 API 均有 TSDoc 注释（覆盖 §2.2.1 全部公开 API） | T7 验收 |
| AC-7 | pnpm docs:gen 执行成功，docs/api/ 下生成多文件 API 文档 | T8 验收 |
| AC-8 | api.md 改为入口索引，指向 docs/api/ | T9 验收 |
| AC-9a | docs-lint 校验范围仅限 docs/INDEX.md、docs/usage.md、docs/custom-rules.md、docs/api.md 四个活文档 | T10 验收 |
| AC-9b | docs-lint 校验四类引用（代码符号、CLI 命令、配置项、文件路径），不一致时输出文件路径、行号、原因 | T10 验收 |
| AC-9c | docs-lint 发现不一致时退出码非 0（退出码 1） | T10 验收 |
| AC-10 | pnpm docs:lint 可执行，退出码正确 | T11 验收 |
| AC-11 | spec-guardian skill 路径评估完成，结论文档输出 | T5 验收 |
| AC-12 | 测试路径评估完成，结论文档输出 | T12 验收 |

### 4.2 工程验收

| 编号 | 验收项 | 验证方式 |
|---|---|---|
| AC-13 | `pnpm typecheck` 通过 | 类型检查零错误 |
| AC-14 | `pnpm build` 通过 | 构建零错误 |
| AC-15 | `pnpm test` 全量通过 | 测试零失败 |
| AC-16 | `pnpm docs:gen` 生成成功 | docs/api/ 产物存在 |
| AC-17 | `pnpm docs:lint` 校验通过 | 退出码 0 |
| AC-18 | `docs/` 目录结构符合预期 | docs/ 仅含 INDEX.md、api.md、api/、usage.md、custom-rules.md、specs/；docs/specs/ 仅含 v1.7/ 及根级 v1.7 spec 文件 |

### 4.3 整体测试计划

| 测试阶段 | 测试内容 | 执行命令 | 覆盖任务 |
|---|---|---|---|
| 类型检查 | TypeScript 编译零错误 | `pnpm typecheck` | T7（TSDoc 注释）、T10（docs-lint 脚本） |
| 构建 | 构建产物零错误 | `pnpm build` | T10（docs-lint 脚本编译） |
| 单元测试 | 现有测试全量通过 | `pnpm test` | T13（回归验证） |
| 文档生成 | TypeDoc 生成 docs/api/ | `pnpm docs:gen` | T8 |
| 契约校验 | docs-lint 校验活文档引用一致性 | `pnpm docs:lint` | T10、T11 |
| 目录结构校验 | docs/ 目录结构符合预期 | 人工检查 | T2、T13 |

---

## 5. 变更范围

### 5.1 涉及文件

| 文件 | 变更类型 | 任务 |
|---|---|---|
| `../legacy-shield-docs/package.json` | 新建 | T1 |
| `../legacy-shield-docs/README.md` | 新建 | T1 |
| `../legacy-shield-docs/specs/` | 新建（空目录） | T1 |
| `../legacy-shield-docs/specs/v1.1/`~`v1.6/` | 新建（迁移内容） | T2 |
| `../legacy-shield-docs/specs/common/` | 新建（迁移内容） | T2 |
| `../legacy-shield-docs/specs/phases/` | 新建（迁移内容） | T2 |
| `../legacy-shield-docs/specs/design.md` 等 | 新建（迁移内容） | T2 |
| `docs/specs/v1.1/`~`v1.6/`、`common/`、`phases/`、根目录早期文档 | 删除（迁移后） | T2 |
| `pnpm-workspace.yaml` | 新建 | T3 |
| `package.json` | 修改（devDependencies + scripts） | T3, T6, T8, T11 |
| `docs/INDEX.md` | 新建 | T4 |
| `typedoc.json` | 新建 | T6 |
| `lib/api.ts` | 修改（加 TSDoc） | T7 |
| `lib/code-quality/index.ts` | 修改（加 TSDoc） | T7 |
| `lib/knowledge-graph/index.ts` | 修改（加 TSDoc） | T7 |
| `lib/custom-rules/index.ts` | 修改（加 TSDoc） | T7 |
| `lib/types.ts` | 修改（加 TSDoc） | T7 |
| `docs/api.md` | 改写（改为入口索引） | T9 |
| `docs/api/` | 新建（TypeDoc 生成产物） | T8 |
| `scripts/docs-lint.ts` | 新建 | T10 |
| `tsconfig.json` | 修改（include 增加 scripts/**/*.ts） | T10 |
| `tsconfig.test.json` | 修改（include 增加 scripts/**/*.ts） | T10 |
| `docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md` | 新建 | T5 |
| `docs/specs/v1.7/evaluations/t12-test-path-evaluation.md` | 新建 | T12 |

### 5.2 不在范围内

- 不修改历史 spec 文档内容（只迁移位置）
- 不对 usage.md / custom-rules.md 做代码化（仅 api.md 代码化）
- 不修改现有 CLI 命令、配置项、公开 API 的行为
- 不调整 spec-guardian 的流程规则本身（仅评估路径引用是否需调整）
- INDEX.md 文档列表与文件系统的一致性校验（P3 优化，本期不实现）
- docs-lint 对 docs/specs/ spec 文档的校验（不强制校验）
- docs-lint 对归档仓库文档的校验（已冻结，不再变化）
- CI/CD 集成（项目当前无 CI/CD 配置）
- Vue 2 支持
- MCP 文档检索方式

---

## 6. 依赖与约束

### 6.1 外部依赖

| 依赖 | 版本 | 用途 | 变更 |
|---|---|---|---|
| typedoc | ^0.28.18 | 从 TSDoc 生成 API 文档 | 新增 devDependency |

### 6.2 复用依赖

| 依赖 | 用途 |
|---|---|
| @babel/parser | docs-lint 解析源码 AST（plugins: ['typescript']） |
| @babel/traverse | docs-lint 遍历 AST 收集符号 |
| @babel/types | docs-lint AST 节点类型判断 |

### 6.3 约束

- Node.js >=20.19.0（项目现有要求）
- TypeScript 5.9.3（TypeDoc ^0.28.18 兼容）
- pnpm 10.33.4（workspace 支持）
- 不修改现有 CLI 命令行为
- 不修改现有测试断言
- 不引入除 typedoc 外的新依赖
- 归档仓库位于主仓库同级目录 `../legacy-shield-docs`

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，发现 3 个 P1 + 4 个 P2 问题
> - 第二轮评审（2026-06-24）：通过，P1 全部闭环，P2 全部修复，遗留 2 个 P2 优化项 + 2 个 P3 建议项不影响可行性
> 评审人：spec-reviewer-expert
