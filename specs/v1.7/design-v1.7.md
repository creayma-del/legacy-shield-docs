# legacy-shield v1.7 设计文档

> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应需求文档：requirements-v1.7.md
> 对应需求分解文档：requirements-decomposition-v1.7.md

## 1. 总体架构

### 1.1 现状

```
legacy-shield/                    # 主仓库
├── docs/
│   ├── api.md                    # 手写 API 文档
│   ├── usage.md                  # 手写使用文档
│   ├── custom-rules.md           # 手写规则文档
│   └── specs/                   # 100+ spec 文档（v1.1~v1.6 + 早期）
│       ├── common/
│       ├── phases/
│       ├── v1.1/ ~ v1.6/
│       ├── design.md
│       ├── requirements.md
│       └── ...
├── lib/                          # 源码
├── tests/
├── package.json                  # 无 workspace 配置
└── ...
```

### 1.2 目标架构

```
legacy-shield/                    # 主仓库（轻量）
├── docs/
│   ├── INDEX.md                  # 文档总入口（新建）
│   ├── api.md                    # 入口索引（改为指向 docs/api/）
│   ├── api/                      # TypeDoc 生成产物（新建）
│   ├── usage.md                  # 活文档（保留）
│   ├── custom-rules.md           # 活文档（保留）
│   └── specs/
│       ├── v1.7/                 # 仅保留当前版本 spec
│       ├── design-v1.7.md        # v1.7 根级 spec
│       ├── requirements-v1.7.md
│       ├── requirements-decomposition-v1.7.md
│       └── execution-plan-v1.7.md
├── lib/                          # 源码（公开 API 加 TSDoc）
├── scripts/
│   └── docs-lint.ts              # 契约校验脚本（新建）
├── tests/
├── pnpm-workspace.yaml           # 新建
├── package.json                  # 增加 @legacy-shield/docs 依赖 + docs:gen / docs:lint script
├── typedoc.json                  # TypeDoc 配置（新建）
└── ...

legacy-shield-docs/               # 归档仓库（独立，../legacy-shield-docs）
├── package.json                  # name: @legacy-shield/docs
├── specs/                        # 历史 spec 归档
│   ├── common/
│   ├── phases/
│   ├── v1.1/ ~ v1.6/
│   ├── design.md
│   └── ...
└── README.md                     # 归档仓库说明
```

### 1.3 设计原则

1. **单一事实源**：API 文档从源码 TSDoc 生成，不再手写，消除版本漂移。
2. **分层归档**：活文档贴近代码，冻结文档独立归档，主仓库轻量。
3. **契约校验**：docs-lint 强制文档引用与源码一致，不一致即报错。
4. **零破坏迁移**：不修改现有代码行为，不破坏现有测试。

## 2. 模块详细设计

### 2.1 模块 1：分层归档

#### 2.1.1 归档仓库结构

```
legacy-shield-docs/
├── package.json          # { "name": "@legacy-shield/docs", "version": "1.0.0", "private": true }
├── README.md             # 归档仓库说明
└── specs/                # 迁移后的历史 spec
    ├── common/
    │   ├── project-rules.md
    │   └── typescript-migration.md
    ├── phases/
    │   ├── phase-1-spec.md
    │   └── ... phase-5-spec.md
    ├── v1.1/
    │   ├── phases/
    │   ├── acceptance-report-v1.1.md
    │   ├── design-v1.1.md
    │   ├── execution-plan-v1.1.md
    │   └── requirements-v1.1.md
    ├── v1.2/ ~ v1.6/     # 同结构
    ├── design.md
    ├── execution-plan.md
    ├── requirements.md
    └── acceptance-report.md
```

**迁移执行方式**：

使用 `cp + git rm` 方式迁移（`git mv` 无法跨仓库移动文件）。迁移步骤：

1. 创建归档仓库 `legacy-shield-docs`，初始化 `package.json` 与 `README.md`，创建 `specs/` 空目录。
2. 在主仓库执行 `cp -r docs/specs/v1.1 ../legacy-shield-docs/specs/v1.1`（逐版本复制）。
3. 在归档仓库执行 `git add . && git commit`，记录迁移（commit message 中记录主仓库原始 commit hash 以便追溯）。
4. 在主仓库执行 `git rm -r docs/specs/v1.1 && git commit`，记录删除。
5. 对 common/、phases/、根目录早期文档重复步骤 2-4。
6. 迁移后校验文件数：主仓库 `docs/specs/` 下仅剩 `v1.7/` 目录及 v1.7 根级 spec 文件（design-v1.7.md、requirements-v1.7.md、requirements-decomposition-v1.7.md、execution-plan-v1.7.md），归档仓库 `specs/` 下文件数与迁移前一致。

> 注意：`cp + git rm` 方式不保留文件 git 历史（归档仓库是新 commit，主仓库是删除 commit），迁移时在归档仓库 commit message 中记录主仓库原始 commit hash 以便后续追溯。

#### 2.1.2 pnpm workspace 引入

主仓库新增 `pnpm-workspace.yaml`：

```yaml
packages:
  - '../legacy-shield-docs'
```

主仓库 `package.json` 增加依赖：

```json
{
  "devDependencies": {
    "@legacy-shield/docs": "workspace:*"
  }
}
```

引入后，归档仓库内容可通过 `node_modules/@legacy-shield/docs/specs/` 访问，AI 与脚本均可读取。

#### 2.1.3 主仓库 docs/ 重组

迁移后主仓库 `docs/` 只保留：

| 文件 | 类型 | 说明 |
|------|------|------|
| `INDEX.md` | 新建 | 文档总入口，列出所有文档位置与状态 |
| `api.md` | 改写 | 入口索引，指向 `docs/api/` |
| `api/` | 生成 | TypeDoc 产物 |
| `usage.md` | 保留 | 活文档 |
| `custom-rules.md` | 保留 | 活文档 |
| `specs/v1.7/` | 保留 | 当前版本 spec（含任务 Spec 等） |
| `specs/design-v1.7.md` | 保留 | v1.7 根级 spec |
| `specs/requirements-v1.7.md` | 保留 | v1.7 根级 spec |
| `specs/requirements-decomposition-v1.7.md` | 保留 | v1.7 根级 spec |
| `specs/execution-plan-v1.7.md` | 保留 | v1.7 根级 spec |

#### 2.1.4 INDEX.md 设计

`INDEX.md` 作为文档总入口，结构：

```markdown
# legacy-shield 文档索引

## 活文档（主仓库）
| 文档 | 路径 | 说明 |
|------|------|------|
| API 文档 | docs/api/ | TypeDoc 生成，pnpm docs:gen 更新 |
| 使用指南 | docs/usage.md | 手写 |
| 自定义规则 | docs/custom-rules.md | 手写 |

## 当前版本 Spec（主仓库）
| 文档 | 路径 | 状态 |
|------|------|------|
| v1.7 需求 | docs/specs/requirements-v1.7.md | 草稿 |
| ... | ... | ... |

## 历史归档（@legacy-shield/docs）
| 版本 | 路径 | 状态 |
|------|------|------|
| v1.1~v1.6 | node_modules/@legacy-shield/docs/specs/v{x}.{y}/ | 已归档冻结 |
| 早期文档 | node_modules/@legacy-shield/docs/specs/ | 已归档冻结 |
| common | node_modules/@legacy-shield/docs/specs/common/ | 已归档冻结 |

## 历史文档路径映射表

迁移前后路径对应关系，供 spec-guardian skill 与 AI 查询历史文档时定位：

| 迁移前路径（主仓库） | 迁移后路径（归档仓库） |
|---------------------|----------------------|
| docs/specs/v1.1/ | node_modules/@legacy-shield/docs/specs/v1.1/ |
| docs/specs/v1.2/ | node_modules/@legacy-shield/docs/specs/v1.2/ |
| docs/specs/v1.3/ | node_modules/@legacy-shield/docs/specs/v1.3/ |
| docs/specs/v1.4/ | node_modules/@legacy-shield/docs/specs/v1.4/ |
| docs/specs/v1.5/ | node_modules/@legacy-shield/docs/specs/v1.5/ |
| docs/specs/v1.6/ | node_modules/@legacy-shield/docs/specs/v1.6/ |
| docs/specs/common/ | node_modules/@legacy-shield/docs/specs/common/ |
| docs/specs/phases/ | node_modules/@legacy-shield/docs/specs/phases/ |
| docs/specs/design.md | node_modules/@legacy-shield/docs/specs/design.md |
| docs/specs/requirements.md | node_modules/@legacy-shield/docs/specs/requirements.md |
| docs/specs/execution-plan.md | node_modules/@legacy-shield/docs/specs/execution-plan.md |
| docs/specs/acceptance-report.md | node_modules/@legacy-shield/docs/specs/acceptance-report.md |
```

> **路径映射表的作用**：spec-guardian skill 在执行归档检查（如"已归档文档不可直接修改"）时需定位历史文档。迁移后历史文档路径从 `docs/specs/v1.6/design-v1.6.md` 变为 `node_modules/@legacy-shield/docs/specs/v1.6/design-v1.6.md`，路径映射表提供迁移前后的对应关系，避免 skill 与 AI 找不到历史文档。详见 8.1 场景 2。

### 2.2 模块 2：API 文档代码化

#### 2.2.1 公开 API 范围

需加 TSDoc 的公开 API（`lib/` 下 export 的函数/类/类型）：

| 模块 | 文件 | 公开 API | 纳入 TypeDoc |
|------|------|---------|-------------|
| REST API | `lib/api.ts` | `startApiServer` | 是 |
| Code Quality | `lib/code-quality/index.ts` | `runAll`, `runModule`, `runDiff`, `runWatch`, `loadLocalLLMConfig`, `createCLI` | 是 |
| Knowledge Graph | `lib/knowledge-graph/index.ts` | `runKnowledgeGraph` | 是 |
| Custom Rules | `lib/custom-rules/index.ts` | `runCustomRules`, `scanFiles`, `scanFile`, `RULE_IMPLEMENTATIONS` | 是 |
| CLI | `cli.ts` | 5 个子命令：shield, quality, report, api, graph | 否（CLI 是命令行入口，非可 import 的 API，文档保留在 usage.md） |
| 类型 | `lib/types.ts` | `ApiOptions`, `GraphOptions`, `GraphResult`, `CodeQualityResult` 等 | 是 |

#### 2.2.2 TypeDoc 配置

新建 `typedoc.json`：

```json
{
  "entryPoints": [
    "lib/api.ts",
    "lib/code-quality/index.ts",
    "lib/knowledge-graph/index.ts",
    "lib/custom-rules/index.ts",
    "lib/types.ts"
  ],
  "out": "docs/api",
  "name": "legacy-shield API",
  "readme": "none",
  "excludePrivate": true,
  "excludeInternal": true,
  "tsconfig": "tsconfig.json"
}
```

#### 2.2.3 docs:gen 脚本

`package.json` scripts 增加：

```json
{
  "docs:gen": "typedoc"
}
```

执行 `pnpm docs:gen` 后，`docs/api/` 下生成多文件 API 文档（按模块拆分）。

#### 2.2.4 api.md 改写

原 `docs/api.md`（手写 REST API + graph 子命令文档）改为入口索引：

```markdown
# legacy-shield API 文档

API 文档由 TypeDoc 从源码 TSDoc 自动生成（HTML 格式）。

## 在线文档

- [API 文档首页](./api/index.html)（浏览器打开）

## 更新方式

\`\`\`bash
pnpm docs:gen
\`\`\`

## 源码位置

- REST API：lib/api.ts
- Code Quality：lib/code-quality/index.ts
- Knowledge Graph：lib/knowledge-graph/index.ts
- Custom Rules：lib/custom-rules/index.ts
- CLI 入口：cli.ts（CLI 文档见 docs/usage.md）
```

#### 2.2.5 TSDoc 迁移策略

将原 `api.md` 中的端点说明、参数说明、响应示例迁移为源码 TSDoc（仅 `lib/` 下文件，不含 `cli.ts`）：

- `lib/api.ts` 的 `startApiServer` 函数加 TSDoc，描述所有端点
- `lib/code-quality/index.ts` 的 `runAll`/`runModule`/`runDiff`/`runWatch`/`loadLocalLLMConfig`/`createCLI` 加 TSDoc
- `lib/knowledge-graph/index.ts` 的 `runKnowledgeGraph` 加 TSDoc（已有部分注释，需补全）
- `lib/custom-rules/index.ts` 的 `runCustomRules`/`scanFiles`/`scanFile`/`RULE_IMPLEMENTATIONS` 加 TSDoc
- `lib/types.ts` 的公开类型加 TSDoc

> CLI 子命令文档保留在 `docs/usage.md` 中手写，不纳入 TypeDoc，因为 CLI 是命令行入口而非可 import 的库 API。

### 2.3 模块 3：契约校验（docs-lint）

#### 2.3.1 脚本位置与入口

新建 `scripts/docs-lint.ts`，编译后为 `dist/scripts/docs-lint.js`。

`package.json` scripts 增加：

```json
{
  "docs:lint": "node ./dist/scripts/docs-lint.js"
}
```

#### 2.3.2 校验范围

仅校验主仓库活文档：

- `docs/INDEX.md`
- `docs/usage.md`
- `docs/custom-rules.md`
- `docs/api.md`

不校验：
- `docs/api/`（TypeDoc 生成产物，由源码保证一致性）
- `docs/specs/`（spec 文档，不强制校验）
- 归档仓库文档（已冻结，不再变化）

> **INDEX.md 一致性校验（P3 优化，本期不实现）**：docs-lint 当前仅校验 INDEX.md 中的代码符号/CLI/配置/路径引用，不校验 INDEX.md 列出的文档列表与实际文件系统是否一致。若新增/删除文档后忘记更新 INDEX.md，不会被发现。此校验可作为后续优化项，在 docs-lint 中增加 INDEX.md 文档列表与文件系统的一致性校验。

#### 2.3.3 校验规则

docs-lint 校验四类引用：

| 引用类型 | 提取方式 | 校验来源 | 示例 |
|---------|---------|---------|------|
| 代码符号 | markdown 中反引号包裹的标识符 | lib/ 下 export 的函数/类/类型名 | `startApiServer`、`runAll` |
| CLI 命令 | markdown 中 `node ./dist/cli.js <cmd>`、`legacy-shield <cmd>`、`pnpm <cmd>` 等模式 | cli.ts 中注册的子命令 | `shield`、`quality`、`api`、`graph`、`report` |
| 配置项 | markdown 中 `--<option>` 模式 | cli.ts 中 option / requiredOption 定义 | `--project`、`--port`、`--cors` |
| 文件路径 | markdown 中 `lib/*.ts`、`docs/*.md` 模式 | 文件系统实际存在 | `lib/api.ts`、`docs/usage.md` |

#### 2.3.4 技术实现

**符号收集**（源码侧）：

复用项目已有 Babel 依赖（`@babel/parser`、`@babel/traverse`、`@babel/types`），解析 `lib/` 下所有 `.ts` 文件，收集所有 `export` 的函数名、类名、类型名、接口名。

> **Babel parser 配置**：`@babel/parser` 默认不支持 TypeScript 语法，必须显式启用 TypeScript 插件：
> ```typescript
> import { parse } from '@babel/parser';
> const ast = parse(code, {
>   sourceType: 'module',
>   plugins: ['typescript'],
> });
> ```

```
1. 遍历 lib/**/*.ts
2. @babel/parser 解析为 AST（plugins: ['typescript']）
3. @babel/traverse 遍历 ExportNamedDeclaration / ExportDefaultDeclaration
4. 收集导出符号名集合
```

**CLI 命令与选项收集**（源码侧）：

同样使用 Babel AST 解析 `cli.ts`，收集 commander 注册的子命令与选项：

```
1. @babel/parser 解析 cli.ts 为 AST（plugins: ['typescript']）
2. @babel/traverse 遍历 CallExpression 节点
3. 对 .command('xxx') 调用，提取第一个字符串参数作为子命令名
   - 收集结果：shield, quality, report, api, graph
4. 对 .option('--xxx <type>', ...) / .option('--xxx', ...) /
   .requiredOption('--xxx <type>', ...) / .requiredOption('--xxx', ...) 调用，
   提取第一个参数中的选项名
   - .option 与 .requiredOption 在 commander 中均注册为选项，仅是否必填的区别，选项名提取规则一致
   - 解析规则：从 '--project <path>' 提取 'project'，从 '--no-body' 提取 'body'
   - 对 --no- 前缀选项，commander 内部存储为去掉 no- 前缀的名称（如 --no-body 存储为 body）
   - 收集结果：project, target, proxy-port, start-page, headless, body, insecure, ...
5. 子命令集合与选项集合供文档侧 CLI 命令/配置项校验使用
```

**引用提取**（文档侧）：

用正则从 markdown 提取四类引用：

```
代码符号：/`([A-Z][a-zA-Z]\w*|[a-z][a-zA-Z]\w*)`/g  （反引号包裹的标识符）
CLI 命令：/(?:node \.\/dist\/cli\.js|legacy-shield|pnpm)\s+(\w+)/g  （支持多种调用方式）
配置项：/--([a-z][a-z-]*)/g  （仅在有效 CLI 命令上下文中校验，见下方比对与输出）
文件路径：/(lib\/[^\s`)]+\.ts|docs\/[^\s`)]+\.md)/g
```

**比对与输出（白名单匹配策略）**：

校验语义为"文档引用了源码符号 → 源码必须存在"，而非"所有反引号内容 → 必须是源码符号"。CLI 命令同理：仅当提取到的命令在子命令集合中时才校验，不在集合中的命令直接跳过（因为 `pnpm build`、`pnpm test`、`pnpm exec` 等是 npm/pnpm 通用命令，非 legacy-shield CLI 子命令）。即：

```
对每个文档中的每个引用：
  代码符号：
    若反引号内容在源码符号集合中 → 校验通过
    若反引号内容不在源码符号集合中 → 跳过（不报错，因为可能是普通英文单词）
    仅当反引号内容形如源码符号（大驼峰/小驼峰且在集合中）时才校验
  CLI 命令：
    若命令在 cli.ts 注册的子命令集合中 → 校验通过
    若命令不在集合中 → 跳过（不报错，因为可能是 pnpm/npm 通用命令如 build/test/exec）
  配置项：
    仅校验出现在有效 CLI 命令上下文（同行或相邻行有子命令集合内的 CLI 命令引用）的 --option
    pnpm install、pnpm build 等非子命令的 pnpm 通用命令不构成有效 CLI 命令上下文，其后的 --option 不校验
    对 --no-<option> 模式，提取实际选项名 <option>（commander 的 --no- 前缀语法）
    若配置项在 cli.ts option / requiredOption 定义集合中 → 校验通过
    若不在集合中 → 记录不一致
  文件路径：
    若路径在文件系统中存在 → 校验通过
    若不存在 → 记录不一致
若存在不一致：
  输出所有不一致项（文件路径、行号、引用内容、类型、原因）
  退出码 1
否则：
  输出 "docs-lint: all references valid"
  退出码 0
```

#### 2.3.5 误报控制

为降低误报，以下情况跳过校验：

- **代码符号白名单策略**：仅当反引号内容在源码符号集合中存在时才校验，不在集合中的反引号内容直接跳过（不报错）。
- **CLI 命令白名单策略**：仅当提取到的命令在 cli.ts 注册的子命令集合中时才校验，不在集合中的命令直接跳过（不报错）。避免 `pnpm build`、`pnpm test`、`pnpm exec`、`pnpm install` 等 npm/pnpm 通用命令被误报为不一致。
- **TypeScript 关键字 denylist**：对 `true`、`false`、`null`、`undefined`、`string`、`number`、`boolean`、`void`、`never`、`any`、`unknown` 等关键字/内置类型显式跳过。
- **反引号内容过滤**：含空格、含中文、为完整句子的反引号内容跳过。
- **配置项上下文校验**：`--option` 仅在**有效 CLI 命令上下文**（同行或相邻行有**子命令集合内的** CLI 命令引用）中校验。`pnpm install`、`pnpm build` 等非子命令的 pnpm 通用命令不构成有效 CLI 命令上下文，其后的 `--save`、`--dev` 等 npm/pnpm 选项不校验。
- **`--no-` 前缀处理**：对 `--no-<option>` 模式，提取 `<option>` 作为实际选项名校验（commander 的 `--no-` 前缀语法）。源码侧收集时需同步处理：cli.ts 中 `.option('--no-body', ...)` 注册的选项，commander 内部存储为 `body`（布尔型，默认 true，`--no-body` 表示设为 false），因此源码符号集合中存储 `body` 而非 `no-body`，与文档侧提取的 `<option>` 保持一致。
- **配置项 `--` 后跟非字母时跳过**。

## 3. 数据流与状态变更

### 3.1 文档生成流

```
源码 TSDoc 注释
    │
    ▼
TypeDoc（pnpm docs:gen）
    │
    ▼
docs/api/ 目录（生成产物）
```

### 3.2 文档校验流

```
活文档（INDEX.md / usage.md / custom-rules.md / api.md）
    │
    ├─ 提取引用（代码符号 / CLI 命令 / 配置项 / 文件路径）
    │
    ▼
源码符号集合（Babel AST 收集） + CLI 命令集合 + 配置项集合 + 文件系统
    │
    ▼
比对 → 一致 / 不一致
    │
    ▼
输出结果（退出码 0 / 1）
```

### 3.3 归档引入流

```
归档仓库 legacy-shield-docs/
    │
    ▼
pnpm workspace 引入
    │
    ▼
node_modules/@legacy-shield/docs/
    │
    ▼
主仓库可通过 node_modules/@legacy-shield/docs/specs/ 访问历史 spec
```

## 4. 接口设计

### 4.1 docs-lint 脚本接口

```typescript
// scripts/docs-lint.ts
interface DocsLintOptions {
  docsDir: string;       // docs 目录路径，默认 docs/
  projectRoot: string;   // 项目根路径，默认 process.cwd()
}

interface DocsLintResult {
  valid: boolean;
  errors: DocsLintError[];
}

interface DocsLintError {
  file: string;       // 文档文件路径
  line: number;       // 行号
  reference: string;  // 引用内容
  type: 'symbol' | 'cli' | 'option' | 'path';
  reason: string;     // 不一致原因
}

export function runDocsLint(options: DocsLintOptions): DocsLintResult;
```

**CLI 入口**：

`scripts/docs-lint.ts` 末尾包含 CLI 入口，供 `node ./dist/scripts/docs-lint.js` 直接执行。

> **入口判定方式**：不能使用 `import.meta.url === \`file://${process.argv[1]}\`` 直接比较，因为 `process.argv[1]` 在通过 `node ./dist/scripts/docs-lint.js`（相对路径）执行时为相对路径，而 `import.meta.url` 为绝对 file:// URL，两者字符串不相等导致入口判定静默失败。必须使用 `fileURLToPath` 将 `import.meta.url` 转为绝对路径，再用 `resolve` 规范化 `process.argv[1]` 后比较：

```typescript
// scripts/docs-lint.ts 末尾
import { fileURLToPath } from 'node:url';
import { resolve } from 'node:path';

if (fileURLToPath(import.meta.url) === resolve(process.argv[1])) {
  const result = runDocsLint({
    docsDir: resolve(process.cwd(), 'docs'),
    projectRoot: process.cwd(),
  });
  if (result.errors.length > 0) {
    for (const err of result.errors) {
      console.error(`[docs-lint] ${err.file}:${err.line} ${err.type}: ${err.reference} → ${err.reason}`);
    }
    process.exit(1);
  } else {
    console.log('docs-lint: all references valid');
    process.exit(0);
  }
}
```

### 4.2 TypeDoc 配置接口

通过 `typedoc.json` 配置，无自定义代码接口。

## 5. 依赖与集成

### 5.1 新增依赖

| 依赖 | 类型 | 用途 | 版本要求 |
|------|------|------|---------|
| `typedoc` | devDependency | 从 TSDoc 生成 API 文档 | `^0.28.18`（TypeDoc 0.27.x 及 0.28.0-0.28.17 仅支持 TypeScript 5.0-5.8；0.28.18（2026-03-23）起新增 TypeScript 6.0 支持，按 TypeDoc 支持最新两个 TS 版本的策略兼容 5.9，本项目使用 `^0.28.18` 兼容 TypeScript 5.9.3） |

### 5.2 复用依赖

| 依赖 | 用途 |
|------|------|
| `@babel/parser` | docs-lint 解析源码 AST |
| `@babel/traverse` | docs-lint 遍历 AST 收集符号 |
| `@babel/types` | docs-lint AST 节点类型判断 |

### 5.3 pnpm workspace 集成

主仓库 `pnpm-workspace.yaml` 新增 `../legacy-shield-docs`，`package.json` 新增 `@legacy-shield/docs: workspace:*` 依赖。

### 5.4 npm scripts 集成

| script | 命令 | 说明 |
|--------|------|------|
| `docs:gen` | `typedoc` | 生成 API 文档到 docs/api/ |
| `docs:lint` | `node ./dist/scripts/docs-lint.js` | 校验文档引用一致性 |

## 6. 非功能性设计

### 6.1 兼容性

- **现有 CLI 命令**：不修改，行为不变。
- **现有测试**：不修改，测试中无 docs/ 文档路径引用（已评估）。
- **tsconfig.json 变更**：现有 `tsconfig.json` 的 `include` 不含 `scripts/`，需新增 `scripts/**/*.ts`，否则 `scripts/docs-lint.ts` 无法编译为 `dist/scripts/docs-lint.js`。变更内容：

```json
{
  "include": [
    "cli.ts",
    "bin/**/*.ts",
    "lib/**/*.ts",
    "scripts/**/*.ts"
  ]
}
```

`tsconfig.test.json` 同步增加 `scripts/**/*.ts`（若 docs-lint 需单元测试）。`scripts/strip-inject-export.js` 仅修改 `dist/lib/inject.iife.js`，不影响 `dist/scripts/` 产物。

- **spec-guardian skill**：skill 中的 `docs/specs/` 路径引用是命名规范，v1.7+ spec 仍在主仓库 docs/specs/ 下，无需调整（已评估，详见 8.1）。
- **CI/CD 适配**：本项目当前无 CI/CD 配置（无 `.github/workflows/`、无 `.gitlab-ci.yml` 等），因此本期不涉及 CI/CD 改造。若未来引入 CI/CD，需注意：pnpm workspace 引入 `../legacy-shield-docs` 后，CI 环境需同时检出两个仓库（如 GitHub Actions 需增加归档仓库 checkout 步骤，或使用 `actions/checkout` 的 `path` 参数将归档仓库检出至 `../legacy-shield-docs`）。开发者 onboarding 需在 README 或 INDEX.md 中补充说明：克隆主仓库后需额外克隆归档仓库至同级目录，否则 `pnpm install` 失败。

### 6.2 性能

- **docs-lint**：校验 4 个活文档 + 解析 lib/ 下约 55 个 .ts 文件（含 cli/、code-quality/、custom-rules/、knowledge-graph/、runtime-monitor/ 子目录及根级文件），预计 < 2s。
- **docs:gen**：TypeDoc 生成，预计 < 5s。

### 6.3 可维护性

- API 文档从源码生成，改代码时同步改 TSDoc，天然对齐。
- docs-lint 作为独立 npm script（`pnpm docs:lint`）执行，文档与代码不一致即报错；若未来引入 CI/CD，可将 docs:lint 纳入 CI 流水线。
- INDEX.md 手动维护，文档结构变化时同步更新。

## 7. 风险与回滚方案

| 风险 | 影响 | 应对措施 | 回滚方案 |
|------|------|---------|---------|
| 归档仓库路径错误 | pnpm install 失败 | T3 验证 workspace 配置 | 删除 pnpm-workspace.yaml，恢复 docs/specs/ |
| TypeDoc 生成失败 | docs:gen 报错 | T8 验证 typedoc.json 配置 | 移除 typedoc 依赖，恢复手写 api.md |
| docs-lint 误报 | 校验失败 | T10 细化提取规则，优先 AST | 移除 docs:lint script |
| TSDoc 迁移遗漏 | API 文档不完整 | T7 分批迁移，优先核心 API | 补全 TSDoc |
| 迁移过程文件丢失 | 历史 spec 丢失 | T2 迁移前 git 备份 | 从 git 历史恢复 |
| pnpm workspace 与现有依赖冲突 | install 失败 | T3 验证依赖解析 | 移除 workspace 配置 |

### 回滚策略

若整体方案需回滚：

1. 删除 `pnpm-workspace.yaml`，移除 `@legacy-shield/docs` 依赖。
2. 从 git 历史恢复 `docs/specs/` 下的历史 spec。
3. 移除 `typedoc` 依赖与 `docs:gen` script，恢复手写 `api.md`。
4. 移除 `scripts/docs-lint.ts` 与 `docs:lint` script。
5. 删除 `docs/api/`、`docs/INDEX.md`。

## 8. 兼容性评估结论（REQ-4-2 / REQ-4-3）

### 8.1 spec-guardian skill 路径引用评估（REQ-4-2）

**评估方法**：搜索 `.trae/skills/` 下所有 skill 文件中对 `docs/specs/` 的引用。

**评估结论**：5 个 skill 文件引用了 `docs/specs/` 路径，分两类场景：

**场景 1：命名规范（新文档创建）**

skill 中的路径引用（如 `docs/specs/requirements-v{x}.{y}.md`）用于指导新版本 spec 文档的创建位置。迁移后：

- v1.7+ 的 spec 文档仍在主仓库 `docs/specs/v1.7/` 下创建。
- skill 中的命名规范仍适用于主仓库。

**是否需调整**：**否**。

**场景 2：历史文档读取（归档检查）**

spec-guardian skill 在执行归档检查时（如"已归档文档不可直接修改"），需要定位历史文档。迁移后历史文档路径从 `docs/specs/v1.6/design-v1.6.md` 变为 `node_modules/@legacy-shield/docs/specs/v1.6/design-v1.6.md`。

此外，`spec-guardian-templates/SKILL.md` 中引用 `docs/specs/project-rules.md` 作为"项目总规范"路径，但实际文件在 `docs/specs/common/project-rules.md`，迁移后在 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`。这是迁移前已存在的不一致，迁移会使其更难发现。

**是否需调整**：**是（需在 INDEX.md 中提供路径映射表）**。具体措施：

1. 在 `docs/INDEX.md` 中增加"历史文档路径映射表"，列出迁移前后的路径对应关系，供 spec-guardian skill 与 AI 查询。
2. spec-guardian skill 本身不修改流程规则，但在需要读取历史 spec 时，通过 INDEX.md 的路径映射表定位。
3. 对 `project-rules.md` 路径不一致问题（`docs/specs/project-rules.md` vs `docs/specs/common/project-rules.md`），在迁移时一并修正：归档仓库中统一为 `specs/common/project-rules.md`，INDEX.md 路径映射表记录正确路径。

### 8.2 测试路径引用评估（REQ-4-3）

**评估方法**：搜索 `tests/` 下所有文件中对 `docs/` 的引用。

**评估结论**：仅 `tests/fixtures/vue3/vendor/` 下的第三方库 JS 文件匹配，这些是测试用的 vendor 文件，非文档引用。测试中无引用 `docs/` 文档路径的断言。

**是否需调整**：**否**。测试不引用文档路径，无需调整。

### 8.3 lib/ 代码路径引用评估

**评估方法**：搜索 `lib/` 下所有文件中对 `docs/specs/` 的引用。

**评估结论**：无匹配。代码中不引用 docs/specs/ 路径。

**是否需调整**：**否**。
