# 配置 pnpm workspace

> 任务编号：T3
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.1.2、§5.3
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T3
> 依赖任务：T2
> 评审记录：见本文档末尾

---

## 1. 任务目标

主仓库新增 `pnpm-workspace.yaml` 引入归档仓库 `legacy-shield-docs`（位于主仓库同级目录 `../legacy-shield-docs`），`package.json` 增加 `@legacy-shield/docs` 依赖，执行 `pnpm install` 验证成功，归档仓库内容可通过 `node_modules/@legacy-shield/docs/specs/` 访问。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-1-3 | 主仓库通过 pnpm workspace 引入 `@legacy-shield/docs`，workspace 路径为 `../legacy-shield-docs` | `pnpm-workspace.yaml` 含该路径，`package.json` 含该依赖，`pnpm install` 成功 |

**对应阶段 Spec 验收标准**：AC-3（pnpm workspace 配置正确，pnpm install 成功）

---

## 3. 实现步骤

### 3.1 前置条件确认

T2 已完成历史 spec 迁移，归档仓库 `../legacy-shield-docs` 存在且含 `specs/` 目录与 `package.json`（name: `@legacy-shield/docs`）。执行以下命令确认：

```bash
ls ../legacy-shield-docs/package.json
cat ../legacy-shield-docs/package.json | grep '"name"'
# 预期：@legacy-shield/docs
ls ../legacy-shield-docs/specs/
# 预期：v1.1 v1.2 v1.3 v1.4 v1.5 v1.6 common phases design.md execution-plan.md requirements.md acceptance-report.md
```

### 3.2 创建 pnpm-workspace.yaml

在主仓库根目录新建 `pnpm-workspace.yaml`：

```yaml
packages:
  - '../legacy-shield-docs'
```

### 3.3 修改 package.json

在 `package.json` 的 `devDependencies` 中增加 `@legacy-shield/docs` 依赖：

```json
{
  "devDependencies": {
    "@legacy-shield/docs": "workspace:*"
  }
}
```

> **注意**：仅新增 `@legacy-shield/docs` 一行，不修改现有 devDependencies 中其他依赖。若 T6（引入 TypeDoc）已先行执行并新增了 `typedoc`，T3 执行时保留 `typedoc` 不变，仅追加 `@legacy-shield/docs`。

### 3.4 执行 pnpm install

```bash
pnpm install
```

预期：pnpm install 成功，无依赖冲突。

### 3.5 验证归档仓库可访问

```bash
ls node_modules/@legacy-shield/docs/specs/
# 预期：v1.1 v1.2 v1.3 v1.4 v1.5 v1.6 common phases design.md execution-plan.md requirements.md acceptance-report.md
```

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| pnpm-workspace.yaml 存在 | `cat pnpm-workspace.yaml` | 内容为 YAML 格式 `packages:\n  - '../legacy-shield-docs'` |
| package.json 含 @legacy-shield/docs | `cat package.json \| grep @legacy-shield/docs` | devDependencies 含 `"@legacy-shield/docs": "workspace:*"` |
| pnpm install 成功 | `pnpm install` | 无错误，退出码 0 |
| node_modules/@legacy-shield/docs 可访问 | `ls node_modules/@legacy-shield/docs/specs/` | 含 v1.1~v1.6、common、phases 等目录 |
| 现有 devDependencies 未被修改（T6 未先行场景） | `git diff package.json` | 仅新增 `@legacy-shield/docs` 一行，其余 devDependencies 不变 |
| 现有 devDependencies 未被修改（T6 先行场景） | `git diff package.json` | 新增 `@legacy-shield/docs` 和 `typedoc` 两行（T6 新增），其余 devDependencies 不变 |

### 4.2 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过 |
| 构建 | `pnpm build` | 通过 |
| 测试 | `pnpm test` | 全量通过 |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T2 未完成（归档仓库未迁移） | workspace 引入失败 | T3 依赖 T2，执行前确认 T2 已完成 |
| pnpm workspace 与现有依赖冲突 | pnpm install 失败 | 验证依赖解析；失败时按 §5.1 回滚步骤回滚 |
| 归档仓库 package.json name 不匹配 | workspace 引用找不到包 | T1 已确保 name 为 @legacy-shield/docs |
| 归档仓库路径不正确 | pnpm install 找不到 ../legacy-shield-docs | T1 已确保路径为 ../legacy-shield-docs |

### 5.1 回滚步骤

若 pnpm workspace 引入失败或导致依赖冲突，按以下步骤回滚：

1. 删除 `pnpm-workspace.yaml`：`rm pnpm-workspace.yaml`
2. 从 `package.json` 的 `devDependencies` 中移除 `@legacy-shield/docs` 条目（保留 T6 先行新增的 `typedoc` 不变）
3. 重新执行 `pnpm install` 恢复 `node_modules` 状态（pnpm 会自动清理 workspace 软链并更新 `pnpm-lock.yaml`）
4. 验证 `pnpm typecheck`、`pnpm build`、`pnpm test` 恢复正常
5. 若归档仓库 `../legacy-shield-docs` 仍需保留（T1/T2 产物），不删除归档仓库；仅回滚主仓库的 workspace 引入配置
6. 使用 `git checkout pnpm-lock.yaml` 恢复 lock 文件（若 pnpm install 已修改）

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `pnpm-workspace.yaml` | 新建 |
| `package.json` | 修改（devDependencies 新增 @legacy-shield/docs） |
| `pnpm-lock.yaml` | 修改（pnpm install 自动更新） |

不修改任何源码文件、不修改现有测试、不修改 cli.ts。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：修改后通过（第二轮）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：缺少独立回滚步骤章节 → 已新增 §5.1 回滚步骤
- P1-2：§4.1 验证项预期与 §3.3 注意事项在 T6 并行场景下不一致 → 已拆分为两个验证项

第二轮评审：无新增 P1 问题。

### P2 优化项

第二轮 P2 问题（已修复）：
- P2-1：§4.1 验证项 pnpm-workspace.yaml 预期结果格式与 §3.2 YAML 格式不一致 → 已统一为 YAML 格式
- P2-2：§5.1 回滚步骤未显式提及 pnpm-lock.yaml → 已新增第 6 步恢复 lock 文件
- P2-3：文档状态未更新为"评审中" → 已更新评审记录
