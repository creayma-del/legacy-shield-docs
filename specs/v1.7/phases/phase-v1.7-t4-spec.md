# 创建 INDEX.md

> 任务编号：T4
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.1.4
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T4
> 依赖任务：T3
> 评审记录：见本文档末尾

---

## 1. 任务目标

创建 `docs/INDEX.md` 作为文档总入口，列出所有文档位置与状态，包含活文档表、当前版本 Spec 表、历史归档表、历史文档路径映射表，以及 onboarding 说明（克隆主仓库后需额外克隆归档仓库至 `../legacy-shield-docs`，否则 `pnpm install` 失败）。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-1-5 | 主仓库维护 `docs/INDEX.md` 作为文档总入口 | INDEX.md 列出所有文档位置与状态，活文档在主仓库，历史 spec 在归档仓库 |

**对应阶段 Spec 验收标准**：AC-4（INDEX.md 存在，内容完整覆盖所有文档，含路径映射表与 onboarding 说明）

---

## 3. 实现步骤

### 3.1 前置条件确认

T3 已完成 pnpm workspace 配置，归档仓库可通过 `node_modules/@legacy-shield/docs/` 访问。执行以下命令确认：

```bash
ls node_modules/@legacy-shield/docs/specs/
# 预期：v1.1 v1.2 v1.3 v1.4 v1.5 v1.6 common phases design.md ...
```

### 3.2 创建 docs/INDEX.md

按设计文档 §2.1.4 结构创建 `docs/INDEX.md`，包含以下章节：

#### 3.2.1 活文档（主仓库）

| 文档 | 路径 | 说明 |
|------|------|------|
| API 文档 | docs/api/ | TypeDoc 生成，pnpm docs:gen 更新 |
| API 入口 | docs/api.md | 指向 docs/api/ 的入口索引 |
| 使用指南 | docs/usage.md | 手写 |
| 自定义规则 | docs/custom-rules.md | 手写 |

#### 3.2.2 当前版本 Spec（主仓库）

列出 v1.7 的所有 spec 文档（状态反映开发阶段实际状态，即所有任务 Spec 评审通过后进入开发阶段时的状态）：

| 文档 | 路径 | 状态 |
|------|------|------|
| v1.7 需求 | docs/specs/requirements-v1.7.md | 已通过 |
| v1.7 需求分解 | docs/specs/requirements-decomposition-v1.7.md | 已通过 |
| v1.7 设计 | docs/specs/design-v1.7.md | 已通过 |
| v1.7 执行计划 | docs/specs/execution-plan-v1.7.md | 已通过 |
| v1.7 阶段 Spec | docs/specs/v1.7/phases/phase-v1.7-spec.md | 已通过 |
| v1.7 任务 Spec（T1~T13） | docs/specs/v1.7/phases/phase-v1.7-t{1~13}-spec.md | 已通过（开发阶段创建 INDEX.md 时全部任务 Spec 已通过评审） |
| v1.7 需求对齐会议纪要 | docs/specs/v1.7/meetings/requirements-alignment-v1.7-20260624.md | 已通过 |

#### 3.2.3 历史归档（@legacy-shield/docs）

| 版本 | 路径 | 状态 |
|------|------|------|
| v1.1~v1.6 | node_modules/@legacy-shield/docs/specs/v{x}.{y}/ | 已归档冻结 |
| 早期文档 | node_modules/@legacy-shield/docs/specs/ | 已归档冻结 |
| common | node_modules/@legacy-shield/docs/specs/common/ | 已归档冻结 |
| phases | node_modules/@legacy-shield/docs/specs/phases/ | 已归档冻结 |

#### 3.2.4 历史文档路径映射表

列出迁移前后路径对应关系：

| 迁移前路径（主仓库） | 迁移后路径（归档仓库） |
|---------------------|----------------------|
| docs/specs/v1.1/ | node_modules/@legacy-shield/docs/specs/v1.1/ |
| docs/specs/v1.2/ | node_modules/@legacy-shield/docs/specs/v1.2/ |
| docs/specs/v1.3/ | node_modules/@legacy-shield/docs/specs/v1.3/ |
| docs/specs/v1.4/ | node_modules/@legacy-shield/docs/specs/v1.4/ |
| docs/specs/v1.5/ | node_modules/@legacy-shield/docs/specs/v1.5/ |
| docs/specs/v1.6/ | node_modules/@legacy-shield/docs/specs/v1.6/ |
| docs/specs/common/ | node_modules/@legacy-shield/docs/specs/common/ |
| docs/specs/phases/ | node_modules/@legacy-shield/docs/specs/phases/ |
| docs/specs/design.md | node_modules/@legacy-shield/docs/specs/design.md |
| docs/specs/requirements.md | node_modules/@legacy-shield/docs/specs/requirements.md |
| docs/specs/execution-plan.md | node_modules/@legacy-shield/docs/specs/execution-plan.md |
| docs/specs/acceptance-report.md | node_modules/@legacy-shield/docs/specs/acceptance-report.md |

#### 3.2.5 Onboarding 说明

INDEX.md 中需包含 Onboarding 说明章节，内容如下：

````markdown
## Onboarding 说明

克隆主仓库后，需额外克隆归档仓库至同级目录（与主仓库同级），否则 pnpm install 失败：

```bash
# 在目标父目录下执行（两个仓库为同级目录）
git clone <main-repo-url> legacy-shield
git clone <archive-repo-url> legacy-shield-docs
cd legacy-shield
pnpm install
```
````

---

## 4. 测试计划

### 4.1 验证项

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| INDEX.md 存在 | `ls docs/INDEX.md` | 文件存在 |
| 活文档表完整 | 读取 INDEX.md，检查活文档表 | 含 API 文档、API 入口、使用指南、自定义规则 4 项 |
| 当前版本 Spec 表完整 | 读取 INDEX.md，检查 Spec 表 | 含 v1.7 全部 spec 文档 |
| 历史归档表完整 | 读取 INDEX.md，检查归档表 | 含 v1.1~v1.6、common、phases、根目录早期文档 |
| 路径映射表完整 | 读取 INDEX.md，检查映射表 | 含 v1.1~v1.6、common、phases、4 个根目录早期文档的映射 |
| Onboarding 说明存在 | 读取 INDEX.md，检查 onboarding | 含克隆归档仓库说明与 pnpm install 命令 |
| 路径映射表路径正确 | 对照 node_modules/@legacy-shield/docs/specs/ 实际目录 | 映射表路径与实际目录一致 |

### 4.2 回归测试

| 检查项 | 命令 | 预期结果 |
|-------|------|---------|
| 类型检查 | `pnpm typecheck` | 通过（INDEX.md 为文档，不影响编译） |
| 构建 | `pnpm build` | 通过 |
| 测试 | `pnpm test` | 全量通过 |

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T3 未完成（workspace 未配置） | 无法验证归档仓库路径 | T4 依赖 T3，执行前确认 T3 已完成 |
| 路径映射表遗漏 | 部分 historical spec 路径缺失 | 对照 T2 迁移范围清单逐一核对 |
| INDEX.md 列出的文档与实际不一致 | 文档索引不准确 | 创建后逐一验证每个路径是否存在 |

---

## 6. 变更范围

| 文件 | 变更类型 |
|------|---------|
| `docs/INDEX.md` | 新建 |

不修改任何源码文件、不修改现有测试、不修改 cli.ts。

---

## 7. 评审记录

> 评审日期：2026-06-24
> 评审结论：通过（第二轮）
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无。

### P1 重要问题

第一轮 P1 问题（已修复）：
- P1-1：Onboarding git clone 路径错误 → 已修正为 legacy-shield-docs 并添加注释
- P1-2：路径映射表省略 v1.3~v1.5 → 已完整列出全部 6 个版本
- P1-3：任务 Spec 状态标注逻辑矛盾 → 已添加说明消除时序歧义

第二轮评审：无新增 P0/P1/P2 问题。

### P2 优化项

无。
