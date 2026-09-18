# 本分支相对上游的改动（dsh-usage-chart v1.1.5 → dsh-0.1.5）

- **上游基线**：[Max-Samson/dsh-usage-chart](https://github.com/Max-Samson/dsh-usage-chart) v1.1.5（commit `dacc1f5`，2026-09-11）。
- **本仓库**：`rezon-aki/dsh-usage-chart`，默认分支 `dsh-0.1.5`；`main` 与上游一致，仅用于同步。
- **发布物**：tag `v1.1.5-dsh.1`（附注 tag，注释即改动清单）。

## 改动清单

| # | 提交 | 类型 | 解决什么 |
|---|---|---|---|
| 1 | `885b0dc` | fix(host) | **价格覆盖文件永远不生效**（上游同病）：`ctx.effect(() => fileSource.dispose(), …)` 是「调用」而非注册 disposer，文件源在 apply 阶段就被销毁 → `pricing.json` 不参与解析、变更监听失效。一行修复 + 回归用例（旧写法下该用例失败）。[上游 PR #10](https://github.com/Max-Samson/dsh-usage-chart/pull/10) |
| 2 | `f06cbeb` | fix(client) | **DSH ≥ 0.1.2 上会话节点取不到**：节点已搬到独立 chat 槽位源，旧读法 `session.chat.legacy.nodes` 恒为空数组 → 每轮成本徽章消失、dock 行模型名缺失、观测轮次回退失真。新增 `useSessionNodes(useChat, useSession)`（新源优先、旧路径回退，hook 顺序稳定）。 |
| 3 | `eeb09c5` | fix(client) | **悬浮面板定位到屏幕右侧**：fixed 定位的包含块被皮肤（maid-atelier 给 dock 子元素加 `backdrop-filter`）改写，原实现直接用视口坐标 → 整体偏移。改为按最近包含块换算，并修正窄窗口下的夹取范围。 |
| 4 | `3a59781` | feat(client) | **面板可拖动**：顶部把手拖动，位置持久化于 `localStorage`（`dsh-usage-chart:panel-pos`）；双击「用量」按钮复位到按钮正上方。 |
| 5 | `c049c60` | chore(pricing) | 内置定价表按官方中英文定价页复核（2026-09-18；数值与 2026-09-10 调价一致，仅更新核验时间戳）。 |

净变更：11 个文件，+226 / −27（`src/` 仅 6 个文件）。

## 安装

```bash
# 方式一：从 GitHub 按 tag 安装（需要本地构建能力）
npm i github:rezon-aki/dsh-usage-chart#v1.1.5-dsh.1

# 方式二：检出后用 link: 挂到 DSH profile（本机当前用法）
git clone git@github.com:rezon-aki/dsh-usage-chart.git
cd dsh-usage-chart && npm ci && npm run build
# ~/.dsh/profiles/web/package.json: "dsh-usage-chart": "link:<绝对路径>"
# 并把 "dsh-usage-chart" 加进 dsh.profile.bundles，然后重启 profile
```

`lib/` 不随仓库跟踪（与上游一致），安装前需 `npm ci && npm run build`。

## 与上游同步

```bash
git fetch upstream --tags
git rebase upstream/main          # 冲突通常只在 src/client/UsageIndicator.tsx（改动 3/4 落在这里）
# 或合并：git merge upstream/main
```

上游若合并了本仓库的某个修复（例如 [PR #10](https://github.com/Max-Samson/dsh-usage-chart/pull/10)），rebase 时该提交可用 `git rebase --skip` 丢弃。

## 验证

`npm ci` → `npm run typecheck` → `npm test`：41/41 通过（node 22，DSH 0.1.5-rc.2）。本机实测：`/dsh-usage-chart/pricing` 对 `deepseek-flash`、`deepseek-v4-pro` 报 `source: "file"`，`/usage` 各轮 `cost.source` 同为 `file`。
