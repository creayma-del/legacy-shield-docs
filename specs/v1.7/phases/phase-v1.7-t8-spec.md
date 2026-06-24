# 配置 docs:gen 生成

> 任务编号：T8
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.2.3、§5.4
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T8
> 依赖任务：T7
> 评审记录：见本文档末尾

---

## 1. 任务目标

在 `package.json` 的 `scripts` 中新增 `docs:gen` 命令，指向 TypeDoc（`typedoc`），执行 `pnpm docs:gen` 后在 `docs/api/` 目录下生成多文件 API 文档（按模块拆分）。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-2-3 | 配置 TypeDoc，生成产物输出到 `docs/api/` 目录（多文件，按模块拆分） | `pnpm docs:gen` 执行成功，`docs/api/` 下生成多文件 API 文档 |
| REQ-2-5 | 新增 npm script `docs:gen` 用于生成 API 文档 | `package.json` scripts 含 `docs:gen`，执行生成 docs/api/ |

**对应阶段 Spec 验收标准**：AC-7（pnpm docs:gen 执行成功，docs/api/ 下生成多文件 API 文档）

---

## 3. 实现步骤

### 3.1 前置条件确认

T6 已引入 TypeDoc（`typedoc ^0.28.18`）并创建 `typedoc.json`（含 5 个 entryPoints）。T7 已为 `lib/` 下公开 API 添加 TSDoc 注释。执行以下命令确认：

```bash
# 确认 typedoc 已安装
cat package.json | grep typedoc
# 预期：devDependencies 含 "typedoc": "^0.28.18"

# 确认 typedoc.json 存在且 entryPoints 正确
cat typedoc.json
# 预期：entryPoints 含 lib/api.ts、lib/code-quality/index.ts、lib/knowledge-graph/index.ts、lib/custom-rules/index.ts、lib/types.ts

# 确认 T7 已添加 TSDoc（抽查）
head -20 lib/api.ts
# 预期：startApiServer 上方有 /** ... */ TSDoc 注释
```

### 3.2 修改 package.json scripts

在 `package.json` 的 `scripts` 对象中新增 `docs:gen` 条目：

**新增条目**：

```json
"docs:gen": "typedoc"
```

**注意**：仅新增 `docs:gen` 一行，不修改其他已有 script。若 T11（配置 docs:lint）已先行执行并新增了 `docs:lint`，T8 执行时保留 `docs:lint` 不变，仅追加 `docs:gen`。若 T3（配置 pnpm workspace）已先行执行并修改了 devDependencies，T8 执行时保留 devDependencies 变更不变，仅修改 scripts。

### 3.3 执行 pnpm docs:gen

```bash
pnpm docs:gen
```

预期：TypeDoc 读取 `typedoc.json` 配置，解析 5 个 entryPoints 的 TSDoc 注释，在 `docs/api/` 目录下生成多文件 API 文档。

### 3.4 验证生成产物

```bash
# 检查 docs/api/ 目录存在
ls docs/api/
# 预期：含 index.html 及按模块拆分的 HTML 文件

# 检查生成产物覆盖全部 5 个 entryPoints
ls docs/api/classes/
ls docs/api/modules/
# 预期：含 startApiServer、runAll、runModule、runDiff、runWatch、loadLocalLLMConfig、createCLI、runKnowledgeGraph、runCustomRules、scanFiles、scanFile、RULE_IMPLEMENTATIONS 等 API

# 检查 re-export TSDoc 是否被拾取
grep -r "loadLocalLLMConfig" docs/api/
grep -rw "scanFiles" docs/api/
grep -rw "scanFile" docs/api/
grep -r "RULE_IMPLEMENTATIONS" docs/api/
# 预期：生成产物中含这 4 个 re-export API 的描述
# 注：scanFiles 和 scanFile 使用 -w（word boundary）避免互相匹配
```

> **re-export TSDoc 拾取问题**：若 TypeDoc 未正确拾取 re-export 语句上方的 TSDoc（`loadLocalLLMConfig`、`scanFiles`、`scanFile`、`RULE_IMPLEMENTATIONS`），需在原始定义文件中补全 TSDoc（T7 §6 变更范围已列出可能涉及的 3 个补充文件）。T8 验证时若发现 re-export API 缺失，通知 T7 补全。

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| package.json scripts 含 docs:gen | 读取 `package.json` 的 `scripts.docs:gen` 字段 | 值为 `typedoc` |
| pnpm docs:gen 执行成功 | 执行 `pnpm docs:gen` | 无错误，退出码 0 |
| docs/api/ 目录生成 | `ls docs/api/` | 目录存在，含 HTML 文件 |
| 生成产物覆盖全部 entryPoints | 检查 docs/api/ 产物 | 含 5 个模块的 API 文档 |
| re-export TSDoc 被拾取 | `grep -rw "scanFiles" docs/api/` 和 `grep -rw "scanFile" docs/api/` 等 | 4 个 re-export API（loadLocalLLMConfig、scanFiles、scanFile、RULE_IMPLEMENTATIONS）均有描述 |
| 现有 scripts 未被修改 | `git diff package.json` | 仅新增 `docs:gen` 一行 |

### 4.2 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（T8 不涉及 TypeScript 文件变更） |
| 构建 | `pnpm build` | 通过 |
| 测试 | `pnpm test` | 全量通过（T8 不修改任何测试） |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T6 未完成（TypeDoc 未引入） | docs:gen 无法执行 | T8 依赖 T6，执行前确认 typedoc 已安装 |
| T7 未完成（TSDoc 未添加） | 生成产物为空或内容不完整 | T8 依赖 T7，执行前确认 TSDoc 已添加 |
| TypeDoc 生成失败 | docs:gen 报错 | 检查 typedoc.json 配置；检查 TSDoc 语法；T6 已做 TypeDoc-TS smoke test |
| re-export TSDoc 未被拾取 | 生成产物缺失 4 个 API | 在原始定义文件中补全 TSDoc（T7 已预案） |
| TypeDoc 版本不兼容 TypeScript 5.9.3 | 生成报错 | T6 已确保 typedoc ^0.28.18 兼容 |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `package.json` | 修改（scripts 新增 `docs:gen`） |
| `docs/api/` | 新建（TypeDoc 生成产物） |
| `pnpm-lock.yaml` | 可能修改（若 docs:gen 引入新依赖，通常不会） |

不修改 `typedoc.json`（T6 已完成）、不修改 `lib/` 下任何源码文件（T7 已完成）、不修改现有测试。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：通过（第二轮）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：re-export TSDoc 拾取验证遗漏 scanFile → 已新增 grep -rw "scanFile" 命令，scanFiles 同步改为 -rw

第二轮评审：无新增 P0/P1/P2 问题。

### P2 优化项

无。
