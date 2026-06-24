# 实现 docs-lint 脚本

> 任务编号：T10
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.3、§4.1
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T10
> 依赖任务：无
> 评审记录：见本文档末尾

---

## 1. 任务目标

实现独立的 docs-lint 脚本（`scripts/docs-lint.ts`），校验主仓库活文档（`docs/INDEX.md`、`docs/usage.md`、`docs/custom-rules.md`、`docs/api.md`）中引用的代码符号、CLI 命令、配置项、文件路径是否与源码一致，不一致时输出文件路径、行号、原因，退出码非 0。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-3-1 | 新增独立 docs-lint 脚本，作为独立 npm script `docs:lint` | `scripts/docs-lint.ts` 实现完整，编译为 `dist/scripts/docs-lint.js` 可执行（docs:lint script 在 T11 配置） |
| REQ-3-2 | docs-lint 校验范围：仅主仓库活文档 | 校验 `docs/INDEX.md`、`docs/usage.md`、`docs/custom-rules.md`、`docs/api.md` 四个文件 |
| REQ-3-3 | docs-lint 校验规则：代码符号、CLI 命令、配置项、文件路径四类引用 | 校验逻辑覆盖上述四类引用 |
| REQ-3-4 | docs-lint 发现不一致时输出具体位置与原因，退出码非 0 | 不一致时输出文件路径、行号、原因，退出码 1；一致时输出 "docs-lint: all references valid"，退出码 0 |

**对应阶段 Spec 验收标准**：
- AC-9a：docs-lint 校验范围仅限 docs/INDEX.md、docs/usage.md、docs/custom-rules.md、docs/api.md 四个活文档
- AC-9b：docs-lint 校验四类引用（代码符号、CLI 命令、配置项、文件路径），不一致时输出文件路径、行号、原因
- AC-9c：docs-lint 发现不一致时退出码非 0（退出码 1）

---

## 3. 实现步骤

### 3.1 创建 scripts/docs-lint.ts 文件

新建 `scripts/docs-lint.ts`，实现以下模块：

#### 3.1.1 接口定义

按设计文档 §4.1 定义接口：

```typescript
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

// 主入口：校验文档引用一致性
export function runDocsLint(options: DocsLintOptions): DocsLintResult;

// 测试辅助函数：直接暴露符号收集结果，供单元测试精确验证
export function collectExportedSymbols(projectRoot: string): {
  symbols: Set<string>;      // lib/ 下所有 export 的函数/类/类型/接口名
  cliCommands: Set<string>;  // cli.ts 中注册的子命令名
  cliOptions: Set<string>;   // cli.ts 中注册的选项名（含 .option 和 .requiredOption）
};
```

> **`collectExportedSymbols` 的作用**：`runDocsLint` 的白名单匹配策略导致行为测试无法区分"符号已收集"与"符号未收集被跳过"（两种情况 `result.valid` 均为 `true`）。`collectExportedSymbols` 直接暴露符号集合，使单元测试可精确验证 specifier 形式导出是否被正确收集。该函数不改变 `runDocsLint` 的公开接口与行为，仅作为测试辅助函数额外导出。

#### 3.1.2 源码符号收集

使用 Babel AST 解析 `lib/**/*.ts`，收集所有 export 的函数名、类名、类型名、接口名：

1. 遍历 `lib/**/*.ts`（含 `lib/api.ts`、`lib/code-quality/index.ts`、`lib/knowledge-graph/index.ts`、`lib/custom-rules/index.ts`、`lib/types.ts` 及子目录）
2. 使用 `@babel/parser` 解析为 AST（`plugins: ['typescript']`，`sourceType: 'module'`）
3. 使用 `@babel/traverse` 遍历以下导出节点形式：
   - `ExportNamedDeclaration`（声明形式）：`export function foo()`、`export class Foo`、`export type Bar`、`export interface Baz` — 从 `declaration` 中提取名称
   - `ExportNamedDeclaration`（specifier 形式）：`export { foo, bar }` 和 `export { foo } from './mod'` — 从 `specifiers` 数组中提取每个 `ExportSpecifier` 的 `exported.name`
   - `ExportDefaultDeclaration`：仅当 `declaration` 为 `FunctionDeclaration` 或 `ClassDeclaration` 且 `declaration.name` 非空时提取名称；`Identifier`（如 `export default rule;`）、`CallExpression`（如 `export default defineConfig({})`）等表达式形式的默认导出跳过（它们导出的是值而非具名声明）
4. 收集导出符号名集合（`Set<string>`）

> **Babel parser 配置**：`@babel/parser` 默认不支持 TypeScript 语法，必须显式启用 `plugins: ['typescript']`。
>
> **specifier 形式导出的重要性**：项目中 `lib/code-quality/index.ts` 使用 `export { loadLocalLLMConfig };`（本地再导出，无 `from` 子句），`lib/custom-rules/index.ts` 使用 `export { scanFiles, scanFile } from './scanner.js';` 和 `export { RULE_IMPLEMENTATIONS } from './rules/index.js';`（两条独立的 specifier 形式导出语句）。若仅处理声明形式，将遗漏这 4 个公开 API 符号，导致 docs-lint 误报。

#### 3.1.3 CLI 命令与选项收集

使用 Babel AST 解析 `cli.ts`，收集 commander 注册的子命令与选项：

1. 使用 `@babel/parser` 解析 `cli.ts` 为 AST（`plugins: ['typescript']`）
2. 使用 `@babel/traverse` 遍历 `CallExpression` 节点
3. 对 `.command('xxx')` 调用，提取第一个字符串参数作为子命令名
   - 收集结果：`shield`, `quality`, `report`, `api`, `graph`
4. 对 `.option('--xxx <type>', ...)` / `.option('--xxx', ...)` / `.requiredOption('--xxx <type>', ...)` / `.requiredOption('--xxx', ...)` 调用，提取第一个参数中的选项名
   - `.option` 与 `.requiredOption` 均注册为选项，仅是否必填的区别，选项名提取规则一致
   - 解析规则：从 `'--project <path>'` 提取 `project`，从 `'--no-body'` 提取 `body`
   - 对 `--no-` 前缀选项，commander 内部存储为去掉 `no-` 前缀的名称（如 `--no-body` 存储为 `body`）
   - 收集结果：`project`, `target`, `proxy-port`, `start-page`, `headless`, `body`, `insecure`, ...
5. 子命令集合与选项集合供文档侧 CLI 命令/配置项校验使用

#### 3.1.4 文档引用提取

用正则从 4 个活文档 markdown 提取四类引用：

| 引用类型 | 正则 | 说明 |
|---------|------|------|
| 代码符号 | `` /`([A-Z][a-zA-Z]\w*\|[a-z][a-zA-Z]\w*)`/g `` | 反引号包裹的标识符 |
| CLI 命令 | `/(?:node \.\/dist\/cli\.js\|legacy-shield\|pnpm)\s+(\w+)/g` | 支持多种调用方式 |
| 配置项 | `/--([a-z][a-z-]*)/g` | 仅在有效 CLI 命令上下文中校验 |
| 文件路径 | `/(lib\/[^\s`)]+\.ts\|docs\/[^\s`)]+\.md)/g` | lib/ 和 docs/ 下的文件路径 |

#### 3.1.5 比对与输出（白名单匹配策略）

校验语义为"文档引用了源码符号 → 源码必须存在"，而非"所有反引号内容 → 必须是源码符号"：

```
对每个文档中的每个引用：
  代码符号：
    若反引号内容在源码符号集合中 → 校验通过
    若反引号内容不在源码符号集合中 → 跳过（不报错，因为可能是普通英文单词）
  CLI 命令：
    若命令在 cli.ts 注册的子命令集合中 → 校验通过
    若命令不在集合中 → 跳过（不报错，因为可能是 pnpm/npm 通用命令如 build/test/exec）
  配置项：
    仅校验出现在有效 CLI 命令上下文（同一行或前一行有子命令集合内的 CLI 命令引用）的 --option
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

#### 3.1.6 误报控制

为降低误报，以下情况跳过校验：

- **代码符号白名单策略**：仅当反引号内容在源码符号集合中存在时才校验，不在集合中的反引号内容直接跳过。
- **CLI 命令白名单策略**：仅当提取到的命令在 cli.ts 注册的子命令集合中时才校验，不在集合中的命令直接跳过。
- **TypeScript 关键字 denylist**：对 `true`、`false`、`null`、`undefined`、`string`、`number`、`boolean`、`void`、`never`、`any`、`unknown` 等关键字/内置类型显式跳过。
- **反引号内容过滤**：含空格、含中文、为完整句子的反引号内容跳过。
- **配置项上下文校验**：`--option` 仅在有效 CLI 命令上下文（同一行或前一行有子命令集合内的 CLI 命令引用）中校验。
- **`--no-` 前缀处理**：对 `--no-<option>` 模式，提取 `<option>` 作为实际选项名校验。源码符号集合中存储去掉 `no-` 前缀的名称（如 `body` 而非 `no-body`），与文档侧提取保持一致。
- **配置项 `--` 后跟非字母时跳过**。

#### 3.1.7 CLI 入口

在 `scripts/docs-lint.ts` 末尾实现 CLI 入口：

```typescript
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

> **入口判定方式**：不能使用 `import.meta.url === \`file://${process.argv[1]}\`` 直接比较，因为 `process.argv[1]` 在通过 `node ./dist/scripts/docs-lint.js`（相对路径）执行时为相对路径，而 `import.meta.url` 为绝对 file:// URL，两者字符串不相等导致入口判定静默失败。必须使用 `fileURLToPath` 将 `import.meta.url` 转为绝对路径，再用 `resolve` 规范化 `process.argv[1]` 后比较。

### 3.2 更新 tsconfig.json

修改 `tsconfig.json` 的 `include`，增加 `scripts/**/*.ts`：

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

### 3.3 更新 tsconfig.test.json

同步修改 `tsconfig.test.json` 的 `include`，增加 `scripts/**/*.ts`（若 docs-lint 需单元测试）。

---

## 4. 测试计划

### 4.1 单元测试

通过 `collectExportedSymbols` 直接验证符号收集，通过 `runDocsLint` 验证端到端行为：

| 测试项 | 测试方式 | 预期结果 |
|-------|---------|---------|
| 源码符号收集（声明形式） | 调用 `collectExportedSymbols(projectRoot)`，检查 `symbols` 集合 | 集合包含 `startApiServer`、`runAll`、`runModule`、`runDiff`、`runWatch`、`createCLI`、`runKnowledgeGraph`、`runCustomRules` 等声明形式导出的符号 |
| 源码符号收集（specifier 形式） | 调用 `collectExportedSymbols(projectRoot)`，检查 `symbols` 集合 | 集合包含 `loadLocalLLMConfig`、`scanFiles`、`scanFile`、`RULE_IMPLEMENTATIONS` 等 specifier 形式导出的符号 |
| CLI 命令收集 | 调用 `collectExportedSymbols(projectRoot)`，检查 `cliCommands` 集合 | 集合包含 `shield`、`quality`、`report`、`api`、`graph` |
| CLI 选项收集（.requiredOption） | 调用 `collectExportedSymbols(projectRoot)`，检查 `cliOptions` 集合 | 集合包含 `project`（来自 `.requiredOption`） |
| CLI 选项收集（--no- 前缀） | 调用 `collectExportedSymbols(projectRoot)`，检查 `cliOptions` 集合 | 集合包含 `body`（来自 `--no-body`，commander 内部存储为 `body`） |
| 一致引用校验 | 构造测试文档引用源码中存在的符号，调用 `runDocsLint` | `result.valid === true`，`result.errors` 为空 |
| 不一致引用校验 | 构造测试文档引用源码中不存在的文件路径（如 `lib/nonexistent.ts`），调用 `runDocsLint` | `result.valid === false`，`result.errors` 含错误项，含 file、line、reference、type、reason |
| 白名单跳过验证 | 构造测试文档含不在符号集合中的反引号内容（如 `` `foobar` ``），调用 `runDocsLint` | `result.valid === true`（不在集合中，跳过不报错） |
| 退出码 | 执行 CLI 入口 | 一致时退出码 0，不一致时退出码 1 |

### 4.2 集成测试

| 测试项 | 测试方式 | 预期结果 |
|-------|---------|---------|
| 端到端校验 | `node ./dist/scripts/docs-lint.js` | 对当前 docs/ 下 4 个活文档执行校验，输出结果 |
| 白名单策略验证 | 检查 `pnpm build`、`pnpm test` 等 pnpm 通用命令不被误报 | 不在子命令集合中的命令被跳过，不报错 |
| `--no-` 前缀处理 | 检查 `--no-body` 提取为 `body` | 源码集合与文档提取一致 |
| 配置项上下文校验 | 检查 `pnpm install --save` 中的 `--save` 不被校验 | 非子命令上下文的 `--option` 被跳过 |

### 4.3 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（scripts/**/*.ts 已加入 include） |
| 构建 | `pnpm build` | 通过（dist/scripts/docs-lint.js 生成） |
| 测试 | `pnpm test` | 全量通过（不影响现有测试） |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| Babel parser 未启用 TypeScript 插件 | 解析 .ts 文件报错 | 显式配置 `plugins: ['typescript']` |
| CLI 入口判定失败 | `node ./dist/scripts/docs-lint.js` 执行无输出 | 使用 `fileURLToPath(import.meta.url) === resolve(process.argv[1])` 而非字符串直接比较 |
| 误报（反引号内容被误判为代码符号） | 校验结果不准确 | 白名单匹配策略：仅当反引号内容在源码符号集合中时才校验 |
| 误报（pnpm 通用命令被误判为 CLI 子命令） | `pnpm build`、`pnpm test` 被误报 | CLI 命令白名单策略：仅当命令在子命令集合中时才校验 |
| `--no-` 前缀选项处理不一致 | `--no-body` 校验失败 | 源码侧收集 `body`（commander 内部存储），文档侧提取 `body`（去掉 `no-` 前缀） |
| `.requiredOption()` 未被收集 | `--project` 校验失败 | 源码侧同时收集 `.option()` 和 `.requiredOption()` 调用 |
| tsconfig.json 未包含 scripts/ | docs-lint.ts 无法编译 | T10 步骤 3.2 同步修改 tsconfig.json |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `scripts/docs-lint.ts` | 新建 |
| `tsconfig.json` | 修改（include 增加 `scripts/**/*.ts`） |
| `tsconfig.test.json` | 修改（include 增加 `scripts/**/*.ts`） |

不修改 lib/ 下任何源码文件、不修改 cli.ts、不修改现有测试。

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，发现 3 个 P1 + 7 个 P2 问题
> - 第二轮评审（2026-06-24）：修改后通过，P1-1/P1-3 已修复，P1-2 部分修复（遗留测试有效性盲区），P2-1/P2-2 新发现
> - 第三轮评审（2026-06-24）：通过，遗留 P1 已修复（新增 collectExportedSymbols），P2-1/P2-2 已修复
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无（第一轮 P1-1/P1-2/P1-3 均已在第三轮修复闭环）

### P2 优化项

无（第一轮/第二轮 P2 已全部处理）
