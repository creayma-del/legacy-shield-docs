# 全量回归测试

> 任务编号：T13
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md)（整体）
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T13
> 依赖任务：T4, T5, T9, T11, T12
> 评审记录：见本文档末尾

---

## 1. 任务目标

执行全量回归测试，确保 v1.7 全部变更（分层归档 + API 文档代码化 + 契约校验）不破坏现有功能。验证 `pnpm typecheck`、`pnpm build`、`pnpm test` 全部通过，`pnpm docs:gen` 生成成功，`pnpm docs:lint` 校验通过，`docs/` 目录结构符合预期。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-4-1 | 迁移过程不破坏现有构建、测试、typecheck | `pnpm typecheck`、`pnpm build`、`pnpm test` 全部通过 |

**对应阶段 Spec 验收标准**：

- AC-13（`pnpm typecheck` 通过）
- AC-14（`pnpm build` 通过）
- AC-15（`pnpm test` 全量通过）
- AC-16（`pnpm docs:gen` 生成成功）
- AC-17（`pnpm docs:lint` 校验通过）
- AC-18（`docs/` 目录结构符合预期）

---

## 3. 实现步骤

### 3.1 前置条件确认

T4（INDEX.md）、T5（spec-guardian 评估）、T9（api.md 改写）、T11（docs:lint 配置）、T12（测试路径评估）均已完成。执行以下命令确认：

```bash
# 确认 INDEX.md 存在
ls docs/INDEX.md

# 确认 INDEX.md 内容完整性（AC-4）
grep "路径映射表" docs/INDEX.md
grep -i "onboarding" docs/INDEX.md

# 确认 T5 评估文档存在
ls docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md

# 确认 T12 评估文档存在
ls docs/specs/v1.7/evaluations/t12-test-path-evaluation.md

# 确认 api.md 已改写
grep "api/index.html" docs/api.md

# 确认 docs/api/ 已生成
ls docs/api/index.html

# 确认 pnpm-workspace.yaml 存在
cat pnpm-workspace.yaml

# 确认 package.json 含 docs:gen 和 docs:lint
grep -E "docs:gen|docs:lint" package.json

# 确认 scripts/docs-lint.ts 存在
ls scripts/docs-lint.ts

# 确认 tsconfig.json 含 scripts/**/*.ts（精确匹配）
grep "scripts/\*\*/\*.ts" tsconfig.json
```

### 3.2 pnpm install 验证

```bash
pnpm install
```

预期：`pnpm install` 成功，无依赖冲突，workspace 依赖正确解析。

> **注**：pnpm install 验证置于所有依赖命令之前，因为后续 typecheck/build/test/docs:gen/docs:lint 全部依赖 `node_modules` 已安装。

### 3.3 类型检查

```bash
pnpm typecheck
```

预期：TypeScript 编译零错误。覆盖 T7（TSDoc 注释）、T10（docs-lint 脚本）的 TypeScript 变更。

### 3.4 构建

```bash
pnpm build
```

预期：构建零错误。覆盖 T10（docs-lint 脚本编译为 `dist/scripts/docs-lint.js`）。

### 3.5 单元测试

```bash
pnpm test
```

预期：现有测试全量通过，零失败。v1.7 变更不影响现有测试（T12 已评估确认）。

### 3.6 文档生成验证

```bash
pnpm docs:gen
```

预期：TypeDoc 生成成功，`docs/api/` 下产物存在。覆盖 T6（TypeDoc 引入与 typedoc.json 配置）、T7（TSDoc 注释）、T8（docs:gen script 配置）。

### 3.7 契约校验验证

```bash
pnpm docs:lint
```

预期：docs-lint 校验通过，退出码 0，输出 `docs-lint: all references valid`。覆盖 T10（docs-lint 脚本）、T11（docs:lint 配置）。

### 3.8 目录结构校验

```bash
# 校验 docs/ 目录结构
echo "=== docs/ 目录结构 ==="
ls -1 docs/
# 预期：INDEX.md、api.md、api/、usage.md、custom-rules.md、specs/

# 校验 docs/specs/ 仅含 v1.7 相关文档
echo "=== docs/specs/ 目录结构 ==="
ls -1 docs/specs/
# 预期：design-v1.7.md、execution-plan-v1.7.md、requirements-decomposition-v1.7.md、requirements-v1.7.md、v1.7/

# 校验 docs/specs/ 下无历史文档残留
echo "=== 历史文档残留检查 ==="
find docs/specs -maxdepth 1 \( -name "design.md" -o -name "execution-plan.md" -o -name "requirements.md" -o -name "acceptance-report.md" \)
# 预期：无输出

find docs/specs -maxdepth 1 -type d ! -name "specs" ! -name "v1.7"
# 预期：无输出

# 校验归档仓库可访问
echo "=== 归档仓库可访问性 ==="
ls node_modules/@legacy-shield/docs/specs/
# 预期：含 acceptance-report.md、common、design.md、execution-plan.md、phases、requirements.md、v1.1~v1.6
```

### 3.9 验收标准逐项核对

对照阶段 Spec §4.1 和 §4.2 的全部验收标准（AC-1~AC-18）逐项核对：

| AC 编号 | 验收项 | 对应验证步骤 | 验证命令 | 预期结果 |
|---------|--------|------------|---------|---------|
| AC-1 | 归档仓库存在 | §3.2/§3.8 | `ls node_modules/@legacy-shield/docs/specs/` | 归档仓库通过 workspace 可访问 |
| AC-2 | 历史 spec 完整迁移 | §3.8 | `ls node_modules/@legacy-shield/docs/specs/` | 含 v1.1~v1.6、common、phases、4 个根级早期文档 |
| AC-3 | pnpm workspace 配置正确 | §3.2 | `pnpm install` | 成功，退出码 0 |
| AC-4 | INDEX.md 内容完整 | §3.1 | `grep "路径映射表" docs/INDEX.md && grep -i "onboarding" docs/INDEX.md` | 含路径映射表与 onboarding 说明 |
| AC-5 | TypeDoc 配置正确 | §3.1/§3.6 | `cat typedoc.json` + `pnpm docs:gen` | typedoc.json 含 5 个 entryPoints，docs:gen 生成成功 |
| AC-6 | TSDoc 覆盖全部公开 API | §3.6 | `ls docs/api/modules/` | 含 5 个模块 API 文档（api、code-quality、knowledge-graph、custom-rules、types） |
| AC-7 | docs:gen 生成成功 | §3.6 | `pnpm docs:gen` | docs/api/ 产物存在 |
| AC-8 | api.md 改为入口索引 | §3.1 | `grep "api/index.html" docs/api.md` | 含指向 docs/api/ 的链接 |
| AC-9 | docs-lint 校验通过 | §3.7 | `pnpm docs:lint` | 退出码 0，输出 all references valid |
| AC-10 | docs:lint script 配置 | §3.1 | `grep "docs:lint" package.json` | scripts 含 docs:lint |
| AC-11 | spec-guardian 路径评估完成 | §3.1 | `ls docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md` | 评估文档存在 |
| AC-12 | 测试路径评估完成 | §3.1 | `ls docs/specs/v1.7/evaluations/t12-test-path-evaluation.md` | 评估文档存在 |
| AC-13 | pnpm typecheck 通过 | §3.3 | `pnpm typecheck` | 退出码 0 |
| AC-14 | pnpm build 通过 | §3.4 | `pnpm build` | 退出码 0 |
| AC-15 | pnpm test 全量通过 | §3.5 | `pnpm test` | 退出码 0 |
| AC-16 | pnpm docs:gen 生成成功 | §3.6 | `pnpm docs:gen` | 退出码 0 |
| AC-17 | pnpm docs:lint 校验通过 | §3.7 | `pnpm docs:lint` | 退出码 0 |
| AC-18 | docs/ 目录结构符合预期 | §3.8 | `ls -1 docs/` | 仅含 INDEX.md、api.md、api/、usage.md、custom-rules.md、specs/ |

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| `pnpm install` 成功 | `pnpm install` | 退出码 0，无依赖冲突 |
| `pnpm typecheck` 通过 | `pnpm typecheck` | 退出码 0，零错误 |
| `pnpm build` 通过 | `pnpm build` | 退出码 0，`dist/` 产物完整 |
| `pnpm test` 全量通过 | `pnpm test` | 退出码 0，零失败 |
| `pnpm docs:gen` 生成成功 | `pnpm docs:gen` | 退出码 0，`docs/api/` 产物存在 |
| `pnpm docs:lint` 校验通过 | `pnpm docs:lint` | 退出码 0，输出 `docs-lint: all references valid` |
| `docs/` 目录结构符合预期 | `ls -1 docs/` | 仅含 INDEX.md、api.md、api/、usage.md、custom-rules.md、specs/ |
| `docs/specs/` 仅含 v1.7 | `ls -1 docs/specs/` | 仅含 design-v1.7.md、execution-plan-v1.7.md、requirements-decomposition-v1.7.md、requirements-v1.7.md、v1.7/ |
| INDEX.md 内容完整（AC-4） | `grep "路径映射表" docs/INDEX.md && grep -i "onboarding" docs/INDEX.md` | 含路径映射表与 onboarding 说明 |
| 归档仓库可访问 | `ls node_modules/@legacy-shield/docs/specs/` | 含 acceptance-report.md、common、design.md、execution-plan.md、phases、requirements.md、v1.1~v1.6 |
| AC-1~AC-18 全部通过 | 逐项核对 §3.9 AC 清单 | 全部通过 |

### 4.2 集成测试

| 测试项 | 测试方式 | 预期结果 |
|-------|---------|---------|
| 构建-校验链路 | `pnpm build && pnpm docs:lint && pnpm test` | 三步依次执行，全部通过 |
| 文档生成-校验链路 | `pnpm docs:gen && pnpm docs:lint` | 先生成 `docs/api/`，再校验活文档引用 |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T4/T5/T9/T11/T12 未完成 | 回归测试范围不完整 | T13 依赖全部前置任务，执行前逐一确认完成状态 |
| `pnpm install` 失败 | workspace 依赖冲突 | 检查 T3（pnpm-workspace.yaml 配置、@legacy-shield/docs 依赖）、归档仓库是否在 `../legacy-shield-docs` |
| 归档仓库不可访问 | `node_modules/@legacy-shield/docs` 不存在 | 检查 pnpm install 是否成功、workspace 配置是否正确 |
| `pnpm typecheck` 失败 | TypeScript 编译错误 | 检查 T7（TSDoc 注释语法）、T10（docs-lint 脚本类型） |
| `pnpm build` 失败 | 构建错误 | 检查 T10（docs-lint 脚本编译）、tsconfig.json include 配置 |
| `pnpm test` 失败 | 现有测试被破坏 | 检查 T7（是否误改了函数体）、T10（是否影响了现有模块） |
| `pnpm docs:gen` 失败 | TypeDoc 生成错误 | 检查 T6（typedoc.json 配置）、T7（TSDoc 语法） |
| `pnpm docs:lint` 失败 | 文档引用不一致 | 修正活文档中不一致的引用 |
| 目录结构不符合预期 | 历史 spec 残留 | 检查 T2（迁移是否完整） |

---

## 6. 变更范围

T13 为测试任务，不修改任何文件。仅执行验证命令并记录结果。

若回归测试发现问题，需回到对应任务修复后重新执行 T13。若需整体回滚，参见 [design-v1.7.md §7 回滚策略](../../design-v1.7.md)（包含 5 步回滚步骤：删除 pnpm-workspace.yaml、恢复 docs/specs/、移除 typedoc、移除 docs-lint、删除 docs/api/ 与 INDEX.md）。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：通过（第二轮复审）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：§3.8 pnpm install 验证排序错误 → 已移至 §3.2（所有依赖命令之前）
- P1-2：§3.1 前置条件缺失 T5/T12 评估产物校验 → 已新增 ls 命令校验两个评估文档
- P1-3：§3.7 归档仓库预期内容不完整 → 已更新为完整列表（含 4 个根级早期文档）

### P2 优化项

第一轮 P2 问题（已修复）：
- P2-1：§3.9 验收标准逐项核对缺乏具体清单 → 已新增 AC-1~AC-18 逐项核对表
- P2-2：§3.5 docs:gen 覆盖描述不完整 → 已更新为覆盖 T6/T7/T8
- P2-3：§3.1 tsconfig.json 校验命令过于宽松 → 已改为精确匹配 `scripts/\*\*/\*.ts`
- P2-4：§5 风险表缺失 pnpm install 失败与归档仓库不可访问风险 → 已新增 2 行风险
- P2-5：§3.7 find 命令运算符优先级问题 → 已添加括号分组
- P2-6：§6 变更范围未引用设计文档 §7 回滚策略 → 已添加引用
- P2-7：§4.1 测试计划缺少 AC-4 验证项 → 已新增 INDEX.md 内容完整性验证项
