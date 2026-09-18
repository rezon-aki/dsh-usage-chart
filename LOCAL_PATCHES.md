# 本分支相对上游的改动（dsh-usage-chart v1.1.5 → dsh-0.1.5）

- **基线**：上游 [Max-Samson/dsh-usage-chart](https://github.com/Max-Samson/dsh-usage-chart) v1.1.5（commit `dacc1f5`，2026-09-11）。
- **本分支**：`rezon-aki/dsh-usage-chart` → `dsh-0.1.5`（只含下列改动，main 保持与上游一致）。
- **改动清单**（按提交顺序）：
  1. `fix(host)` 价格覆盖文件源在 apply 阶段被销毁：`ctx.effect(() => fileSource.dispose(), …)` 是「调用」而非注册 disposer，`pricing.json` 永不参与解析、变更监听失效（**上游 v1.1.5 同病**）。一行修复 + `/pricing` 回归用例（旧写法下该用例失败）。
  2. `fix(client)` DSH ≥ 0.1.2 的会话节点已搬到独立 chat 槽位源：原读法 `session.chat.legacy.nodes` 恒为空数组，导致每轮成本徽章消失、dock 行模型名缺失、观测轮次回退失真。新增 `useSessionNodes(useChat, useSession)`（新源优先、旧路径回退，两个 hook 无条件调用）。
  3. `fix(client)` 悬浮面板按 fixed 包含块换算锚点：皮肤（maid-atelier 给 `[data-slot='conversation.composer.dock'] > *` 加 `backdrop-filter`）下定位基准是祖先而非视口，原实现会把面板推到屏幕右侧。
  4. `feat(client)` 面板可拖动（顶部把手，位置持久化 `dsh-usage-chart:panel-pos`），双击「用量」复位到按钮正上方。
  5. `chore(pricing)` 内置定价表按官方中英文定价页复核（2026-09-18；数值与上游 2026-09-10 调价一致，仅更新核验时间戳）。
- **安装**：`npm ci && npm run build`，随后在 DSH profile（`~/.dsh/profiles/web/package.json`）里把 `dsh-usage-chart` 指向本仓库检出（`link:<path>`）并加入 `dsh.profile.bundles`。
- **验证**：`npm run typecheck` 通过；`npm test` 41/41 通过（含新增回归用例）。
