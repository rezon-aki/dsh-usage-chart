# 本 fork 做了什么

> 写法约定：先说「用户会遇到什么问题、修完变成什么样」，再说技术根因 —— README、CHANGELOG、PR 文案一律按这个顺序写。

- **上游基线**：[Max-Samson/dsh-usage-chart](https://github.com/Max-Samson/dsh-usage-chart) v1.1.7（commit `b19a851`，2026-09-24）。
- **性质与署名**：本仓库是上游的**非官方**维护分支，未获原作者背书；沿用上游 MIT 许可与版权署名（见 `LICENSE` / `THIRD_PARTY_NOTICES.md`）。
- **本仓库**：`rezon-aki/dsh-usage-chart`；默认分支 `dsh-0.1.5`（= 上游 v1.1.7 + 下列 fork 改动），`main` 与上游一致、只用于同步。
- **发布**：分支即发布（当前不打 tag）；`package.json` 版本跟随上游（1.1.7）。

## fork 独有的改动

### 面板可拖动（`src/client/panel-position.ts` + `UsageIndicator.tsx`）

- **现象**：面板固定在「用量」按钮正上方，内容一多或窗口一窄就会挡住正文，除了收起来没有别的办法；开着皮肤（dock 带模糊 / 变换）时可用空间更小。
- **修法**：面板顶部加拖动把手，按住拖动即改变位置，偏移写入 `localStorage`（键 `dsh-usage-chart:panel-pos`）；双击「用量」按钮复位到按钮正上方。拖动只做 `transform: translate()` 相对位移，边界用与锚点同一个 `containingBlock()` 夹取（即上游 PR #14 的包含块换算），面板四条边至少保留 48px 在可视区内。
- **上游状态**：已提 issue [#16](https://github.com/Max-Samson/dsh-usage-chart/issues/16)；对方要的话以 PR 形式回收（内容 = `src/` + 双语 CHANGELOG + `tests/panel-position.test.mjs`，**不含生成的 `lib/`**）。

## 已并入上游、不再由 fork 维护（保留追溯）

| 用户会遇到的现象 | 上游落地 |
|---|---|
| 升级 DSH 0.1.2+ 后成本徽章 / 模型名消失 | 上游 PR #12（v1.1.7） |
| 装了皮肤 / 主题后面板偏到屏幕一边 | 上游 PR #14（v1.1.7） |
| 「会话总计」与逐轮徽章之和对不上 | 上游 PR #15（v1.1.7） |
| `pricing.json` 自定义价格不生效 | 上游 PR #10（v1.1.6） |
| PR #15 合并时被覆盖的 `useChat` 接线 | 上游提交 `9753856`（v1.1.7） |

## 维护方式

- 上游每发一版，把 fork 改动 rebase 到新的上游 `main` 上（目前只剩「面板可拖动」一项），跑 `npm run verify` 后 force-push 默认分支。
- 验证记录：拖动补丁在上游 v1.1.7 基线上 `npm run verify` 通过 50/50（上游 46 项 + 新增 4 项）。
