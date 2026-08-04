# 01 — mapping-table-column

## Question

`skills/edenxiang/README.md` 的映射表加一列「魔改原因」，让未来同步上游时更容易判断适配优先级。这个决策 ticket 要解决的：魔改原因列应该写什么粒度的信息？

## Type: grilling
## Status: resolved

## Answer

决策：**加两列** — 「魔改原因」（一行短句，如「XhonSkills 迁移」「本地化 tracker」「消除 CLAUDE.md 注入」）+「适配关注点」（上游更新时要重点检查什么）。理由：同步时 5 秒判断适配优先级，细节看 git diff。

