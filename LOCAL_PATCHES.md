# dsh-usage-chart 本地补丁副本（link: 安装）

- **来源**：上游 `dsh-usage-chart@1.1.7`（commit `b19a851`，2026-09-24 发布）+ 本地补丁「面板可拖动」。
- **安装方式**：`~/.dsh/profiles/web/package.json` 里 `"dsh-usage-chart": "link:/home/rezon/dsh-workspace/dsh-plugins/dsh-usage-chart"`；
  `node_modules/dsh-usage-chart` 是指向本目录的符号链接；pnpm 重装不会覆盖本目录内的源码。
- **分支布局**：`main` = 旧本地线（1.1.5，tag `local/1.1.5-patched` 保留）；当前工作分支 `pr/panel-drag` = 上游 `b19a851`（v1.1.7）+ 拖动补丁——**目录里签出的是哪个分支，装的就是哪份代码**。
- **升级记录**：
  - 2026-09-18：并入上游 1.1.5（当时 5 项手改中 4 项上游已自带，本地补丁缩到 1 项）。
  - 2026-09-24：并入上游 1.1.7（`up/main` = `b19a851`）。此前 4 项本地补丁里 3 项已被上游收下（PR #12 chat 节点源、PR #14 包含块定位、PR #15 逐轮成本；价格文件 dispose 由 PR #10 收下），**只剩「面板可拖动」需要重放**——本轮按 v1.1.7 重写，不是旧补丁的逐行重放。
- **本地补丁（唯一一项）：面板可拖动**
  - `src/client/panel-position.ts`（新增）：持久化键 `dsh-usage-chart:panel-pos`、读取容错、按包含块夹取（`MIN_VISIBLE_PX = 48`）。
  - `src/client/UsageIndicator.tsx`：面板加 `transform: translate(offset)` + 顶部把手（pointer 拖动），双击「用量」复位；夹取基准复用与锚点同一个 `containingBlock()`（皮肤 transform / backdrop-filter 下不会算错）。
  - `src/client/i18n.ts`：`dragPanelTitle`（中英）。
  - `src/client/styles.ts`：`.duc-popover-handle`（sticky 顶部把手 + 抓取光标 + `touch-action: none`）。
  - `tests/panel-position.test.mjs`（新增）：持久化容错 + 夹取数学。
  - `CHANGELOG.md` / `CHANGELOG_ZH.md`：`[Unreleased] / [未发布]` 段已写好——**日后要提 PR，这段就是现成的发布说明**。
- **验证**（2026-09-24）：`npm run verify`（typecheck + build + 全部测试）通过；`lib/client.js` 已重建（link 安装即时生效，浏览器需刷新页面加载新 bundle）。
- **升级提醒**：再次升级上游前先备份本目录；升级后按本文件重放这一项，并把 CHANGELOG 的 `[未发布]` 段并入上游对应版本。
