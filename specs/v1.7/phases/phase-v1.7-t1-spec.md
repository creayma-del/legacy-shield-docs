# 创建归档仓库

> 任务编号：T1
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.1.1
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T1
> 依赖任务：无
> 评审记录：见本文档末尾

---

## 1. 任务目标

创建独立归档仓库 `legacy-shield-docs`（位于主仓库同级目录 `../legacy-shield-docs`），初始化 `package.json`（name 为 `@legacy-shield/docs`）、`README.md`，并创建空的 `specs/` 目录，为 T2 迁移历史 spec 做好准备。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-1-1 | 创建独立归档仓库 `legacy-shield-docs`，`package.json` name 为 `@legacy-shield/docs` | 归档仓库存在，含 `package.json`（name 字段为 `@legacy-shield/docs`，private: true），含 `README.md`，含空 `specs/` 目录 |

**对应阶段 Spec 验收标准**：AC-1（归档仓库 `legacy-shield-docs` 存在，package.json name 为 `@legacy-shield/docs`）

---

## 3. 实现步骤

### 3.1 创建归档仓库目录

在主仓库同级目录创建 `legacy-shield-docs/` 目录：

```bash
mkdir -p ../legacy-shield-docs
```

### 3.2 初始化 package.json

创建 `../legacy-shield-docs/package.json`：

```json
{
  "name": "@legacy-shield/docs",
  "version": "1.0.0",
  "private": true,
  "description": "legacy-shield 历史 spec 归档仓库"
}
```

### 3.3 创建 README.md

创建 `../legacy-shield-docs/README.md`，说明归档仓库的用途、内容、与主仓库的关系：

- 归档仓库用途：存放 legacy-shield 项目 v1.1~v1.6 及早期历史 spec 文档
- 引入方式：主仓库通过 pnpm workspace 以 `@legacy-shield/docs` 包名引入
- 访问路径：主仓库中通过 `node_modules/@legacy-shield/docs/specs/` 访问
- 冻结策略：归档文档已冻结，不再修改

### 3.4 创建空 specs 目录

创建 `../legacy-shield-docs/specs/` 空目录（T2 将迁移历史 spec 内容到此目录）：

```bash
mkdir -p ../legacy-shield-docs/specs
```

> **注意**：仅创建 `specs/` 空目录，**不创建** common/、phases/、v1.1/~v1.6/ 等子目录。子目录由 T2 迁移时通过 `cp -r` 自动创建，预先创建会导致 `cp -r` 产生嵌套目录（如 `specs/v1.1/v1.1/`）。

### 3.5 初始化 git 仓库

在归档仓库初始化 git：

```bash
cd ../legacy-shield-docs && git init
```

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| 归档仓库目录存在 | `ls ../legacy-shield-docs/` | 目录存在 |
| package.json name 正确 | 读取 `../legacy-shield-docs/package.json` 的 name 字段 | `@legacy-shield/docs` |
| package.json version 正确 | 读取 `../legacy-shield-docs/package.json` 的 version 字段 | `1.0.0` |
| package.json private 正确 | 读取 `../legacy-shield-docs/package.json` 的 private 字段 | `true` |
| README.md 存在 | `ls ../legacy-shield-docs/README.md` | 文件存在 |
| specs/ 目录存在 | `ls -d ../legacy-shield-docs/specs/` | 目录存在 |
| specs/ 目录为空 | `ls ../legacy-shield-docs/specs/` | 无内容（T2 迁移后填充） |
| git 仓库已初始化 | `cd ../legacy-shield-docs && git status` | 退出码 0，显示分支信息 |

### 4.2 集成测试

无（本任务为仓库初始化，无代码逻辑）。

### 4.3 回归测试

无（本任务不修改主仓库任何文件，不影响现有功能）。

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| 归档仓库路径不在主仓库同级目录 | T3 pnpm workspace 配置失败 | T1 执行时确认目录路径为 `../legacy-shield-docs` |
| 归档仓库 git 未初始化 | T2 迁移时无法 commit | T1 步骤 3.5 初始化 git |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `../legacy-shield-docs/package.json` | 新建 |
| `../legacy-shield-docs/README.md` | 新建 |
| `../legacy-shield-docs/specs/` | 新建（空目录） |

不修改主仓库任何文件。

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，发现 1 个 P1 + 5 个 P2 问题
> - 第二轮评审（2026-06-24）：通过，P1 已修复，P2-1/P2-2/P2-4 已修复，P2-3/P2-5 说明合理
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无（第一轮 P1-1 已在第二轮修复闭环）

### P2 优化项

无（第一轮 P2 已全部处理）
