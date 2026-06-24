# @legacy-shield/docs

legacy-shield 历史 spec 归档仓库。

## 说明

- **归档仓库用途**：存放 legacy-shield 项目 v1.1~v1.6 及早期历史 spec 文档。
- **引入方式**：主仓库通过 pnpm git url 协议引用本仓库（`"@legacy-shield/docs": "git+https://github.com/creayma-del/legacy-shield-docs.git#main"`），`pnpm install` 时自动从 GitHub 拉取。
- **访问路径**：主仓库中通过 `node_modules/@legacy-shield/docs/specs/` 访问。
- **文档驱动流程**：本仓库更新文档 → push 到 GitHub → 主仓库执行 `pnpm update @legacy-shield/docs` 拉取最新 → 根据文档编码。
- **冻结策略**：归档文档已冻结，不再修改。
