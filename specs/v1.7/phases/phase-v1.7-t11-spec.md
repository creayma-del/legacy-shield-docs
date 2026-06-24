# 配置 docs:lint script

> 任务编号：T11
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.3.1、§5.4
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T11
> 依赖任务：T10
> 评审记录：见本文档末尾

---

## 1. 任务目标

在 `package.json` 的 `scripts` 中新增 `docs:lint` 命令，指向 T10 实现并编译的 `dist/scripts/docs-lint.js`，使 `pnpm docs:lint` 可执行文档引用一致性校验，退出码正确反映校验结果（一致时退出码 0，不一致时退出码 1）。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-3-1 | 新增独立 docs-lint 脚本，作为独立 npm script `docs:lint` | `package.json` scripts 含 `docs:lint`，值为 `node ./dist/scripts/docs-lint.js`，`pnpm docs:lint` 可执行并输出校验结果，退出码正确（一致时 0，不一致时 1） |

**对应阶段 Spec 验收标准**：AC-10（pnpm docs:lint 可执行，退出码正确）

---

## 3. 实现步骤

### 3.1 前置条件确认

T10 已完成以下工作，T11 在此基础上配置 npm script：

- `scripts/docs-lint.ts` 已实现（含 `runDocsLint` 函数与 CLI 入口）
- `tsconfig.json` 和 `tsconfig.test.json` 的 include 已增加 `scripts/**/*.ts`
- `pnpm build` 能成功生成 `dist/scripts/docs-lint.js`

执行以下命令确认前置条件满足：

```bash
pnpm build
ls dist/scripts/docs-lint.js
```

> **注意**：若 `dist/scripts/docs-lint.js` 不存在，说明 T10 未完成或 `pnpm build` 未执行，需先回到 T10 完成脚本实现与编译。

### 3.2 修改 package.json scripts

在 `package.json` 的 `scripts` 对象中新增 `docs:lint` 条目。

**当前 `package.json` scripts（T11 执行前）**：

```json
{
  "scripts": {
    "build": "tsc && node scripts/strip-inject-export.js",
    "dev": "tsc --watch",
    "clean": "node -e \"import('node:fs').then(fs => fs.rmSync('dist', { recursive: true, force: true }))\"",
    "typecheck": "tsc --noEmit && tsc -p tsconfig.test.json --noEmit",
    "shield": "node ./dist/cli.js shield",
    "quality": "node ./dist/cli.js quality",
    "report": "node ./dist/cli.js report",
    "api": "node ./dist/cli.js api",
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

**新增条目**：

```json
"docs:lint": "node ./dist/scripts/docs-lint.js"
```

**修改后 `package.json` scripts（T11 执行后）**：

```json
{
  "scripts": {
    "build": "tsc && node scripts/strip-inject-export.js",
    "dev": "tsc --watch",
    "clean": "node -e \"import('node:fs').then(fs => fs.rmSync('dist', { recursive: true, force: true }))\"",
    "typecheck": "tsc --noEmit && tsc -p tsconfig.test.json --noEmit",
    "shield": "node ./dist/cli.js shield",
    "quality": "node ./dist/cli.js quality",
    "report": "node ./dist/cli.js report",
    "api": "node ./dist/cli.js api",
    "test": "vitest run",
    "test:watch": "vitest",
    "docs:lint": "node ./dist/scripts/docs-lint.js"
  }
}
```

> **注意**：仅新增 `docs:lint` 一行，不修改其他已有 script。若 T8（配置 docs:gen 生成）已先行执行并在 scripts 中新增了 `docs:gen`，T11 执行时保留 `docs:gen` 不变，仅追加 `docs:lint`。

### 3.3 验证 docs:lint 可执行

执行以下命令验证 `pnpm docs:lint` 可正常运行：

```bash
pnpm build
pnpm docs:lint
```

预期输出：

- 文档引用一致时：输出 `docs-lint: all references valid`，退出码 0
- 文档引用不一致时：输出不一致项详情（文件路径、行号、引用内容、类型、原因），退出码 1

> **退出码查看**：使用 `pnpm docs:lint; echo $?` 查看退出码。退出码 0 表示校验通过，退出码 1 表示存在不一致项。

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| package.json scripts 含 docs:lint | 读取 `package.json` 的 `scripts.docs:lint` 字段 | 值为 `node ./dist/scripts/docs-lint.js` |
| dist/scripts/docs-lint.js 存在 | `pnpm build` 后执行 `ls dist/scripts/docs-lint.js` | 文件存在 |
| pnpm docs:lint 可执行 | 执行 `pnpm docs:lint` | 命令执行，输出校验结果 |
| 退出码正确（一致时） | 执行 `pnpm docs:lint; echo $?` | 退出码 0，输出 `docs-lint: all references valid` |
| 退出码正确（不一致时） | 临时在活文档（如 `docs/usage.md`）中引入不存在的文件路径引用（如 `` `lib/nonexistent.ts` ``），执行 `pnpm docs:lint; echo $?`，验证后执行 `git checkout docs/usage.md` 恢复文档原内容 | 退出码 1，输出不一致项详情（含文件路径、行号、引用内容、类型、原因） |
| 现有 scripts 未被修改 | 执行 `git diff package.json` | 仅新增 `docs:lint` 一行，其余 scripts 值不变 |

### 4.2 集成测试

| 测试项 | 测试方式 | 预期结果 |
|-------|---------|---------|
| 端到端校验 | `pnpm build && pnpm docs:lint` | 先编译生成 `dist/scripts/docs-lint.js`，再执行校验，输出结果 |
| 退出码传递 | `pnpm docs:lint && echo "PASS" \|\| echo "FAIL"` | 一致时输出 `PASS`，不一致时输出 `FAIL` |
| 构建-校验链路 | `pnpm build && pnpm docs:lint && pnpm test` | 三步依次执行；docs:lint 退出码 0 时继续执行 test，退出码 1 时中断链路 |

### 4.3 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（T11 不涉及 TypeScript 文件变更） |
| 构建 | `pnpm build` | 通过（`dist/scripts/docs-lint.js` 生成） |
| 测试 | `pnpm test` | 全量通过（T11 不修改任何测试） |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T10 未完成 | `dist/scripts/docs-lint.js` 不存在，`pnpm docs:lint` 报错 `Cannot find module` | T11 依赖 T10，执行前确认 T10 已完成且 `pnpm build` 成功生成 `dist/scripts/docs-lint.js` |
| `pnpm build` 未执行 | `dist/scripts/docs-lint.js` 不存在 | 执行 `pnpm docs:lint` 前先执行 `pnpm build` |
| docs-lint 校验发现不一致 | 退出码 1，`pnpm docs:lint` 失败 | 修正活文档中不一致的引用（代码符号、CLI 命令、配置项、文件路径），使引用与源码一致 |
| package.json JSON 格式错误 | pnpm 无法解析 scripts | 仅新增一行 `"docs:lint": "node ./dist/scripts/docs-lint.js"`，不破坏现有 JSON 结构，修改后验证 `pnpm docs:lint` 可执行 |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `package.json` | 修改（scripts 新增 `docs:lint`） |

不修改 `scripts/docs-lint.ts`（T10 已完成）、不修改 `tsconfig.json` / `tsconfig.test.json`（T10 已完成）、不修改 `lib/` 下任何源码文件、不修改现有测试。

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：通过，发现 3 个 P2 优化项（已全部修复）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无

### P2 优化项

无（第一轮 P2-1 验收标准补充退出码要求、P2-2 新增现有 scripts 未被修改验证项、P2-3 退出码 1 测试清理方法明确为 git checkout，均已修复闭环）
