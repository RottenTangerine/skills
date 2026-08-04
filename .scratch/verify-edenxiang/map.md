# Verify EdenXiang Fork

## 目的

验证魔改后的 pipeline skill（wayfinder → to-spec → to-tickets → implement）在本仓库可用。魔改要点：本地 `.scratch/` tracker、`feature/<map-slug>` 分支模型、implement worktree 隔离。

## 验证目标

1. wayfinder chart 创建 `.scratch/<map-slug>/map.md` + `issues/` 决策 ticket，记录 `Branch: feature/<map-slug>`
2. to-spec 输出 `.scratch/<map-slug>/spec.md`
3. to-tickets 输出 `.scratch/<map-slug>/implements/`（非 issues/）
4. implement 走 worktree 隔离流程（worktree-ticket-<n> 分支）实现 ticket

## 迷你 effort 主题

给本仓库的 `skills/edenxiang/README.md` 映射表加一列「魔改原因」（why 信息），让未来同步时更容易判断适配优先级。

## 流程

按魔改后的 skill 逐步执行，验证每个环节的输出位置和格式。

## Decisions so far

- [01 — mapping-table-column](./issues/01-mapping-table-column.md) — 映射表加「魔改原因」+「适配关注点」两列，一行短句粒度

