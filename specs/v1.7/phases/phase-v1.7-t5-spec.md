# 评估 spec-guardian 路径引用

> 任务编号：T5
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §8.1
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T5
> 依赖任务：T3
> 评审记录：见本文档末尾

---

## 1. 任务目标

评估 spec-guardian skill 中引用 `docs/specs/` 路径的地方，迁移后路径变化是否需同步调整。输出评估结论文档至 `docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md`。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-4-2 | 评估 spec-guardian skill 中引用 `docs/specs/` 路径的地方，迁移后路径变化是否需要同步调整 | 输出评估结论，若需调整则同步调整 |

**对应阶段 Spec 验收标准**：AC-11（spec-guardian skill 路径评估完成，结论文档输出）

### 2.1 设计文档 §8.1 评估结论（作为本任务的评估依据）

设计文档已给出预评估结论，T5 需在实际迁移完成后进行验证性评估：

**评估方法**：搜索 `.trae/skills/` 下所有 skill 文件中对 `docs/specs/` 的引用。

**预评估结论**：5 个 skill 文件引用了 `docs/specs/` 路径，分两类场景：

**场景 1：命名规范（新文档创建）**
- skill 中的路径引用（如 `docs/specs/requirements-v{x}.{y}.md`）用于指导新版本 spec 文档的创建位置
- 迁移后：v1.7+ 的 spec 文档仍在主仓库 `docs/specs/v1.7/` 下创建
- 是否需调整：**否**

**场景 2：历史文档读取（归档检查）**
- spec-guardian skill 在执行归档检查时需要定位历史文档
- 迁移后历史文档路径从 `docs/specs/v1.6/design-v1.6.md` 变为 `node_modules/@legacy-shield/docs/specs/v1.6/design-v1.6.md`
- 是否需调整：**是（需在 INDEX.md 中提供路径映射表）**
- 具体措施：
  1. 在 `docs/INDEX.md` 中增加"历史文档路径映射表"（T4 负责）
  2. spec-guardian skill 本身不修改流程规则，但在需要读取历史 spec 时，通过 INDEX.md 的路径映射表定位
  3. 对 `project-rules.md` 路径不一致问题（`docs/specs/project-rules.md` vs `docs/specs/common/project-rules.md`），在迁移时一并修正

---

## 3. 实现步骤

### 3.1 前置条件确认

T3 已完成 pnpm workspace 配置，归档仓库可通过 `node_modules/@legacy-shield/docs/` 访问。T4 已创建 INDEX.md 含路径映射表。

### 3.2 搜索 spec-guardian skill 中的 docs/specs/ 路径引用

```bash
# 搜索 .trae/skills/ 下所有文件中对 docs/specs/ 的引用
grep -rn "docs/specs/" .trae/skills/
```

记录所有匹配的文件路径、行号、引用内容。

### 3.2.1 专项搜索 project-rules.md 路径引用

设计文档 §8.1 已识别 `spec-guardian-templates/SKILL.md` 中存在 `docs/specs/project-rules.md` 错误路径引用（实际路径为 `docs/specs/common/project-rules.md`）。T5 需对该问题进行专项评估：

```bash
# 搜索 .trae/skills/ 下所有文件中对 project-rules.md 的引用
grep -rn "project-rules" .trae/skills/

# 搜索全局 skills 目录（主目录）中对 project-rules.md 的引用
grep -rn "project-rules" ~/.trae-cn/skills/ 2>/dev/null || echo "全局目录不存在或无匹配"

# 搜索归档仓库中 project-rules.md 的实际位置
find ../legacy-shield-docs/specs/ -name "project-rules.md"
# 预期：../legacy-shield-docs/specs/common/project-rules.md

# 搜索主仓库中是否仍有 project-rules.md 残留
find docs/ -name "project-rules.md"
# 预期：无输出（已迁移至归档仓库）
```

评估要点：
1. 逐一检查每个 `project-rules.md` 引用，判断其路径是否正确（`docs/specs/common/project-rules.md` vs 错误的 `docs/specs/project-rules.md`）
2. 对路径错误的引用，评估迁移后是否需修正为归档仓库路径 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`
3. 记录所有路径错误的引用位置（文件路径、行号），在评估结论文档中列出修正方案
4. 若全局目录 `~/.trae-cn/skills/` 与项目本地 `.trae/skills/` 均存在相同错误，修正方案需同时覆盖两处；若全局目录不存在，在评估文档中记录为已知限制

### 3.3 分类评估

将搜索结果按两类场景分类：

1. **命名规范场景**：路径引用用于指导新文档创建位置（如 `docs/specs/requirements-v{x}.{y}.md`）
2. **历史文档读取场景**：路径引用用于读取/检查已存在的 spec 文档

### 3.4 验证设计文档 §8.1 预评估结论

对照设计文档 §8.1 的预评估结论，验证：

1. 场景 1（命名规范）：v1.7+ spec 仍在主仓库，无需调整 — 验证此结论是否成立
2. 场景 2（历史文档读取）：历史文档路径已变更，需通过 INDEX.md 路径映射表定位 — 验证 T4 创建的路径映射表是否覆盖所有历史文档路径
3. project-rules.md 路径不一致问题：验证归档仓库中路径是否为 `specs/common/project-rules.md`

### 3.5 输出评估结论文档

创建 `docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md`，包含：

1. 评估方法与搜索结果（含 §3.2 通用搜索 + §3.2.1 project-rules.md 专项搜索）
2. 分类评估结论（场景 1 + 场景 2）
3. 设计文档 §8.1 预评估结论验证结果
4. project-rules.md 路径不一致问题专项评估结论（列出所有错误路径引用位置、修正方案）
5. 最终结论：是否需调整 spec-guardian skill（预期结论：不需调整 skill 流程规则，通过 INDEX.md 路径映射表定位历史文档；project-rules.md 错误路径引用需修正）

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| 评估结论文档存在 | `ls docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md` | 文件存在 |
| 搜索结果完整 | 检查评估文档中的搜索结果 | 覆盖 .trae/skills/ 下所有 docs/specs/ 引用 |
| 分类评估结论明确 | 检查评估文档中的分类结论 | 场景 1 和场景 2 均有明确结论 |
| 设计文档预评估结论已验证 | 检查评估文档中的验证部分 | §8.1 预评估结论已逐一验证 |
| 最终结论明确 | 检查评估文档中的最终结论 | 明确是否需调整 spec-guardian skill |
| project-rules.md 路径问题已评估并记录修正方案 | 检查评估文档中的 project-rules.md 部分 | 列出所有错误路径引用位置（文件路径、行号），给出修正方案 |
| project-rules.md 专项搜索完整 | 检查评估文档中的 §3.2.1 搜索结果 | 覆盖 .trae/skills/ 下所有 project-rules 引用，含归档仓库实际路径验证 |

### 4.2 回归测试

无（本任务为评估类任务，不修改代码，不影响现有功能。T13 全量回归测试将验证 pnpm typecheck、pnpm build、pnpm test 全部通过）。

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T3 未完成（workspace 未配置） | 无法验证归档仓库路径 | T5 依赖 T3，执行前确认 T3 已完成 |
| T4 未完成（INDEX.md 未创建） | 无法验证路径映射表 | T5 依赖 T4 的 INDEX.md 路径映射表，建议 T4 完成后执行 T5 |
| skill 文件引用遗漏 | 评估不完整 | 使用 grep -r 递归搜索 .trae/skills/ 全部文件 |
| 设计文档预评估结论与实际不符 | 评估结论偏差 | 以实际搜索结果为准，记录差异原因 |
| project-rules.md 路径修正无人实施 | 评估结论记录的修正方案无法落地 | T5 评估文档输出修正方案后，由项目负责人决定是否开 PATCH 任务修正 spec-guardian-templates/SKILL.md 路径引用 |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `docs/specs/v1.7/evaluations/t5-spec-guardian-evaluation.md` | 新建（评估结论文档） |

不修改 spec-guardian skill 文件（T5 为评估类任务，仅输出评估结论与修正方案文档，不执行 skill 文件修改。评估预期结论：skill 流程规则不需调整，但 project-rules.md 错误路径引用需修正，修正方案由评估结论文档记录，后续由独立 PATCH 或跟进任务实施）。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：修改后通过（第二轮）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：project-rules.md 路径引用评估覆盖不足 → 已新增 §3.2.1 专项搜索

第二轮 P1 问题（已修复）：
- P1-1：§6 变更范围与 §3.5 评估结论逻辑矛盾 → 已修正 §6 括注，明确 project-rules.md 修正责任归属

### P2 优化项

第二轮 P2 问题（已修复）：
- P2-1：§3.2 通用搜索命令缺少 -n 参数 → 已改为 grep -rn
- P2-2：未覆盖全局 skills 目录 → 已新增 ~/.trae-cn/skills/ 搜索命令与评估要点
- P2-3：§4.1 验证项名称"已处理"语义不准确 → 已改为"已评估并记录修正方案"
