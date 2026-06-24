# 迁移 api.md 为 TSDoc

> 任务编号：T7
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.2.1、§2.2.5
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T7
> 依赖任务：T6
> 评审记录：见本文档末尾

---

## 1. 任务目标

将 `docs/api.md` 中的端点说明、参数说明、响应示例等内容迁移为 `lib/` 下公开 API 的源码 TSDoc 注释，使 API 文档从源码生成（经 T8 TypeDoc 生成），消除手写文档与代码的版本漂移。

迁移范围仅限 `lib/` 下文件（不含 `cli.ts`），覆盖设计文档 §2.2.1 定义的全部公开 API：

| 模块 | 文件 | 公开 API |
|------|------|---------|
| REST API | `lib/api.ts` | `startApiServer` |
| Code Quality | `lib/code-quality/index.ts` | `runAll`, `runModule`, `runDiff`, `runWatch`, `loadLocalLLMConfig`, `createCLI` |
| Knowledge Graph | `lib/knowledge-graph/index.ts` | `runKnowledgeGraph` |
| Custom Rules | `lib/custom-rules/index.ts` | `runCustomRules`, `scanFiles`, `scanFile`, `RULE_IMPLEMENTATIONS` |
| 类型 | `lib/types.ts` | `ApiOptions`, `GraphOptions`, `GraphResult`, `CodeQualityResult` 等全部 export 类型 |

> CLI 子命令文档保留在 `docs/usage.md` 中手写，不纳入 TypeDoc（`cli.ts` 不在 entryPoints 中）。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-2-2 | 将 `docs/api.md` 的内容迁移为源码 TSDoc 注释（覆盖 `lib/` 下的公开 API） | 源码中公开 API 均有 TSDoc 注释，注释内容覆盖原 api.md 的描述 |

**对应阶段 Spec 验收标准**：AC-6（源码公开 API 均有 TSDoc 注释，覆盖 §2.2.1 全部公开 API）

---

## 3. 实现步骤

### 3.1 TSDoc 编写规范

编写 TSDoc 时遵循以下规范（与项目现有 TSDoc 风格一致，参考 `lib/knowledge-graph/index.ts` 的 `runKnowledgeGraph` 与 `lib/types.ts` 的 `GraphOptions`/`ERROR_RUNTIME_SUB_TYPES`）：

1. **语言**：使用中文描述。
2. **格式**：使用块注释 `/** ... */`，非行内注释。
3. **@param 标签格式**：新加 TSDoc 统一采用 `@param name - description` 标准格式（带连字符），已有 TSDoc 在补全时同步对齐为此格式。这是 TSDoc 标准格式，TypeDoc 完整支持。
4. **标签**：
   - 函数使用 `@param` 描述参数，`@returns` 描述返回值，`@throws` 描述抛出的异常（如有）。
   - 接口/类型使用首行简要描述 + 后续行补充说明，字段使用 `/** ... */` 行内注释。
5. **内容来源**：从 `docs/api.md` 迁移端点说明、参数说明、响应示例；`api.md` 中无对应内容的 API（如 code-quality 模块函数），根据函数签名与实现逻辑编写 TSDoc。
6. **不修改代码逻辑**：仅添加/补全 TSDoc 注释，不修改任何函数体、类型定义、import 语句。

### 3.2 lib/api.ts — startApiServer

**当前状态**：`startApiServer`（第 103 行）无 TSDoc。

**需添加 TSDoc 内容**（从 `docs/api.md` 迁移）：

```
/**
 * 启动本地 REST API 服务，将运行日志、分析报告以 JSON 形式暴露给外部消费者
 * （如 AI 智能体、IDE 插件、CI 流水线）。
 *
 * 服务仅监听 127.0.0.1，不对外暴露，适合本地开发环境使用。
 * 所有日志读取均为只读操作，不会修改 .runtime-log-ignore/ 下的原始日志。
 *
 * 端点列表：
 *
 * | 方法 | 端点 | 说明 |
 * |------|------|------|
 * | GET | /health | 健康检查 |
 * | GET | /logs | 获取指定类型原始日志 |
 * | GET | /report | 获取分析报告 |
 * | GET | /errors/top | 获取高频错误 TOP N |
 * | GET | /timeline | 获取用户行为时间线 |
 * | POST | /suggest | 根据 errorId 生成 AI 修复提示词 |
 *
 * 所有端点（除 /health 外）均支持 date 查询参数，格式为 YYYY-MM-DD，默认当天。
 *
 * 通用响应格式：
 * - 成功：返回 200 与对应 JSON 数据。
 * - 路径不存在：返回 404 JSON：{ "error": "not found" }。
 * - 参数错误：返回 400 JSON，如 { "error": "invalid date", "detail": "date must be YYYY-MM-DD" }。
 * - 服务端异常：返回 500 JSON：{ "error": "internal error", "detail": "..." }。
 *
 * CORS 配置：默认不启用跨域。options.cors 为 true 时，响应头包含
 * Access-Control-Allow-Origin: *、Access-Control-Allow-Methods: GET, POST, OPTIONS、
 * Access-Control-Allow-Headers: Content-Type。
 *
 * @param options - API 服务配置（projectPath、port、cors）
 * @returns HTTP Server 实例
 */
```

**端点详解 TSDoc 内容**（可放在函数 TSDoc 中或以 `@remarks` 补充）：

- **GET /health**：健康检查，返回 `{ ok: true, project: "<项目名>" }`。
- **GET /logs**：查询参数 `type`（runtime/network/behavior/quality，默认 runtime）、`date`（YYYY-MM-DD，默认当天）。返回 `{ type, date, count, logs }`。runtime 子类型通过 `subType` 字段区分，完整列表见 {@link RuntimeSubType} 类型定义。
- **GET /report**：查询参数 `format`（json/md，默认 json）、`date`。返回 `{ format, date, report }`，report 含 meta/summary/topErrors/networkIssues/behaviorTimeline/qualitySummary。
- **GET /errors/top**：查询参数 `limit`（默认 10）、`date`。返回 `{ date, limit, errors }`，按 errorId 聚合。
- **GET /timeline**：查询参数 `date`。返回 `{ date, count, timeline }`，已按时间戳与 sequence 排序。
- **POST /suggest**：请求体 `{ errorId: string }`，查询参数 `date`。返回 `{ errorId, date, prompt }`。400：请求体非合法 JSON 或缺少 errorId；404：未找到对应 errorId。

### 3.3 lib/code-quality/index.ts — 6 个函数 + 4 个接口

**当前状态**：`runAll`、`runModule`、`runDiff`、`runWatch`、`loadLocalLLMConfig`、`createCLI` 均无 TSDoc；`CodeQualityAllOptions`、`CodeQualityModuleOptions`、`CodeQualityDiffOptions`、`CodeQualityWatchOptions` 接口无 TSDoc。

**需添加 TSDoc 的 API**：

#### 3.3.1 接口 TSDoc

| 接口 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `CodeQualityAllOptions` | 19 | 全量质量检查选项。projectPath 为老项目根路径；skip 为跳过阶段列表（type-check/lint/test） |
| `CodeQualityModuleOptions` | 24 | 模块级质量检查选项。projectPath 为老项目根路径；targets 为 src 下的源文件路径列表；model 为 LLM 模型名 |
| `CodeQualityDiffOptions` | 30 | git diff 质量检查选项。projectPath 为老项目根路径；base 为 diff 比较基线（默认 origin/main）；model 为 LLM 模型名 |
| `CodeQualityWatchOptions` | 36 | watch 模式选项。projectPath 为老项目根路径；debounce 为防抖毫秒（默认 800）；model 为 LLM 模型名 |

#### 3.3.2 函数 TSDoc

| 函数 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `runAll` | 134 | 对 Vue3 + JS + Webpack 老项目执行全量质量检查，依次执行 type-check → lint → test，任一失败整体失败。@param opts - 全量检查选项；@returns CodeQualityResult 含合并后的退出码、stdout、stderr 与 summary；@throws 老项目路径不存在/缺少 package.json/缺少 src 目录/skip 非法值时抛错 |
| `runModule` | 214 | 为 --target 指定的老项目源文件生成单测并执行。@param opts - 模块检查选项；@returns CodeQualityResult；@throws target 不位于 src 内/不支持扩展名/target 不存在/未提供 target 时抛错 |
| `runDiff` | 270 | 收集 git 变更的 src 内源文件，生成单测并执行。@param opts - diff 检查选项；@returns CodeQualityResult（无变更时返回 code=0）；@throws 老项目路径不存在/缺少 package.json/缺少 src 目录时抛错 |
| `runWatch` | 321 | 监听老项目 src 变更，实时生成单测并执行（默认 800ms 防抖）。@param opts - watch 选项；@returns CodeQualityResult（code=0，watch 持续运行直到 SIGINT/SIGTERM）；@throws 老项目路径不存在/缺少 package.json/缺少 src 目录时抛错 |
| `loadLocalLLMConfig` | 394（re-export） | 加载本地 LLM 配置。re-export 自 `./lib/load-local-config.js`，TSDoc 添加在 re-export 语句上方。上方已有 `//` 行注释，实现时将其内容迁移为 TSDoc 块注释 `/** */` 格式，删除原行注释，避免重复描述 |
| `createCLI` | 398 | 内部调试 CLI 入口，创建 Commander 程序实例（all/module/diff/watch 子命令）。不随 legacy-shield 主 CLI 发布，仅用于独立调试。@returns Command 实例。上方已有 `//` 行注释，实现时将其内容迁移为 TSDoc 块注释 `/** */` 格式，删除原行注释，避免重复描述 |

> **re-export TSDoc 注意**：`loadLocalLLMConfig` 通过 `export { loadLocalLLMConfig }` re-export。TypeDoc 0.28+ 支持在 re-export 语句上方添加 `/** ... */` 注释。若 TypeDoc 未正确拾取，需在原始定义文件 `lib/code-quality/lib/load-local-config.ts` 中补全 TSDoc。

### 3.4 lib/knowledge-graph/index.ts — runKnowledgeGraph（补全）

**当前状态**：`runKnowledgeGraph`（第 21 行）已有基础 TSDoc（第 14-20 行），内容为简要描述 + `@param` + `@returns`。

**需补全内容**（从 `docs/api.md` graph 子命令章节迁移相关 API 描述）：

```
/**
 * 知识图谱编排入口。
 * 编排 scanner → graph → analyzer → monorepo → output 完整流程。
 *
 * 基于 Babel AST 静态分析 import / export / require / dynamic-import 依赖关系，
 * 输出文件级依赖图、循环依赖链、hub 文件、分层架构等关键信息。
 *
 * 路径解析能力：
 * - 相对路径（./foo / ../bar）
 * - tsconfig/jsconfig paths alias（@/* / ~/* 等自定义 paths）
 * - webpack resolve.alias（对象格式，含 webpack 5 对象形态）
 * - vite resolve.alias（对象格式 + 数组格式）
 * - node_modules 包（含 scoped 包）
 * - 扩展名补全（.ts / .tsx / .js / .jsx / .vue）
 * - index 文件解析
 *
 * alias 优先级：tsconfig paths > vite alias > webpack alias。
 * 自动检测项目根目录下的 webpack.config.{js,ts} 与 vite.config.{ts,js}，无需新增 CLI 参数。
 *
 * monorepo 支持：自动识别 package.json workspaces / lerna.json / pnpm-workspace.yaml /
 * packages/* 目录约定，为每个子包生成独立图谱并合并为全局聚合图谱。
 *
 * 输出文件：
 * - knowledge-graph.json：JSON 格式知识图谱（meta / nodes / edges / cycles / stats）
 * - architecture-summary.md：Markdown 架构摘要（中文 6 章节结构）
 * - .cache.json：mtime 缓存（隐藏文件，用于增量更新）
 *
 * @param options - 图谱生成选项
 * @returns GraphResult 含耗时与统计指标
 */
```

### 3.5 lib/custom-rules/index.ts — 4 个 API

**当前状态**：`runCustomRules`、`scanFiles`、`scanFile`、`RULE_IMPLEMENTATIONS` 均无 TSDoc。

**需添加 TSDoc 的 API**：

| API | 行号 | TSDoc 内容 |
|-----|------|-----------|
| `runCustomRules` | 6 | 对老项目执行全部自定义规则扫描。遍历 RULE_IMPLEMENTATIONS 中所有规则（no-sync-script 除外，单独调用 scanHtmlForSyncScripts），合并命中结果。@param legacyRoot - 老项目根路径；@param options - 扫描选项（disabled 为禁用规则 ID 列表，scanOptions 为扫描范围）；@returns CustomRulesResult 含命中列表与统计摘要 |
| `scanFiles` | 35（re-export） | 对老项目执行指定规则的文件扫描。re-export 自 `./scanner.js`，TSDoc 添加在 re-export 语句上方 |
| `scanFile` | 35（re-export） | 对单个文件执行规则扫描。re-export 自 `./scanner.js`，TSDoc 添加在 re-export 语句上方 |
| `RULE_IMPLEMENTATIONS` | 36（re-export） | 自定义规则实现映射表，key 为规则名，value 为 ShieldRule 实现。re-export 自 `./rules/index.js`，TSDoc 添加在 re-export 语句上方 |

> **re-export TSDoc 注意**：`scanFiles`、`scanFile` 通过 `export { scanFiles, scanFile } from './scanner.js'` re-export；`RULE_IMPLEMENTATIONS` 通过 `export { RULE_IMPLEMENTATIONS } from './rules/index.js'` re-export。TypeDoc 0.28+ 支持在 re-export 语句上方添加 `/** ... */` 注释。若 TypeDoc 未正确拾取，需在原始定义文件中补全 TSDoc。

### 3.6 lib/types.ts — 全部公开类型

**当前状态**：大部分 export 类型无 TSDoc。`GraphOptions`（第 417 行）与 `GraphResult`（第 432 行）已有字段级 JSDoc，但缺少接口/类型级 TSDoc；`ERROR_RUNTIME_SUB_TYPES`（第 461 行）已有 TSDoc。

**字段级 TSDoc 指导**：字段级 TSDoc 仅针对语义不明显的字段添加；已有字段级 JSDoc 的类型（如 GraphOptions、GraphResult）保持现状；字段名语义清晰的类型（如 ApiOptions 的 projectPath/port/cors）无需为每个字段添加 TSDoc。

**需添加 TSDoc 的类型**（按逻辑分组）：

#### 日志类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `LogLevel` | 3 | 日志级别：error / warn / info |
| `RuntimeSubType` | 5 | 运行时日志子类型（js-error / promise-rejection / vue-render-error / pinia-error / vuex-error 等） |
| `BehaviorSubType` | 22 | 用户行为日志子类型（click / input / change / submit / keydown / route-change 等） |
| `NetworkSubType` | 33 | 网络日志子类型（xhr / fetch / static-resource / proxy-error / unknown） |
| `QualitySubType` | 35 | 质量日志子类型（code-quality / custom-rule） |

#### 日志记录接口

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `RuntimeLog` | 37 | 运行时日志记录结构 |
| `NetworkRequestRecord` | 55 | 网络请求记录（含脱敏 headers、body 摘要） |
| `NetworkResponseRecord` | 64 | 网络响应记录（含脱敏 headers、body 摘要） |
| `NetworkLog` | 75 | 网络日志记录结构 |
| `BehaviorTarget` | 90 | 用户行为目标元素信息 |
| `BehaviorLog` | 98 | 用户行为日志记录结构 |
| `QualityLog` | 111 | 质量日志记录结构 |
| `ShieldLog` | 125 | legacy-shield 日志联合类型（RuntimeLog / NetworkLog / BehaviorLog / QualityLog） |

#### 规则与风险类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `RiskType` | 127 | 静态风险类型：memory-leak / resource-load |
| `RuleHit` | 129 | 规则命中记录 |
| `StaticRiskItem` | 141 | 静态风险项 |
| `ShieldRule` | 153 | 自定义规则定义（含 Babel Visitor 生成函数） |
| `CustomRulesResult` | 161 | 自定义规则扫描结果（含命中列表与统计摘要） |
| `ScanOptions` | 171 | 文件扫描范围选项（include / exclude glob 列表） |

#### Code Quality 类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `CodeQualitySummary` | 176 | 质量检查摘要（退出码、测试状态、ESLint 问题数、类型检查状态） |
| `CodeQualityResult` | 183 | 质量检查结果（命令、退出码、stdout、stderr、摘要） |
| `RunCodeQualityOptions` | 193 | 质量检查运行选项（targets / base / skipList） |

#### Logger 与运行时类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `Logger` | 199 | 日志记录器接口（logRuntime / logNetwork / logBehavior / logQuality / close） |
| `StartProxyOptions` | 211 | 代理服务启动选项 |
| `StructuredLogEntry` | 239 | 结构化日志条目 |
| `StructuredLogger` | 255 | 结构化日志记录器接口 |
| `StartBrowserOptions` | 260 | 浏览器启动选项 |

#### 平台检测类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `PlatformType` | 220 | 平台类型：web / h5 |
| `DetectPlatformOptions` | 222 | 平台检测选项 |
| `DetectPlatformResult` | 227 | 平台检测结果（含推断上下文） |

#### 分析与报告类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `AnalyzerOptions` | 274 | 日志分析选项（date / networkIssueThresholdMs） |
| `AnalysisSummary` | 279 | 分析摘要（运行时错误数、网络请求数、行为数、ESLint 问题数、测试状态、自定义规则命中数） |
| `TopError` | 290 | 高频错误聚合项（按 errorId 聚合） |
| `NetworkIssue` | 302 | 网络问题记录 |
| `BehaviorTimelineItem` | 312 | 用户行为时间线条目 |
| `QualityAnalysisSummary` | 321 | 质量分析摘要 |
| `AnalysisResult` | 329 | 完整分析结果（summary / topErrors / networkIssues / behaviorTimeline / qualitySummary） |

#### 报告类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `ReportFormat` | 337 | 报告格式：md / json |
| `ReportOptions` | 339 | 报告生成选项 |
| `JsonReport` | 346 | JSON 报告结构（meta / summary / topErrors / networkIssues / behaviorTimeline / qualitySummary） |

#### API 与 CLI 选项类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `ApiOptions` | 359 | REST API 服务启动选项（projectPath / port / cors） |
| `FixPromptResult` | 365 | AI 修复提示词生成结果 |
| `ShieldCommandOptions` | 371 | shield 子命令选项 |
| `QualityCommandOptions` | 385 | quality 子命令选项 |
| `ReportCommandOptions` | 404 | report 子命令选项 |
| `ApiCommandOptions` | 411 | api 子命令选项 |

#### 知识图谱类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `GraphOptions` | 417 | 知识图谱生成选项（**已有字段级 JSDoc，需补充接口级 TSDoc**） |
| `GraphResult` | 432 | 知识图谱生成结果（**已有字段级 JSDoc，需补充类型级 TSDoc**） |

#### 其他类型

| 类型 | 行号 | TSDoc 内容 |
|------|------|-----------|
| `ShieldEmitEvent` | 451 | shield 运行时事件（type / payload） |

> **已存在 TSDoc 的类型**：`ERROR_RUNTIME_SUB_TYPES`（第 461 行）已有完整 TSDoc，无需修改。

---

## 4. 测试计划

### 4.1 TSDoc 覆盖率验证

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| `lib/api.ts` startApiServer 有 TSDoc | 读取源码，检查 `startApiServer` 上方是否有 `/** ... */` | TSDoc 存在，含端点列表与参数说明 |
| `lib/code-quality/index.ts` 6 个函数有 TSDoc | 读取源码，逐一检查 `runAll`/`runModule`/`runDiff`/`runWatch`/`loadLocalLLMConfig`/`createCLI` 上方 | 6 个 API 均有 TSDoc |
| `lib/code-quality/index.ts` 4 个接口有 TSDoc | 读取源码，逐一检查 `CodeQualityAllOptions`/`CodeQualityModuleOptions`/`CodeQualityDiffOptions`/`CodeQualityWatchOptions` 上方 | 4 个接口均有 TSDoc |
| `lib/knowledge-graph/index.ts` runKnowledgeGraph TSDoc 已补全 | 读取源码，检查 TSDoc 是否含路径解析能力、monorepo 支持、输出文件说明 | TSDoc 内容完整，覆盖 api.md graph 章节描述 |
| `lib/custom-rules/index.ts` 4 个 API 有 TSDoc | 读取源码，逐一检查 `runCustomRules`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` | 4 个 API 均有 TSDoc |
| `lib/types.ts` 全部 export 类型有 TSDoc | 读取源码，逐一检查所有 `export type`/`export interface`/`export const` 上方 | 全部公开类型均有 TSDoc（`ERROR_RUNTIME_SUB_TYPES` 已有，无需修改） |
| TSDoc 内容覆盖原 api.md | 对照 `docs/api.md` 端点详解，检查 `startApiServer` TSDoc 是否覆盖全部 6 个端点 | 端点列表、参数说明、响应格式、CORS 配置均已迁移 |

### 4.2 类型检查

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| TypeScript 编译零错误 | `pnpm typecheck` | 通过（TSDoc 为注释，不影响编译） |

### 4.3 回归测试

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| 构建通过 | `pnpm build` | 通过（仅添加注释，不修改代码逻辑） |
| 现有测试全量通过 | `pnpm test` | 通过（不修改任何函数体与类型定义） |

### 4.4 TypeDoc 生成预检（可选）

> TSDoc 添加完成后，可执行 `pnpm docs:gen`（依赖 T8 配置）预检 TypeDoc 是否能正确拾取 TSDoc。若 T8 尚未完成，可跳过此项，在 T8 中统一验证。

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| TypeDoc 生成成功 | `pnpm docs:gen` | docs/api/ 下生成 API 文档 |
| re-export TSDoc 拾取 | 检查生成产物中 `loadLocalLLMConfig`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` 是否有描述 | 若未拾取，需在原始定义文件中补全 TSDoc |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| 依赖 T6 已完成 | T6 引入 TypeDoc 与 typedoc.json，T7 需确认 entryPoints 覆盖的文件范围 | T7 启动前确认 T6 已通过评审且 typedoc.json entryPoints 包含 5 个 lib/ 文件 |
| re-export TSDoc 未被 TypeDoc 拾取 | `loadLocalLLMConfig`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` 的 TSDoc 在 re-export 语句上方，TypeDoc 可能未正确拾取 | 先在 re-export 语句上方添加 TSDoc；若 TypeDoc 生成产物中缺失，则在原始定义文件（`lib/code-quality/lib/load-local-config.ts`、`lib/custom-rules/scanner.ts`、`lib/custom-rules/rules/index.ts`）中补全 TSDoc |
| types.ts 类型数量多 | 约 50 个 export 类型，遗漏风险高 | 按 §3.6 分组逐一添加，完成后逐条核对 |
| TSDoc 内容与 api.md 不一致 | 迁移过程中信息丢失或描述偏差 | 完成后对照 api.md 逐端点核对；T8 生成 TypeDoc 产物后再次对照检查 |
| TSDoc 语法错误影响 TypeDoc 生成 | TSDoc 中 markdown 表格或特殊字符可能导致 TypeDoc 解析异常 | 简单表格（如端点列表）可使用 Markdown 表格语法，TypeDoc 通过 marked 解析支持；复杂嵌套表格以纯文本列表描述；T8 生成时验证表格渲染正确 |
| 不修改代码逻辑的约束 | 仅添加注释，不能改动函数体、类型字段、import | 实现时严格限定为注释添加，不触碰任何代码行 |

---

## 6. 变更范围

| 文件 | 变更类型 | 说明 |
|------|---------|------|
| `lib/api.ts` | 修改（加 TSDoc） | `startApiServer` 函数添加 TSDoc，迁移 api.md 端点说明 |
| `lib/code-quality/index.ts` | 修改（加 TSDoc） | `runAll`/`runModule`/`runDiff`/`runWatch`/`loadLocalLLMConfig`/`createCLI` 6 个函数 + `CodeQualityAllOptions`/`CodeQualityModuleOptions`/`CodeQualityDiffOptions`/`CodeQualityWatchOptions` 4 个接口添加 TSDoc |
| `lib/knowledge-graph/index.ts` | 修改（补全 TSDoc） | `runKnowledgeGraph` 补全 TSDoc（路径解析、monorepo、输出文件等） |
| `lib/custom-rules/index.ts` | 修改（加 TSDoc） | `runCustomRules`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` 4 个 API 添加 TSDoc |
| `lib/types.ts` | 修改（加 TSDoc） | 全部 export 类型添加 TSDoc（约 50 个，`ERROR_RUNTIME_SUB_TYPES` 已有无需修改） |

**可能涉及的补充文件**（若 re-export TSDoc 未被 TypeDoc 拾取）：

| 文件 | 变更类型 | 说明 |
|------|---------|------|
| `lib/code-quality/lib/load-local-config.ts` | 修改（加 TSDoc） | `loadLocalLLMConfig` 原始定义补全 TSDoc（仅 TypeDoc 未拾取 re-export TSDoc 时） |
| `lib/custom-rules/scanner.ts` | 修改（加 TSDoc） | `scanFiles`/`scanFile` 原始定义补全 TSDoc（仅 TypeDoc 未拾取 re-export TSDoc 时） |
| `lib/custom-rules/rules/index.ts` | 修改（加 TSDoc） | `RULE_IMPLEMENTATIONS` 原始定义补全 TSDoc（仅 TypeDoc 未拾取 re-export TSDoc 时） |

**不修改的文件**：

- `cli.ts`（CLI 子命令文档保留在 `docs/usage.md`，不纳入 TypeDoc）
- `docs/api.md`（T9 负责改写为入口索引，T7 不修改）
- 任何函数体、类型字段定义、import 语句（仅添加注释）

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，发现 1 个 P1 + 5 个 P2 问题
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无（第一轮 P1-1 已修复闭环）

### P2 优化项

无（第一轮 P2-1~P2-5 已全部处理）
