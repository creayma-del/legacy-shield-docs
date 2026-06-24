# @legacy-shield/docs

legacy-shield 历史 spec 归档仓库。

## 说明

- **归档仓库用途**：存放 legacy-shield 项目 v1.1~v1.6 及早期历史 spec 文档。
- **引入方式**：主仓库通过 git submodule 将本仓库嵌入 `docs-archive/` 目录，再以 pnpm workspace `@legacy-shield/docs` 包名引入。
- **访问路径**：主仓库中通过 `node_modules/@legacy-shield/docs/specs/` 访问（pnpm 软链接到 `docs-archive/specs/`）。
- **克隆方式**：`git clone --recurse-submodules <main-repo-url>`，或已克隆后执行 `git submodule update --init --recursive`。
- **冻结策略**：归档文档已冻结，不再修改。
