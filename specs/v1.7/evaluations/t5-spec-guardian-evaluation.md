# T5 评估结论：spec-guardian 路径引用

> 任务编号：T5
> 版本：v1.7
> 创建日期：2026-06-24
> 对应任务 Spec：[phase-v1.7-t5-spec.md](../phases/phase-v1.7-t5-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §8.1

---

## 1. 评估方法

### 1.1 通用搜索

```bash
grep -rn "docs/specs/" .trae/skills/
```

搜索 `.trae/skills/` 下所有 skill 文件中对 `docs/specs/` 的引用。

### 1.2 project-rules.md 专项搜索

```bash
grep -rn "project-rules" .trae/skills/
grep -rn "project-rules" ~/.trae-cn/skills/
find ../legacy-shield-docs/specs/ -name "project-rules.md"
find docs/ -name "project-rules.md"
```

---

## 2. 搜索结果

### 2.1 通用搜索结果（docs/specs/ 引用）

共 16 处匹配，分布在 6 个 skill 文件中：

| 文件 | 行号 | 引用内容 |
|------|------|---------|
| spec-guardian-design/SKILL.md | 42 | `docs/specs/design-v{x}.{y}.md` |
| spec-guardian-design/SKILL.md | 49 | `docs/specs/execution-plan-v{x}.{y}.md` |
| spec-guardian-spec/SKILL.md | 38 | `docs/specs/phases/phase-v{x}.{y}-t{n}-spec.md` |
| spec-guardian-requirements/SKILL.md | 30 | `docs/specs/meetings/requirements-alignment-v{x}.{y}-YYYYMMDD.md` |
| spec-guardian-requirements/SKILL.md | 38 | `docs/specs/requirements-decomposition-v{x}.{y}.md` |
| spec-guardian-doc-templates/SKILL.md | 104 | `docs/specs/phases/phase-v{x}.{y}-t{n}-spec.md` |
| spec-guardian-templates/SKILL.md | 14 | `docs/specs/project-rules.md` |
| spec-guardian-templates/SKILL.md | 15 | `docs/specs/requirements-v{x}.{y}.md` |
| spec-guardian-templates/SKILL.md | 16 | `docs/specs/meetings/requirements-alignment-v{x}.{y}-YYYYMMDD.md` |
| spec-guardian-templates/SKILL.md | 17 | `docs/specs/requirements-decomposition-v{x}.{y}.md` |
| spec-guardian-templates/SKILL.md | 18 | `docs/specs/design-v{x}.{y}.md` |
| spec-guardian-templates/SKILL.md | 19 | `docs/specs/execution-plan-v{x}.{y}.md` |
| spec-guardian-templates/SKILL.md | 20 | `docs/specs/phases/phase-v{x}.{y}-spec.md` |
| spec-guardian-templates/SKILL.md | 21 | `docs/specs/phases/phase-v{x}.{y}-t{n}-spec.md` |
| spec-guardian-templates/SKILL.md | 22 | `docs/specs/phases/phase-v{x}.{y}-patch-{n}-spec.md` |
| spec-guardian-templates/SKILL.md | 23 | `docs/specs/acceptance-report-v{x}.{y}.md` |

### 2.2 project-rules.md 专项搜索结果

| 位置 | 行号 | 引用内容 | 路径是否正确 |
|------|------|---------|------------|
| `.trae/skills/spec-guardian-templates/SKILL.md` | 14 | `docs/specs/project-rules.md` | **错误**（实际路径为 `docs/specs/common/project-rules.md`） |
| `~/.trae-cn/skills/spec-guardian-templates/SKILL.md` | 14 | `docs/specs/project-rules.md` | **错误**（同上，全局目录存在相同错误） |

归档仓库实际路径：`../legacy-shield-docs/specs/common/project-rules.md`
主仓库残留检查：无残留（已迁移至归档仓库）

---

## 3. 分类评估

### 3.1 场景 1：命名规范（新文档创建）

**涉及引用**：除 `project-rules.md` 外的全部 15 处引用。

这些路径引用（如 `docs/specs/requirements-v{x}.{y}.md`、`docs/specs/design-v{x}.{y}.md` 等）用于指导新版本 spec 文档的创建位置。它们使用 `{x}.{y}` 占位符，指向的是未来版本的文档路径。

**迁移后影响**：v1.7+ 的 spec 文档仍在主仓库 `docs/specs/` 下创建（如 `docs/specs/v1.7/`、`docs/specs/design-v1.7.md` 等），路径模式未变。

**结论**：**无需调整**。命名规范场景的路径引用在迁移后仍然有效，新版本 spec 继续在主仓库 `docs/specs/` 下创建。

### 3.2 场景 2：历史文档读取（归档检查）

**涉及引用**：无直接的历史文档读取引用。

spec-guardian skill 在执行归档检查时，如需定位历史文档（v1.1~v1.6），路径已从 `docs/specs/v1.6/design-v1.6.md` 变为 `node_modules/@legacy-shield/docs/specs/v1.6/design-v1.6.md`。

**迁移后影响**：历史文档已迁移至归档仓库，通过 pnpm workspace 引入后可通过 `node_modules/@legacy-shield/docs/specs/` 访问。

**结论**：**无需调整 skill 流程规则**。T4 已在 `docs/INDEX.md` 中创建"历史文档路径映射表"，spec-guardian skill 在需要读取历史 spec 时，通过 INDEX.md 的路径映射表定位即可。

---

## 4. 设计文档 §8.1 预评估结论验证

| 预评估结论 | 验证结果 |
|-----------|---------|
| 5 个 skill 文件引用了 `docs/specs/` 路径 | **修正**：实际为 6 个 skill 文件（spec-guardian-design、spec-guardian-spec、spec-guardian-requirements、spec-guardian-doc-templates、spec-guardian-templates，共 16 处引用） |
| 场景 1（命名规范）：v1.7+ spec 仍在主仓库，无需调整 | **验证通过**：v1.7 spec 确实在主仓库 `docs/specs/v1.7/` 下创建 |
| 场景 2（历史文档读取）：需通过 INDEX.md 路径映射表定位 | **验证通过**：T4 已创建 INDEX.md 含完整路径映射表（12 条映射记录） |
| project-rules.md 路径不一致问题需修正 | **验证通过**：归档仓库路径为 `specs/common/project-rules.md`，skill 中引用为 `docs/specs/project-rules.md`，确为错误路径 |

---

## 5. project-rules.md 路径不一致问题专项评估

### 5.1 问题描述

`spec-guardian-templates/SKILL.md` 第 14 行引用项目总规范路径为 `docs/specs/project-rules.md`，但实际路径为 `docs/specs/common/project-rules.md`（迁移前）或 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`（迁移后）。

### 5.2 影响范围

| 位置 | 文件 | 行号 | 错误路径 |
|------|------|------|---------|
| 项目本地 | `.trae/skills/spec-guardian-templates/SKILL.md` | 14 | `docs/specs/project-rules.md` |
| 全局 | `~/.trae-cn/skills/spec-guardian-templates/SKILL.md` | 14 | `docs/specs/project-rules.md` |

### 5.3 修正方案

迁移后 `project-rules.md` 已归档至归档仓库，路径为 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`。

**修正方案**：
1. 将 `spec-guardian-templates/SKILL.md` 第 14 行的 `docs/specs/project-rules.md` 修正为 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`
2. 同时修正项目本地（`.trae/skills/`）和全局（`~/.trae-cn/skills/`）两处
3. 修正后 project-rules.md 引用指向归档仓库，与实际路径一致

**修正责任归属**：T5 为评估类任务，仅输出修正方案。实际修正由后续独立 PATCH 或跟进任务实施。

---

## 6. 最终结论

1. **spec-guardian skill 流程规则**：**无需调整**。命名规范场景的路径引用在迁移后仍然有效；历史文档读取场景通过 INDEX.md 路径映射表定位。
2. **project-rules.md 错误路径引用**：**需修正**。`spec-guardian-templates/SKILL.md` 第 14 行的 `docs/specs/project-rules.md` 为错误路径，需修正为 `node_modules/@legacy-shield/docs/specs/common/project-rules.md`。项目本地和全局两处均需修正。
3. **INDEX.md 路径映射表**：T4 已创建完整的路径映射表（12 条映射记录），覆盖所有迁移的历史文档路径。
