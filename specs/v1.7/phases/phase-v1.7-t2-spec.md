# 迁移历史 spec

> 任务编号：T2
> 版本：v1.7
> 状态：已通过
> 创建日期：2026-06-24
> 对应阶段 Spec：[phase-v1.7-spec.md](phase-v1.7-spec.md)
> 对应设计文档：[design-v1.7.md](../../design-v1.7.md) §2.1.1
> 对应执行计划：[execution-plan-v1.7.md](../../execution-plan-v1.7.md) T2
> 依赖任务：T1
> 评审记录：见本文档末尾

---

## 1. 任务目标

将主仓库 `docs/specs/` 下全部历史 spec 文档迁移到归档仓库 `legacy-shield-docs`（位于主仓库同级目录 `../legacy-shield-docs`）的 `specs/` 目录下，迁移后主仓库 `docs/specs/` 仅保留 v1.7 相关文档（`v1.7/` 目录及 v1.7 根级 spec 文件）。

迁移范围包括：v1.1~v1.6 版本目录、`common/` 目录、`phases/` 早期文档目录、根目录早期文档（共 4 个）。迁移方法采用 `cp + git rm`（`git mv` 无法跨仓库移动文件）。

---

## 2. 对应需求与验收标准

| 需求编号 | 需求描述 | 本任务验收标准 |
|---------|---------|--------------|
| REQ-1-2 | 将主仓库 `docs/specs/` 下全部历史 spec 迁移到归档仓库 | 迁移范围完整（见 §3.1 迁移范围清单），主仓库对应目录已删除，归档仓库目录结构完整，文件数与迁移前一致 |

**对应阶段 Spec 验收标准**：AC-2（历史 spec 完整迁移，主仓库 docs/specs/ 仅保留 v1.7 相关文档）

---

## 3. 实现步骤

> **说明**：以下所有命令均假设当前工作目录为主仓库根目录（`../legacy-shield`）。仅在需要切换到归档仓库时使用 `cd ../legacy-shield-docs`，切换回主仓库时使用 `cd ../legacy-shield`。

### 3.1 迁移范围清单

#### 3.1.1 迁移到归档仓库的内容

| 序号 | 迁移源路径（主仓库） | 迁移目标路径（归档仓库） | 说明 |
|------|---------------------|----------------------|------|
| 1 | `docs/specs/v1.1/` | `specs/v1.1/` | 含 phases/（6 个任务 Spec）、acceptance-report-v1.1.md、design-v1.1.md、execution-plan-v1.1.md、requirements-v1.1.md |
| 2 | `docs/specs/v1.2/` | `specs/v1.2/` | 含 meetings/、phases/（8 个任务 Spec）、acceptance-report-v1.2.md、design-v1.2.md、execution-plan-v1.2.md、requirements-decomposition-v1.2.md、requirements-v1.2.md |
| 3 | `docs/specs/v1.3/` | `specs/v1.3/` | 含 meetings/、phases/（9 个任务 Spec）、acceptance-report-v1.3.md、design-v1.3.md、execution-plan-v1.3.md、requirements-decomposition-v1.3.md、requirements-v1.3.md |
| 4 | `docs/specs/v1.4/` | `specs/v1.4/` | 含 meetings/、phases/（8 个任务 Spec）、acceptance-report-v1.4.md、design-v1.4.md、execution-plan-v1.4.md、requirements-decomposition-v1.4.md、requirements-v1.4.md |
| 5 | `docs/specs/v1.5/` | `specs/v1.5/` | 含 meetings/、phases/（14 个任务 Spec）、acceptance-report-v1.5.md、design-v1.5.md、execution-plan-v1.5.md、requirements-decomposition-v1.5.md |
| 6 | `docs/specs/v1.6/` | `specs/v1.6/` | 含 meetings/、phases/（11 个任务 Spec）、design-v1.6.md、execution-plan-v1.6.md、requirements-decomposition-v1.6.md、requirements-v1.6.md |
| 7 | `docs/specs/phases/` | `specs/phases/` | 早期无版本号文档：phase-1-spec.md ~ phase-5-spec.md（5 个文件） |
| 8 | `docs/specs/design.md` | `specs/design.md` | 根目录早期文档 |
| 9 | `docs/specs/execution-plan.md` | `specs/execution-plan.md` | 根目录早期文档 |
| 10 | `docs/specs/requirements.md` | `specs/requirements.md` | 根目录早期文档 |
| 11 | `docs/specs/acceptance-report.md` | `specs/acceptance-report.md` | 根目录早期文档 |
| 12 | `docs/specs/common/` | `specs/common/` | project-rules.md、typescript-migration.md（2 个文件） |

#### 3.1.2 保留在主仓库的内容

| 保留路径 | 说明 |
|---------|------|
| `docs/specs/v1.7/` | 本次 v1.7 的 spec 文档（含 meetings/、phases/ 等） |
| `docs/specs/design-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/requirements-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/requirements-decomposition-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/execution-plan-v1.7.md` | v1.7 根级 spec 文件 |

### 3.2 执行前环境检查

迁移执行前，需确认主仓库 git 工作区干净、归档仓库已初始化 git：

```bash
# 确认主仓库 git 工作区干净
if [ -n "$(git status --porcelain)" ]; then
  echo "错误：主仓库 git 工作区不干净，请先提交或 stash 未提交变更"
  git status
  exit 1
fi

# 确认归档仓库存在且 git 已初始化
ls -d ../legacy-shield-docs/.git || { echo "错误：归档仓库未初始化 git"; exit 1; }
```

### 3.3 执行前输出完整文件清单

迁移执行前，需输出主仓库待迁移文件的完整清单并经评审确认，确保迁移范围完整无遗漏：

```bash
# 输出待迁移文件清单（迁移前）

# 列出各版本目录文件
for v in v1.1 v1.2 v1.3 v1.4 v1.5 v1.6; do
  echo "=== docs/specs/$v/ ==="
  find "docs/specs/$v" -type f | sort
done

# 列出 phases/ 早期文档
echo "=== docs/specs/phases/ ==="
find docs/specs/phases -type f | sort

# 列出根目录早期文档
echo "=== docs/specs/ 根目录早期文档 ==="
ls -1 docs/specs/design.md docs/specs/execution-plan.md docs/specs/requirements.md docs/specs/acceptance-report.md

# 列出 common/ 文档
echo "=== docs/specs/common/ ==="
find docs/specs/common -type f | sort

# 统计待迁移文件总数（含根目录早期文档）
DIR_COUNT=$(find docs/specs/v1.1 docs/specs/v1.2 docs/specs/v1.3 docs/specs/v1.4 docs/specs/v1.5 docs/specs/v1.6 docs/specs/phases docs/specs/common -type f | wc -l)
ROOT_COUNT=$(ls -1 docs/specs/design.md docs/specs/execution-plan.md docs/specs/requirements.md docs/specs/acceptance-report.md 2>/dev/null | wc -l)
TOTAL=$((DIR_COUNT + ROOT_COUNT))
echo "待迁移文件总数（含根目录早期文档）: $TOTAL"
```

> **注意**：输出文件清单后需经评审确认，确认迁移范围与 §3.1 一致后方可执行迁移。记录此处输出的 `$TOTAL` 值，用于 §3.10 迁移后校验。

### 3.4 获取主仓库当前 commit hash

迁移前获取主仓库当前 commit hash，用于在归档仓库 commit message 中记录，以便后续追溯：

```bash
MAIN_REPO_HASH=$(git rev-parse HEAD)
echo "主仓库当前 commit hash: $MAIN_REPO_HASH"
```

### 3.5 迁移前 git 备份

迁移前在主仓库创建备份分支，以便迁移失败时回滚（回滚步骤见 §5）：

```bash
git branch backup/pre-migration-v1.7
echo "已创建备份分支 backup/pre-migration-v1.7"
```

### 3.6 迁移版本目录（v1.1~v1.6）

对每个版本目录（v1.1、v1.2、v1.3、v1.4、v1.5、v1.6）依次执行以下步骤。以 v1.1 为例：

```bash
# 步骤 1：复制文件到归档仓库
if [ -d ../legacy-shield-docs/specs/v1.1 ]; then
  echo "错误：../legacy-shield-docs/specs/v1.1 已存在，可能为重试，请先清理后重试"
  exit 1
fi
cp -r docs/specs/v1.1 ../legacy-shield-docs/specs/v1.1 || { echo "cp v1.1 失败，终止"; exit 1; }

# 步骤 2：校验复制内容完整性
diff -r docs/specs/v1.1 ../legacy-shield-docs/specs/v1.1 && echo "v1.1 内容校验通过" || { echo "v1.1 内容校验失败"; exit 1; }

# 步骤 3：在归档仓库提交（commit message 记录主仓库原始 commit hash 以便追溯）
cd ../legacy-shield-docs
git add specs/v1.1/
git commit -m "migrate v1.1 specs from main repo (original commit: $MAIN_REPO_HASH)"

# 步骤 4：在主仓库删除并提交
cd ../legacy-shield
git rm -r docs/specs/v1.1
git commit -m "migrate v1.1 specs to archive repo"
```

对 v1.2、v1.3、v1.4、v1.5、v1.6 重复上述步骤（将 `v1.1` 替换为对应版本号）。

### 3.7 迁移 phases/ 早期文档

```bash
# 步骤 1：复制文件到归档仓库
if [ -d ../legacy-shield-docs/specs/phases ]; then
  echo "错误：../legacy-shield-docs/specs/phases 已存在，可能为重试，请先清理后重试"
  exit 1
fi
cp -r docs/specs/phases ../legacy-shield-docs/specs/phases || { echo "cp phases 失败，终止"; exit 1; }

# 步骤 2：校验复制内容完整性
diff -r docs/specs/phases ../legacy-shield-docs/specs/phases && echo "phases 内容校验通过" || { echo "phases 内容校验失败"; exit 1; }

# 步骤 3：在归档仓库提交
cd ../legacy-shield-docs
git add specs/phases/
git commit -m "migrate early phases specs from main repo (original commit: $MAIN_REPO_HASH)"

# 步骤 4：在主仓库删除并提交
cd ../legacy-shield
git rm -r docs/specs/phases
git commit -m "migrate early phases specs to archive repo"
```

### 3.8 迁移根目录早期文档

根目录早期文档共 4 个：`design.md`、`execution-plan.md`、`requirements.md`、`acceptance-report.md`。

```bash
# 步骤 1：复制文件到归档仓库
cp docs/specs/design.md ../legacy-shield-docs/specs/design.md || { echo "cp design.md 失败，终止"; exit 1; }
cp docs/specs/execution-plan.md ../legacy-shield-docs/specs/execution-plan.md || { echo "cp execution-plan.md 失败，终止"; exit 1; }
cp docs/specs/requirements.md ../legacy-shield-docs/specs/requirements.md || { echo "cp requirements.md 失败，终止"; exit 1; }
cp docs/specs/acceptance-report.md ../legacy-shield-docs/specs/acceptance-report.md || { echo "cp acceptance-report.md 失败，终止"; exit 1; }

# 步骤 2：校验复制内容完整性
diff docs/specs/design.md ../legacy-shield-docs/specs/design.md && \
diff docs/specs/execution-plan.md ../legacy-shield-docs/specs/execution-plan.md && \
diff docs/specs/requirements.md ../legacy-shield-docs/specs/requirements.md && \
diff docs/specs/acceptance-report.md ../legacy-shield-docs/specs/acceptance-report.md && \
echo "根目录早期文档内容校验通过" || { echo "根目录早期文档内容校验失败"; exit 1; }

# 步骤 3：在归档仓库提交
cd ../legacy-shield-docs
git add specs/design.md specs/execution-plan.md specs/requirements.md specs/acceptance-report.md
git commit -m "migrate early root specs from main repo (original commit: $MAIN_REPO_HASH)"

# 步骤 4：在主仓库删除并提交
cd ../legacy-shield
git rm docs/specs/design.md docs/specs/execution-plan.md docs/specs/requirements.md docs/specs/acceptance-report.md
git commit -m "migrate early root specs to archive repo"
```

### 3.9 迁移 common/ 目录

```bash
# 步骤 1：复制文件到归档仓库
if [ -d ../legacy-shield-docs/specs/common ]; then
  echo "错误：../legacy-shield-docs/specs/common 已存在，可能为重试，请先清理后重试"
  exit 1
fi
cp -r docs/specs/common ../legacy-shield-docs/specs/common || { echo "cp common 失败，终止"; exit 1; }

# 步骤 2：校验复制内容完整性
diff -r docs/specs/common ../legacy-shield-docs/specs/common && echo "common 内容校验通过" || { echo "common 内容校验失败"; exit 1; }

# 步骤 3：在归档仓库提交
cd ../legacy-shield-docs
git add specs/common/
git commit -m "migrate common specs from main repo (original commit: $MAIN_REPO_HASH)"

# 步骤 4：在主仓库删除并提交
cd ../legacy-shield
git rm -r docs/specs/common
git commit -m "migrate common specs to archive repo"
```

### 3.10 迁移后校验

迁移完成后，执行文件数校验与目录结构校验：

```bash
# 1. 校验主仓库 docs/specs/ 仅保留 v1.7 相关文档
echo "=== 主仓库 docs/specs/ 内容 ==="
ls -1 docs/specs/
# 预期输出：design-v1.7.md、execution-plan-v1.7.md、requirements-decomposition-v1.7.md、requirements-v1.7.md、v1.7/

# 2. 校验主仓库 docs/specs/ 下无历史文档残留
echo "=== 主仓库历史文档残留检查 ==="
find docs/specs -maxdepth 1 -name "design.md" -o -name "execution-plan.md" -o -name "requirements.md" -o -name "acceptance-report.md"
# 预期：无输出（已全部迁移）

find docs/specs -maxdepth 1 -type d ! -name "specs" ! -name "v1.7"
# 预期：无输出（v1.1~v1.6、common、phases 已全部迁移）

# 3. 校验归档仓库 specs/ 目录结构完整
cd ../legacy-shield-docs
echo "=== 归档仓库 specs/ 内容 ==="
ls -1 specs/
# 预期输出：acceptance-report.md、common、design.md、execution-plan.md、phases、requirements.md、v1.1、v1.2、v1.3、v1.4、v1.5、v1.6

# 4. 校验归档仓库文件数与迁移前一致
echo "=== 归档仓库 specs/ 文件总数 ==="
find specs -type f | wc -l
# 预期：与 §3.3 输出的待迁移文件总数（$TOTAL）一致
```

> **注意**：`cp + git rm` 方式不保留文件 git 历史（归档仓库是新 commit，主仓库是删除 commit），迁移时在归档仓库 commit message 中记录主仓库原始 commit hash（§3.4 获取的 `$MAIN_REPO_HASH`）以便后续追溯。

---

## 4. 测试计划

### 4.1 验证项

> **说明**：以下验证命令均假设当前工作目录为主仓库根目录。主仓库路径直接使用相对路径（如 `docs/specs/v1.1`），归档仓库路径使用 `../legacy-shield-docs/` 前缀（如 `../legacy-shield-docs/specs/v1.1`）。

| 验证项 | 验证方式 | 预期结果 |
|-------|---------|---------|
| 主仓库 v1.1~v1.6 目录已删除 | `ls -d docs/specs/v1.1`（v1.1~v1.6 逐个检查） | 目录不存在（报错 No such file or directory） |
| 主仓库 common/ 目录已删除 | `ls -d docs/specs/common` | 目录不存在 |
| 主仓库 phases/ 目录已删除 | `ls -d docs/specs/phases` | 目录不存在 |
| 主仓库根目录早期文档已删除 | `ls docs/specs/design.md docs/specs/execution-plan.md docs/specs/requirements.md docs/specs/acceptance-report.md` | 文件不存在（报错 No such file or directory） |
| 主仓库 v1.7/ 目录保留 | `ls -d docs/specs/v1.7` | 目录存在 |
| 主仓库 v1.7 根级 spec 文件保留 | `ls docs/specs/design-v1.7.md docs/specs/requirements-v1.7.md docs/specs/requirements-decomposition-v1.7.md docs/specs/execution-plan-v1.7.md` | 4 个文件均存在 |
| 归档仓库 v1.1~v1.6 目录存在 | `ls -d ../legacy-shield-docs/specs/v1.1`（v1.1~v1.6 逐个检查） | 目录存在 |
| 归档仓库 common/ 目录存在 | `ls -d ../legacy-shield-docs/specs/common` | 目录存在，含 project-rules.md、typescript-migration.md |
| 归档仓库 phases/ 目录存在 | `ls -d ../legacy-shield-docs/specs/phases` | 目录存在，含 phase-1-spec.md ~ phase-5-spec.md |
| 归档仓库根目录早期文档存在 | `ls ../legacy-shield-docs/specs/design.md ../legacy-shield-docs/specs/execution-plan.md ../legacy-shield-docs/specs/requirements.md ../legacy-shield-docs/specs/acceptance-report.md` | 4 个文件均存在 |
| 归档仓库文件数与迁移前一致 | 迁移前执行 §3.3 统计命令得到 `$TOTAL`；迁移后 `find ../legacy-shield-docs/specs -type f \| wc -l` | 两者一致 |
| 归档仓库 git 提交记录完整 | `cd ../legacy-shield-docs && git log --oneline` | 含各版本迁移 commit（v1.1~v1.6、phases、根目录早期文档、common） |
| 归档仓库 commit message 含主仓库 hash | `cd ../legacy-shield-docs && git log` | commit message 中含 `original commit: <hash>` |
| 主仓库 git 提交记录完整 | `git log --oneline` | 含各版本迁移删除 commit |

### 4.2 集成测试

无（本任务为文件迁移，无代码逻辑，集成测试在 T13 全量回归测试中执行）。

### 4.3 回归测试

无（本任务不修改源码，不影响现有功能。T13 全量回归测试将验证 `pnpm typecheck`、`pnpm build`、`pnpm test` 全部通过）。

---

## 5. 风险与依赖

| 风险 / 依赖 | 影响 | 应对措施 |
|------------|------|---------|
| T1 未完成（归档仓库未创建或 git 未初始化） | 迁移无法执行 | T2 依赖 T1，执行前确认 T1 已通过评审且归档仓库 `../legacy-shield-docs` 存在、git 已初始化（§3.2 环境检查） |
| 迁移过程文件丢失或损坏 | 历史 spec 内容不完整 | 迁移前 git 备份主仓库当前状态（§3.5）；使用 `cp + git rm` 方式迁移（`git mv` 无法跨仓库）；每个迁移步骤在 cp 后、git rm 前执行 diff 内容校验（§3.6~§3.9）；迁移后校验文件数（§3.10） |
| 迁移范围遗漏 | 部分 spec 未迁移，主仓库残留历史文档 | 执行前输出完整文件清单（§3.3）并经评审确认；迁移后执行目录结构校验（§3.10），确认主仓库无历史文档残留 |
| `cp -r` 产生嵌套目录 | 归档仓库目录结构错误（如 `specs/v1.1/v1.1/`） | T1 仅创建 `specs/` 空目录，不预创建子目录；迁移时 `cp -r docs/specs/v1.1 ../legacy-shield-docs/specs/v1.1` 自动创建目标目录；每个 cp -r 前增加目标目录存在性检查（§3.6~§3.9），防止重试时产生嵌套 |
| 归档仓库 commit message 未记录主仓库 hash | 后续无法追溯迁移来源 | §3.4 获取主仓库当前 commit hash，在各迁移 commit message 中记录 `original commit: <hash>` |
| 主仓库 git 工作区不干净 | `git rm` 可能误删未提交的变更 | 迁移前确认主仓库 git 工作区干净（§3.2，`git status` 无未提交变更） |

### 5.1 回滚步骤

若迁移过程中出现错误需回滚，按以下步骤执行：

```
1. 主仓库：git reset --hard backup/pre-migration-v1.7（恢复所有已删除文件）
2. 归档仓库：git reset --hard HEAD~N（N 为已执行的迁移 commit 数）
3. 重新执行迁移
```

> **说明**：`backup/pre-migration-v1.7` 分支由 §3.5 创建，记录了迁移前主仓库的完整状态。归档仓库的 `HEAD~N` 中 N 需根据实际已提交的迁移 commit 数量确定（每个版本目录、phases、根目录早期文档、common 各产生 1 个 commit，最多 9 个）。

---

## 6. 变更范围

### 6.1 归档仓库（`../legacy-shield-docs/`）新增文件

| 文件 / 目录 | 变更类型 |
|------------|---------|
| `specs/v1.1/` | 新建（迁移内容，含 phases/、acceptance-report-v1.1.md、design-v1.1.md、execution-plan-v1.1.md、requirements-v1.1.md） |
| `specs/v1.2/` | 新建（迁移内容，含 meetings/、phases/、acceptance-report-v1.2.md、design-v1.2.md、execution-plan-v1.2.md、requirements-decomposition-v1.2.md、requirements-v1.2.md） |
| `specs/v1.3/` | 新建（迁移内容，含 meetings/、phases/、acceptance-report-v1.3.md、design-v1.3.md、execution-plan-v1.3.md、requirements-decomposition-v1.3.md、requirements-v1.3.md） |
| `specs/v1.4/` | 新建（迁移内容，含 meetings/、phases/、acceptance-report-v1.4.md、design-v1.4.md、execution-plan-v1.4.md、requirements-decomposition-v1.4.md、requirements-v1.4.md） |
| `specs/v1.5/` | 新建（迁移内容，含 meetings/、phases/、acceptance-report-v1.5.md、design-v1.5.md、execution-plan-v1.5.md、requirements-decomposition-v1.5.md） |
| `specs/v1.6/` | 新建（迁移内容，含 meetings/、phases/、design-v1.6.md、execution-plan-v1.6.md、requirements-decomposition-v1.6.md、requirements-v1.6.md） |
| `specs/phases/` | 新建（迁移内容，phase-1-spec.md ~ phase-5-spec.md） |
| `specs/design.md` | 新建（迁移内容） |
| `specs/execution-plan.md` | 新建（迁移内容） |
| `specs/requirements.md` | 新建（迁移内容） |
| `specs/acceptance-report.md` | 新建（迁移内容） |
| `specs/common/` | 新建（迁移内容，project-rules.md、typescript-migration.md） |

### 6.2 主仓库（`../legacy-shield/`）删除文件

| 文件 / 目录 | 变更类型 |
|------------|---------|
| `docs/specs/v1.1/` | 删除（迁移后） |
| `docs/specs/v1.2/` | 删除（迁移后） |
| `docs/specs/v1.3/` | 删除（迁移后） |
| `docs/specs/v1.4/` | 删除（迁移后） |
| `docs/specs/v1.5/` | 删除（迁移后） |
| `docs/specs/v1.6/` | 删除（迁移后） |
| `docs/specs/phases/` | 删除（迁移后） |
| `docs/specs/design.md` | 删除（迁移后） |
| `docs/specs/execution-plan.md` | 删除（迁移后） |
| `docs/specs/requirements.md` | 删除（迁移后） |
| `docs/specs/acceptance-report.md` | 删除（迁移后） |
| `docs/specs/common/` | 删除（迁移后） |

### 6.3 不修改的文件

主仓库 `docs/specs/` 下以下文件保持不变：

| 文件 / 目录 | 说明 |
|------------|------|
| `docs/specs/v1.7/` | v1.7 spec 文档（含 meetings/、phases/） |
| `docs/specs/design-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/requirements-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/requirements-decomposition-v1.7.md` | v1.7 根级 spec 文件 |
| `docs/specs/execution-plan-v1.7.md` | v1.7 根级 spec 文件 |

不修改任何源码文件（`lib/`、`cli.ts`、`tests/` 等）。

---

## 7. 评审记录

> - 第一轮评审（2026-06-24）：修改后通过，发现 6 个 P1 + 8 个 P2 问题
> 评审人：spec-reviewer-expert

### P0 阻塞缺陷

无

### P1 重要问题

无（第一轮 P1-1~P1-6 均已修复闭环）

### P2 优化项

P2-3（find 命令括号优先级）、P2-5（git log 校验）、P2-6（任务 Spec 计数描述）、P2-8（v1.7 文档确认输出）作为可选优化项，在实现阶段按需处理。
