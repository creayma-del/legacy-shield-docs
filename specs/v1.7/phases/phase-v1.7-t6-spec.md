# 引入 TypeDoc

> 任务编号：T6
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.2.2
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T6
> 依赖任务：无
> 评审记录：见本文档末尾

---

## 1. 任务目标

引入 TypeDoc `^0.28.18` 作为 devDependency，创建 `typedoc.json` 配置文件，配置 5 个入口文件（`lib/api.ts`、`lib/code-quality/index.ts`、`lib/knowledge-graph/index.ts`、`lib/custom-rules/index.ts`、`lib/types.ts`），输出目录为 `docs/api/`，为 T7 TSDoc 迁移和 T8 docs:gen 生成做好准备。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-2-1 | 引入 TypeDoc 作为 devDependency | `package.json` devDependencies 含 `typedoc`（版本 `^0.28.18`），`pnpm install` 成功，`typedoc.json` 配置正确 |

**对应阶段 Spec 验收标准**：AC-5（TypeDoc ^0.28.18 引入，typedoc.json 配置正确）

---

## 3. 实现步骤

### 3.1 安装 TypeDoc 依赖

在主仓库执行：

```bash
pnpm add -D typedoc@^0.28.18
```

> **版本要求说明**：TypeDoc 0.27.x 及 0.28.0-0.28.17 仅支持 TypeScript 5.0-5.8；0.28.18（2026-03-23）起新增 TypeScript 6.0 支持，按 TypeDoc 支持最新两个 TS 版本的策略兼容 5.9。本项目使用 TypeScript 5.9.3，必须使用 `^0.28.18`。

### 3.2 创建 typedoc.json

在主仓库根目录创建 `typedoc.json`：

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

**配置说明**：

| 字段 | 值 | 说明 |
|------|-----|------|
| entryPoints | 5 个 lib/ 入口文件 | 覆盖设计文档 §2.2.1 全部公开 API |
| out | `docs/api` | 生成产物输出目录 |
| name | `legacy-shield API` | 文档标题 |
| readme | `none` | 不生成 README（api.md 已作为入口索引） |
| excludePrivate | `true` | 排除 private 成员 |
| excludeInternal | `true` | 排除 @internal 标注成员 |
| tsconfig | `tsconfig.json` | 复用项目 TypeScript 配置 |

### 3.3 验证安装

执行 `pnpm install` 确认依赖安装成功，无版本冲突。

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| typedoc 在 devDependencies 中 | 读取 `package.json` devDependencies | 含 `typedoc: "^0.28.18"` |
| pnpm install 成功 | 执行 `pnpm install` | 无错误 |
| typedoc.json 存在 | `ls typedoc.json` | 文件存在 |
| typedoc.json 配置正确 | 读取 `typedoc.json` | entryPoints 含 5 个入口，out 为 `docs/api` |
| entryPoints 文件存在 | 检查 5 个入口文件路径 | `lib/api.ts`、`lib/code-quality/index.ts`、`lib/knowledge-graph/index.ts`、`lib/custom-rules/index.ts`、`lib/types.ts` 均存在 |
| typedoc 可执行 | 执行 `pnpm exec typedoc --version` | 输出版本号 |

### 4.2 集成测试

本任务暂不执行 `pnpm docs:gen`（docs:gen script 在 T8 配置），仅验证 TypeDoc 安装与配置就绪。T8 将验证完整生成流程。

**TypeDoc-TS 兼容性冒烟测试**：

执行轻量级冒烟测试，验证 TypeDoc 能成功解析项目 TS 5.9.3 源码：

```bash
pnpm exec typedoc --entryPoints lib/types.ts --out /tmp/typedoc-smoke-test --tsconfig tsconfig.json
```

验证 `/tmp/typedoc-smoke-test/` 下生成产物，确认 TypeDoc 与 TypeScript 5.9.3 兼容。测试后清理临时目录：

```bash
rm -rf /tmp/typedoc-smoke-test
```

### 4.3 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（typedoc.json 不影响类型检查） |
| 构建 | `pnpm build` | 通过（typedoc 仅为 devDependency） |
| 测试 | `pnpm test` | 全量通过（不影响现有测试） |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| TypeDoc ^0.28.18 与 TypeScript 5.9.3 不兼容 | docs:gen 失败 | T6 验证 `pnpm exec typedoc --version` 与 TS 版本兼容性；若不兼容，查阅 TypeDoc changelog 确认最低兼容版本 |
| typedoc.json entryPoints 路径错误 | docs:gen 找不到入口文件 | T6 验证 5 个入口文件路径均存在 |
| typedoc.json tsconfig 字段指向错误 | docs:gen 使用错误 TS 配置 | 确认 tsconfig 字段值为 `tsconfig.json`（项目根目录） |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `package.json` | 修改（devDependencies 增加 typedoc） |
| `pnpm-lock.yaml` | 修改（新增 typedoc 依赖锁定记录） |
| `typedoc.json` | 新建 |

不修改 lib/ 下任何源码文件（TSDoc 迁移在 T7 执行）。

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，无 P1，发现 4 个 P2 问题
> - 第二轮评审（2026-06-24）：通过，P2-1/P2-3/P2-4 已修复，P2-2 说明合理
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无

### P2 优化项

无（第一轮 P2 已全部处理）
