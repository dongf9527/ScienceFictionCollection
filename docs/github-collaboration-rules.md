# GitHub 协作规则

> 适用范围：`ScienceFictionCollection` 及 `ScienceFictionCollection-Web` 两仓库的所有本地与远程协作
> 规则来源：仓库所有者提供（2026-10-31 会话记录）

## 核心规则总览

| # | 规则 | 说明 |
|---|------|------|
| 1 | 唯一同步方式 | 所有代码/文档改动通过 `git commit` + `git push` 推送到 GitHub，不在对话中凭空产生代码 |
| 2 | 仓库位置 | `/home/z/my-project/`（本地沙箱地址），不在其他目录创建 `.git` |
| 3 | 提交规范 | 小步提交，commit message 使用 Conventional Commits 风格（`feat:` / `fix:` / `docs:` / `refactor:` / `chore:` / `test:`） |
| 4 | 会话记录 | 分析/评估/测试记录写入 `docs/` 目录，以 Markdown 形式提交推送 |
| 5 | 禁止 force push | 绝不使用 `git push --force`；同理禁止对已推送 commit 执行 `--amend` 后再推送（等效 force，同样触犯本条）；未推送 commit 的 amend 属修正笔误常规手段，不受限 |
| 6 | Rebase 前置 | rebase 前先评估替代方案（revert/merge），必要时提前告知确认 |
| 7 | bug 修复不重命名变量 | 修复 bug 时保持原变量名，仅在 refactor 时重命名 |

## 附注

- 规则 2 中的 `<本地沙箱地址>` 已解析为 `/home/z/my-project/`，两仓库预期克隆位置为：
  - `/home/z/my-project/ScienceFictionCollection`
  - `/home/z/my-project/ScienceFictionCollection-Web`
- 本文件本身即规则 4 所指的会话记录文档，后续应随 `docs:` 类型 commit 推送至仓库的 `docs/` 目录。
