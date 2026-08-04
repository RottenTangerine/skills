# Verify EdenXiang Fork Pipeline

## Problem Statement

魔改后的 pipeline skill（wayfinder → to-spec → to-tickets → implement）需要验证其在本仓库（skills 仓库自身）可用。验证的要点是本地 `.scratch/` tracker 布局、`feature/<map-slug>` 分支模型、implement worktree 隔离。

## Solution

在 `feature/verify-edenxiang` 分支上跑一个迷你 effort：给 `skills/edenxiang/README.md` 映射表加「魔改原因」+「适配关注点」两列。逐步验证 wayfinder chart、to-spec、to-tickets、implement 的输出位置和格式。

## User Stories

1. 作为仓库维护者，我想 wayfinder 在本地 `.scratch/<map-slug>/` 创建 map 和决策 ticket，以便不依赖 GitHub issues。
2. 作为仓库维护者，我想 to-spec 把 spec 写到 `.scratch/<map-slug>/spec.md`，以便随 feature 分支跟踪。
3. 作为仓库维护者，我想 to-tickets 把 implement tickets 写到 `.scratch/<map-slug>/implements/`，以便与 wayfinder 决策 tickets 分离。
4. 作为仓库维护者，我想 implement 在独立 worktree 中实现每个 ticket，以便并行任务不冲突。
5. 作为仓库维护者，我想映射表有「魔改原因」+「适配关注点」两列，以便同步上游时快速判断适配优先级。

## Implementation Decisions

- 映射表加「魔改原因」+「适配关注点」两列（一行短句粒度），细节看 git diff。
- 验证 effort 不实际改动任何 skill 正文，只改 `skills/edenxiang/README.md` 的映射表。
- `.scratch/verify-edenxiang/` 是验证产物，随 feature 分支跟踪，验证完成后保留作示例。

## Testing Decisions

- 验证不依赖自动化测试——检查产物文件的位置、格式、分支提交历史。
- 验证重点是 pipeline 各环节的输出位置（map.md、spec.md、implements/、worktree 分支）。

## Out of Scope

- 不验证 butler 编排器（未迁移）。
- 不验证 setup skill 的完整配置写入（.gitignore 会排除 CLAUDE.md，不适合 plugin 仓库，用手动最小配置替代）。

## Further Notes

- 验证完成后，feature/verify-edenxiang 分支的产物可保留作示例，或合并回 edenxiang 分支后清理。
