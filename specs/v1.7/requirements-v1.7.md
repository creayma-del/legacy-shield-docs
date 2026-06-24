# legacy-shield v1.7 需求文档

> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24

## 1. 背景

随着项目演进，docs 目录已累积 100+ 个 markdown 文件（v1.1~v1.6 的 spec 文档为主），带来以下问题：

1. **体量膨胀**：spec 文档按版本累积，主仓库 git 历史膨胀，未来 v1.7+ 会继续增长。
2. **版本对齐风险**：spec 文档冻结后不再修改，但代码持续演进，容易出现 spec 描述与实际实现脱节。
3. **AI 读取效率**：文档量大时，AI 每次需扫描大量文件定位有效信息。
4. **主仓库膨胀**：文档占据主仓库大量空间，干扰日常开发。

本次 v1.7 旨在通过「分层归档 + API 文档代码化 + 契约校验」三套组合方案，建立可持续的文档维护机制，保证文档与代码强对齐，AI 读取高效。

## 2. 需求条目

### 模块 1：分层归档（方案 A）

| 编号 | 需求描述 | 验收标准 |
|------|---------|---------|
| REQ-1-1 | 创建独立归档仓库 `legacy-shield-docs`，`package.json` name 为 `@legacy-shield/docs` | 归档仓库存在，含 `package.json`，name 字段正确 |
| REQ-1-2 | 将主仓库 `docs/specs/` 下全部历史 spec 迁移到归档仓库 | 迁移范围见下方「迁移范围清单」，主仓库对应目录已删除 |
| REQ-1-3 | 主仓库通过 pnpm workspace 引入 `@legacy-shield/docs`，workspace 路径为 `../legacy-shield-docs` | `pnpm-workspace.yaml` 含该路径，`package.json` 含该依赖，`pnpm install` 成功 |
| REQ-1-4 | 主仓库 `docs/` 只保留活文档 | 迁移后主仓库 `docs/` 只含：`INDEX.md`、`usage.md`、`custom-rules.md`、`api/`（TypeDoc 生成）、`api.md`（入口索引） |
| REQ-1-5 | 主仓库维护 `docs/INDEX.md` 作为文档总入口 | INDEX.md 列出所有文档位置与状态，活文档在主仓库，历史 spec 在归档仓库 |

#### 迁移范围清单（REQ-1-2 细化）

迁移到归档仓库的内容：

- `docs/specs/v1.1/` 全部（含 phases/、acceptance-report、design、execution-plan、requirements）
- `docs/specs/v1.2/` 全部（含 meetings/、phases/、acceptance-report、design、execution-plan、requirements-decomposition、requirements）
- `docs/specs/v1.3/` 全部（含 meetings/、phases/、acceptance-report、design、execution-plan、requirements-decomposition、requirements）
- `docs/specs/v1.4/` 全部（含 meetings/、phases/、acceptance-report、design、execution-plan、requirements-decomposition、requirements）
- `docs/specs/v1.5/` 全部（含 meetings/、phases/、acceptance-report、design、execution-plan、requirements-decomposition）
- `docs/specs/v1.6/` 全部（含 meetings/、phases/、design、execution-plan、requirements-decomposition、requirements）
- `docs/specs/phases/` 下无版本号的早期文档（phase-1-spec.md ~ phase-5-spec.md）
- `docs/specs/` 根目录下无版本号的早期文档（design.md、execution-plan.md、requirements.md、acceptance-report.md）
- `docs/specs/common/` 下两个文档（project-rules.md、typescript-migration.md）

保留在主仓库的内容：

- `docs/usage.md`（活文档）
- `docs/custom-rules.md`（活文档）
- `docs/api.md`（改为入口索引，指向 docs/api/）
- `docs/api/`（TypeDoc 生成产物，REQ-2-3）
- `docs/INDEX.md`（新建，REQ-1-5）
- `docs/specs/v1.7/`（本次 v1.7 的 spec 文档）

### 模块 2：API 文档代码化（方案 C）

| 编号 | 需求描述 | 验收标准 |
|------|---------|---------|
| REQ-2-1 | 引入 TypeDoc 作为 devDependency | `package.json` devDependencies 含 typedoc，`pnpm install` 成功 |
| REQ-2-2 | 将 `docs/api.md` 的内容迁移为源码 TSDoc 注释（覆盖 `lib/` 下的公开 API） | 源码中公开 API 均有 TSDoc 注释，注释内容覆盖原 api.md 的描述 |
| REQ-2-3 | 配置 TypeDoc，生成产物输出到 `docs/api/` 目录（多文件，按模块拆分） | `pnpm docs:gen` 执行成功，`docs/api/` 下生成多文件 API 文档 |
| REQ-2-4 | 原 `docs/api.md` 改为入口索引，指向 `docs/api/` 目录 | api.md 内容改为索引说明，指向 docs/api/ |
| REQ-2-5 | 新增 npm script `docs:gen` 用于生成 API 文档 | `package.json` scripts 含 `docs:gen`，执行生成 docs/api/ |

### 模块 3：契约校验（方案 E）

| 编号 | 需求描述 | 验收标准 |
|------|---------|---------|
| REQ-3-1 | 新增独立 docs-lint 脚本，作为独立 npm script `docs:lint` | `package.json` scripts 含 `docs:lint`，执行脚本 |
| REQ-3-2 | docs-lint 校验范围：仅主仓库活文档 | 校验 `docs/INDEX.md`、`docs/usage.md`、`docs/custom-rules.md`、`docs/api.md` |
| REQ-3-3 | docs-lint 校验规则：文档中引用的代码符号（函数名、类名、类型名）、CLI 命令、配置项、文件路径是否在源码中真实存在 | 校验逻辑覆盖上述四类引用 |
| REQ-3-4 | docs-lint 发现不一致时输出具体位置与原因，退出码非 0 | 不一致时输出文件路径、行号、原因，退出码 1 |

### 模块 4：流程与兼容性

| 编号 | 需求描述 | 验收标准 |
|------|---------|---------|
| REQ-4-1 | 迁移过程不破坏现有构建、测试、typecheck | `pnpm typecheck`、`pnpm build`、`pnpm test` 全部通过 |
| REQ-4-2 | 评估 spec-guardian skill 中引用 `docs/specs/` 路径的地方，迁移后路径变化是否需要同步调整 | 输出评估结论，若需调整则同步调整 |
| REQ-4-3 | 评估测试中是否有引用 `docs/` 路径的断言，迁移后是否需要同步调整 | 输出评估结论，若需调整则同步调整 |

## 3. 验收标准（整体）

1. 归档仓库 `legacy-shield-docs` 创建完成，含全部历史 spec 文档。
2. 主仓库通过 pnpm workspace 引入归档仓库，`pnpm install` 成功。
3. 主仓库 `docs/` 只保留活文档 + INDEX.md + api/ + v1.7 spec。
4. `docs/INDEX.md` 存在且列出所有文档位置与状态。
5. 源码公开 API 有 TSDoc 注释，`pnpm docs:gen` 生成 `docs/api/` 目录。
6. `docs/api.md` 改为入口索引。
7. `pnpm docs:lint` 执行成功，无不一致项（或已修正）。
8. `pnpm typecheck`、`pnpm build`、`pnpm test` 全部通过。
9. spec-guardian skill 路径引用评估完成，必要时已同步调整。
10. 测试中 docs/ 路径引用评估完成，必要时已同步调整。

## 4. 范围与非范围

### 范围内

- 创建归档仓库并迁移历史 spec
- pnpm workspace 引入归档仓库
- 主仓库 docs/ 结构重组（保留活文档 + INDEX.md）
- TypeDoc 引入与 api.md 代码化
- docs-lint 独立脚本

### 范围外

- 不修改历史 spec 文档内容（只迁移位置）
- 不引入 MCP 检索层（方案 D，未来考虑）
- 不对 usage.md / custom-rules.md 做代码化（仅 api.md 代码化）
- 不修改现有 CLI 命令、配置项、公开 API 的行为
- 不调整 spec-guardian 的流程规则本身（仅评估路径引用是否需调整）

## 5. 风险与假设

| 风险 / 假设 | 说明 | 应对措施 |
|------------|------|---------|
| 假设：归档仓库为本地目录 `../legacy-shield-docs` | pnpm workspace 通过相对路径引入 | 迁移时确认目录结构与 workspace 配置 |
| 风险：TypeDoc 生成的文档结构与原 api.md 差异较大 | 用户可能需要适应新文档结构 | 生成后与用户确认，必要时调整 TypeDoc 配置 |
| 风险：docs-lint 误报 | 正则提取代码符号可能不准确 | 设计阶段细化提取规则，优先用 AST |
| 风险：spec-guardian skill 引用 docs/specs/ 路径失效 | 迁移后路径变为 node_modules/@legacy-shield/docs/specs/ | REQ-4-2 评估并同步调整 |
