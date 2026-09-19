# 本 fork 做了什么（相对上游 v1.1.6）

- **上游基线**：[Max-Samson/dsh-usage-chart](https://github.com/Max-Samson/dsh-usage-chart) v1.1.6（commit `45d0671`，2026-09-19）。
- **性质与署名**：本仓库是上游的**非官方**维护分支，未获原作者背书；沿用上游 MIT 许可与版权署名（见 `LICENSE` / `THIRD_PARTY_NOTICES.md`）。
- **本仓库**：`rezon-aki/dsh-usage-chart`，默认分支 `dsh-0.1.5`；`main` 与上游一致，仅用于同步。
- **当前发布**：tag `v1.1.5-dsh.2`（上一版 `v1.1.5-dsh.1`）。
- **净变更**：见 [与上游的对比视图](https://github.com/Max-Samson/dsh-usage-chart/compare/main...rezon-aki:dsh-0.1.5)；相对 v1.1.6 全部集中在 `src/client/`（原来那一行 host 修复已随上游 v1.1.6 回流）。

---

## 一、兼容适配（不修就跑不起来的功能）

### 1. DSH ≥ 0.1.2：会话节点换了来源 → 「本轮 ≈ ¥0.0x」成本徽章消失

**现象**：每条助手消息尾部的「本轮 ≈ ¥0.0x」成本徽章不再出现；指示器行里的模型名消失；面板在拿不到宿主历史时，实时回退的轮次全部堆到 turn 0、没有模型与成本。

**根因**：DSH 0.1.2 起，chat 快照不再挂在会话快照上（0.1.5 的 `SessionSnapshot` 类型里已经没有 `chat` / `nodes` 字段），节点搬到了独立的 chat 槽位标准源，组件只能通过 `props.useChat` 读到。插件仍按老写法读 `session.chat.legacy.nodes`，于是`snapshotNodes()` 恒返回空数组。

**修法**：新增 `useSessionNodes(useChat, useSession)` —— 新源优先、旧路径保留回退（兼容老内核），两个 hook 无条件按固定顺序调用（否则 hook 数随数据变化会错位后续 hook 状态）。`CostBadge` 与 `UsageIndicator` 改走它。（提交 `f06cbeb`）

**影响面**：所有 DSH ≥ 0.1.2 的用户；上游 1.1.6 仍未修（已提 [PR #12](https://github.com/Max-Samson/dsh-usage-chart/pull/12)）。

### 2. 皮肤 backdrop-filter 改写 fixed 定位基准 → 面板跑到屏幕右侧

**现象**：装了 maid-atelier 皮肤时打开用量面板，面板整体偏到屏幕右侧（看起来像 x 坐标少除了一次）。

**根因**：`position: fixed` 的定位基准是「最近包含块」，而任一 `transform / filter / backdrop-filter / contain / container-type` 祖先都会把基准从视口改成它自己。该皮肤给 `[data-slot='conversation.composer.dock'] > *` 加了 `backdrop-filter: blur(2px)`，于是面板被按「指示器行的左边」解释了一遍视口坐标 —— 实际落点 ≈ 按钮左边 + 视口坐标。

**修法**：定位前查找最近的包含块，把视口坐标换算到它的坐标系（并修正窄窗口下的夹取范围）。（提交 `eeb09c5`）

---

## 二、缺陷修复

### 3. 价格覆盖文件（`pricing.json`）从未生效

**现象**：`~/.dsh/data/dsh-usage-chart/pricing.json` 写了不生效 —— `/dsh-usage-chart/pricing` 对每个模型都报 `source: "builtin"`；改文件不触发重载，变更监听形同虚设。

**根因**：`src/index.ts` 写成 `ctx.effect(() => fileSource.dispose(), …)`。表达式体箭头 = **注册时立刻调用** `dispose()`，并把 `undefined` 当 disposer 注册。文件源在被第一次读盘前就销毁了，`entries` 恒为 `{}`、watcher 也不会建立。

**修法**：改成返回 disposer（`ctx.effect(() => () => fileSource.dispose(), …)`）；测试里的 ctx mock 按 cordis 语义重写（执行 setup、保留返回的 disposer、在 `after()` 释放），并新增 `/pricing` 路由回归用例 —— **旧写法下该用例失败**，可当作修复的证据。（提交 `885b0dc`；已随上游 v1.1.6 合并收录，见 [PR #10](https://github.com/Max-Samson/dsh-usage-chart/pull/10)）

**影响面**：所有用户 —— 这个功能在上游 1.1.5 及更早版本里从未工作过（v1.1.6 已修复）。

### 4. 会话总计与逐轮之和对不上（按「当前时段」估算）

**现象**：指示器行 / 面板里的**会话总计**与每条消息尾部的**逐轮徽章之和**对不上 —— 高峰时段最多差一倍；同一会话在高峰时段打开会比空闲时段看到更大的总计。

**根因**：总计走的是「会话总量 × 刊例价」，而时段取的是**打开面板的那一刻**（`Date.now()`），不是各轮实际发生的时段；模型名在 DSH 0.1.2+ 上又取不到（快照节点已无 `provenance` / `requestConfig`），只能落到回退价。逐轮徽章则是宿主按**每轮开始时刻**折叠的，两者口径不同。

**修法**：总计改为「**Σ 各轮成本**」（新增 `sumRoundCosts()`，`src/client/rounds/types.ts`）—— 指示器行与面板共用，与逐轮徽章天然自洽；历史不可用时才退回原估算。顺带把模型名在快照取不到时的回退改为取**最后一轮历史**的模型（面板原有逻辑，指示器行补齐）。（v1.1.5-dsh.2）

---

## 三、新增功能

### 5. 用量面板可拖动 + 位置记忆 + 双击复位

- 面板顶部多了一条**把手**，按住即可拖动；拖动偏移持久化在 `localStorage` 的 `dsh-usage-chart:panel-pos`，下次打开沿用同一位置。
- **双击「用量」按钮**复位：面板回到按钮正上方的默认锚点（并清掉持久化偏移）。
- 未拖动时行为不变：面板贴在指示器行上方展开，随窗口 resize / 滚动重新锚定。（提交 `3a59781`）

---

## 四、维护

### 6. 内置定价表复核（2026-09-18）

逐项核对官方中英文定价页：`deepseek-flash` 空闲时段 CNY 1 / 0.02 / 4（高峰 ×2）、`deepseek-v4-pro` 空闲时段 CNY 4.5 / 0.15 / 13.5，USD 按官方英文页同口径。**数值与上游 2026-09-10 调价一致**，仅把核验时间戳更新到 2026-09-18。（提交 `c049c60`）

---

## 安装

```bash
# 方式一：从 GitHub 按 tag 安装（需本地构建能力）
npm i github:rezon-aki/dsh-usage-chart#v1.1.5-dsh.2

# 方式二：检出后 link: 到 DSH profile（本机当前用法）
git clone git@github.com:rezon-aki/dsh-usage-chart.git
cd dsh-usage-chart && npm ci && npm run build
# ~/.dsh/profiles/web/package.json → "dsh-usage-chart": "link:<绝对路径>"
# 并把 "dsh-usage-chart" 加进 dsh.profile.bundles，然后重启 profile
```

`lib/` 不随仓库跟踪（与上游一致），安装前必须 `npm ci && npm run build`。

## 与上游同步

```bash
git fetch upstream --tags
git rebase upstream/main          # 冲突通常只在 src/client/UsageIndicator.tsx（改动 2、5 都落在这里）
```

上游若合并了本仓库提的修复，rebase 时对应用 `git rebase --skip`（或 `git rebase --onto`）丢弃即可 —— [PR #10](https://github.com/Max-Samson/dsh-usage-chart/pull/10) 已随 v1.1.6 收录，对应提交 `885b0dc` 已在本次同步中丢弃。

## 验证

`npm ci` → `npm run typecheck` → `npm test`：**41/41 通过**（node 22，DSH 0.1.5-rc.2）。本机实测：

- `/dsh-usage-chart/pricing` 对 `deepseek-flash`、`deepseek-v4-pro` 报 `source: "file"`，`/usage` 各轮 `cost.source` 同为 `file`；
- 逐轮徽章与 DSH 自带「本轮用量」面板逐项一致（同一轮 miss/hit/out 完全相同，¥ 金额 = token × 官方刊例价）；
- 会话总计 = Σ 各轮成本（空闲时段与旧估算一致，高峰时段不再被当前时段放大一倍）。
