# 需求对齐会议纪要

> 版本：v1.7
> 会议时间：2026-06-24 18:30
> 记录人：SOLO Coder

## 参与人员

- 用户 / 产品负责人：creayma
- SOLO Coder：SOLO Coder

## 会议背景

项目 docs 目录已累积 100+ 个 markdown 文件，未来会持续增长。需建立可持续的文档维护机制，保证文档与代码强对齐，AI 读取高效，主项目与文档互相依赖，避免版本不一致或文档与项目功能不对齐。

用户提出几个候选方向：MCP、文档抽取到独立项目、其他方式。SOLO Coder 提供 5 种方案对比分析（A 分层归档 / B 文档抽取独立仓库 / C 文档即代码 / D MCP 检索层 / E 契约校验），用户选择「方案 A + C + E 全套」组合。

## 需求条目确认

| 需求编号 | 需求描述 | 确认结论 | 备注 |
|---------|---------|---------|------|
| REQ-1-1 | 创建独立归档仓库 `legacy-shield-docs`，package.json name 为 `@legacy-shield/docs` | 确认 | 归档仓库名称用户已选定 |
| REQ-1-2 | 全部历史 spec 迁移到归档仓库 | 确认 | 用户选择「全部历史 spec」，含 v1.1~v1.6 + 早期无版本号文档 + common/ |
| REQ-1-3 | 主仓库通过 pnpm workspace 引入归档仓库 | 确认 | 用户选择 pnpm workspace（非 git submodule） |
| REQ-1-4 | 主仓库 docs/ 只保留活文档 | 确认 | 活文档：usage.md、custom-rules.md、api.md（入口索引）、api/（生成）、INDEX.md |
| REQ-1-5 | 主仓库维护 docs/INDEX.md 作为文档总入口 | 确认 | |
| REQ-2-1 | 引入 TypeDoc 作为 devDependency | 确认 | |
| REQ-2-2 | api.md 内容迁移为源码 TSDoc 注释 | 确认 | 仅 api.md 代码化，usage.md/custom-rules.md 保持手写 |
| REQ-2-3 | TypeDoc 生成产物输出到 docs/api/ 目录 | 确认 | 用户选择 docs/api/ 目录（多文件按模块拆分），非覆盖单文件 |
| REQ-2-4 | 原 docs/api.md 改为入口索引 | 确认 | |
| REQ-2-5 | 新增 npm script docs:gen | 确认 | |
| REQ-3-1 | 新增独立 docs-lint 脚本，npm script docs:lint | 确认 | 用户选择独立脚本（非集成到 quality，非 pre-commit hook） |
| REQ-3-2 | docs-lint 校验范围：仅主仓库活文档 | 确认 | 用户选择仅活文档（非主仓库全部 md，非主仓库+归档） |
| REQ-3-3 | docs-lint 校验规则：代码符号、CLI 命令、配置项、文件路径 | 确认 | |
| REQ-3-4 | docs-lint 不一致时输出位置与原因，退出码非 0 | 确认 | |
| REQ-4-1 | 迁移不破坏现有构建、测试、typecheck | 确认 | |
| REQ-4-2 | 评估 spec-guardian skill 路径引用是否需同步调整 | 确认 | |
| REQ-4-3 | 评估测试中 docs/ 路径引用是否需同步调整 | 确认 | |

## 关键决策点

1. **归档引入方式**：pnpm workspace（非 git submodule，非主仓库内归档目录）
2. **迁移范围**：全部历史 spec（含 v1.1~v1.6、早期无版本号文档、common/ 下两个文档）
3. **TypeDoc 代码化范围**：仅 api.md（usage.md/custom-rules.md 保持手写 markdown）
4. **TypeDoc 产物位置**：docs/api/ 目录（多文件按模块拆分）
5. **docs-lint 集成方式**：独立 npm script（非集成到 quality，非 pre-commit hook）
6. **docs-lint 校验范围**：仅主仓库活文档
7. **common/ 文档处理**：都迁移归档（project-rules.md 与 typescript-migration.md 都迁移）

## 验收标准确认

- 归档仓库创建完成，含全部历史 spec
- 主仓库 pnpm workspace 引入成功
- 主仓库 docs/ 只保留活文档 + INDEX.md + api/ + v1.7 spec
- TypeDoc 生成 docs/api/ 目录
- docs/api.md 改为入口索引
- pnpm docs:lint 通过
- pnpm typecheck / build / test 全部通过
- spec-guardian skill 路径引用评估完成
- 测试中 docs/ 路径引用评估完成

## 签字确认

- 用户 / 产品负责人：creayma ________ 2026-06-24
- SOLO Coder：SOLO Coder ________ 2026-06-24

> 在 IDE 场景下，「签字」指用户明确回复「确认」「同意」。
