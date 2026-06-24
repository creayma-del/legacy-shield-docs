# 改写 api.md 为入口索引

> 任务编号：T9
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.2.4
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T9
> 依赖任务：T8
> 评审记录：见本文档末尾

---

## 1. 任务目标

原 `docs/api.md`（手写 REST API + graph 子命令文档）改为入口索引，指向 TypeDoc 生成的 `docs/api/` 目录。改写后 api.md 包含在线文档链接、更新方式、源码位置说明。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-2-4 | 原 `docs/api.md` 改为入口索引，指向 `docs/api/` 目录 | api.md 内容改为索引说明，指向 docs/api/ |

**对应阶段 Spec 验收标准**：AC-8（api.md 改为入口索引，指向 docs/api/）

---

## 3. 实现步骤

### 3.1 前置条件确认

T8 已完成 docs:gen 配置并成功生成 `docs/api/` 目录。执行以下命令确认（仅验证产物存在，不重跑 docs:gen 以避免修改 docs/api/）：

```bash
# 确认 docs/api/ 目录存在
ls docs/api/
# 预期：含 index.html 及按模块拆分的 HTML 文件
```

### 3.2 改写 docs/api.md

将 `docs/api.md` 原有内容（手写 REST API 端点详解 + graph 子命令文档）替换为入口索引内容。

**改写前**：api.md 含手写的 REST API 端点详解（/health、/logs、/report、/errors/top、/timeline、/suggest）、graph 子命令文档、参数说明、响应示例等。

**改写后**：api.md 仅包含入口索引说明，指向 `docs/api/` 目录。具体内容按设计文档 §2.2.4：

1. 标题与简介：说明 API 文档由 TypeDoc 从源码 TSDoc 自动生成
2. 在线文档链接：指向 `./api/index.html`
3. 更新方式：`pnpm docs:gen` 命令
4. 源码位置：列出 5 个源码文件路径（lib/api.ts、lib/code-quality/index.ts、lib/knowledge-graph/index.ts、lib/custom-rules/index.ts、cli.ts）

改写后 api.md 的完整内容（来自设计文档 §2.2.4）：

````markdown
# legacy-shield API 文档

API 文档由 TypeDoc 从源码 TSDoc 自动生成（HTML 格式）。

## 在线文档

- [API 文档首页](./api/index.html)（浏览器打开）

## 更新方式

```bash
pnpm docs:gen
```

## 源码位置

- REST API：lib/api.ts
- Code Quality：lib/code-quality/index.ts
- Knowledge Graph：lib/knowledge-graph/index.ts
- Custom Rules：lib/custom-rules/index.ts
- CLI 入口：cli.ts（CLI 文档见 docs/usage.md）
````

> **注意**：原 api.md 中的端点详解内容已在 T7 中迁移为源码 TSDoc 注释，改写后不再保留手写内容。cli.ts 的 CLI 文档保留在 `docs/usage.md` 中手写（设计文档 §2.2.5），api.md 仅提及 cli.ts 的源码位置。

### 3.3 验证 api.md 改写结果

```bash
# 检查 api.md 内容
cat docs/api.md
# 预期：含 API 文档首页链接、更新方式、源码位置

# 检查 api.md 不再含手写端点详解
grep -c "GET /health" docs/api.md
# 预期：0（手写端点详解已移除）

# 检查 api.md 指向 docs/api/
grep "api/index.html" docs/api.md
# 预期：含指向 ./api/index.html 的链接
```

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| api.md 改为入口索引 | 读取 docs/api.md | 内容为索引说明，含在线文档链接、更新方式、源码位置 |
| api.md 指向 docs/api/ | `grep "api/index.html" docs/api.md` | 含指向 ./api/index.html 的链接 |
| api.md 不再含手写端点详解 | `grep -c "GET /health" docs/api.md` | 0（手写内容已移除） |
| api.md 含更新方式 | `grep "docs:gen" docs/api.md` | 含 pnpm docs:gen 命令 |
| api.md 含源码位置 | `grep "lib/api.ts" docs/api.md` | 含 5 个源码文件路径 |
| docs/api/ 目录存在 | `ls docs/api/` | 目录存在，含 TypeDoc 生成产物 |

### 4.2 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（T9 不涉及 TypeScript 文件变更） |
| 构建 | `pnpm build` | 通过 |
| 测试 | `pnpm test` | 全量通过（T9 不修改任何测试） |

> **注**：`pnpm docs:lint` 不在 T9 回归测试范围内，docs:lint 归属 T10（实现脚本）/T11（配置 script）/T13（全量回归）。T9 仅改写 api.md 文档，不影响代码编译与测试。

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T8 未完成（docs/api/ 未生成） | api.md 指向不存在的目录 | T9 依赖 T8，执行前确认 docs/api/ 已生成 |
| api.md 改写遗漏原内容 | 部分 API 文档丢失 | T7 已将原 api.md 内容迁移为 TSDoc，T9 改写后通过 TypeDoc 生成 |
| docs-lint 误报 api.md 引用 | pnpm docs:lint 失败 | api.md 中的源码路径（lib/api.ts 等）均真实存在，不会误报 |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `docs/api.md` | 改写（从手写 API 文档改为入口索引） |

不修改 `docs/api/`（T8 生成产物）、不修改 `lib/` 下任何源码文件（T7 已完成）、不修改现有测试。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：通过（第二轮）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：回归测试包含尚未就绪的 docs:lint → 已移除并添加注释
- P1-2：前置条件重跑 docs:gen 与"不修改 docs/api/"矛盾 → 已移除重跑命令

第二轮评审：无新增 P0/P1 问题。

### P2 优化项

无（2 个 P2 验证覆盖优化项可在执行阶段顺带优化，不阻塞通过）。
