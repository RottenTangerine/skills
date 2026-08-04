# Edenxiang（个人魔改区）

本目录存放从上游复制出来、由我（EdenXiang）自行修改的 skill。**不随插件发布，不进入顶层 `README.md` 推广列表** —— 只通过 `scripts/link-skills.sh` 链接进本地 Claude Code / Codex 使用。

## 为什么需要这个目录

上游仓库（mattpocock/skills）持续更新。直接改原目录里的 skill，每次同步上游（rebase）都会产生冲突；且上游的 `scripts/link-skills.sh` 会把原版 skill 链接到本地，我的修改会被覆盖。

所以魔改一律复制到本目录进行。`link-skills.sh` 已修改为：**本目录下的 skill 优先**，同名时不再链接原版目录。

## 映射关系

| 本目录 skill | 上游来源（复制时的 commit） | 魔改内容摘要 | 上次对照上游版本 |
|---|---|---|---|
| wayfinder | `2ab9580` (upstream main) | 本地 `.scratch/` tracker 为固定实现（非 GitHub issues）；chart 时创建 `feature/<map-slug>`（走分支计划）；ticket 用 `Status:`/`Type:` 行、`Blocked by:` 行；引用 xhon-workflows | `2ab9580` |
| to-spec | `2ab9580` | 输出到 `.scratch/<map-slug>/spec.md`；无 triage label | `2ab9580` |
| to-tickets | `2ab9580` | 输出到 `.scratch/<map-slug>/implements/`（非 issues/）；ticket 模板加 `**Branch:**` 行 | `2ab9580` |
| implement | `2ab9580` | 增加 worktree 隔离流程（worktree-ticket-<n> 分支 + merge 回 feature）；session start 找分支；引用 xhon-workflows | `2ab9580` |
| setup-matt-pocock-skills | `2ab9580` | 合并 XhonSkills setup：.gitignore（.scratch/**/{prototype,worktrees} 排除）、worktree.bgIsolation none、merge.ff/pull.ff、dev 分支、.scratch 骨架；去掉 CLAUDE.md 指针注入；固定本地 tracker | `2ab9580` |
| grilling | `2ab9580` | 增加 6 点决策格式（推荐/权衡/代码结构/生产就绪/性能/标准）；引用 xhon-workflows | `2ab9580` |
| grill-with-docs | `2ab9580` | 引用 xhon-workflows 质量层 | `2ab9580` |
| code-review | `2ab9580` | spec 来源优先 `.scratch/<map-slug>/spec.md`；质量标准引用；去 GitHub 依赖 | `2ab9580` |
| resolving-merge-conflicts | `2ab9580` | 融合分支模型（ticket→feature 冲突以 wayfinder 决策记录为唯一依据；feature→dev/dev→main 报告用户）；引用 xhon-workflows | `2ab9580` |
| triage | `2ab9580` | 本地版：`Status:`/`Type:` 行状态机，无 GitHub labels/PRs；`## Triage Notes` 追加到文件 | `2ab9580` |
| xhon-workflows | 自创（无上游来源） | 策略 skill：branching（main←dev←feature/*、分支计划、merge 流程、.scratch 排除）、wayfinder-git（.scratch 布局、implements/ 独立）、quality-standards（6 点格式、质量标准） | — |

## 同步适配流程

上游更新后（`git fetch upstream` 并在 main 上 rebase），对新内容做适配：

1. 查看上游改动是否涉及我的魔改 skill：`git log upstream/main -- <原目录>`
2. 对照上游的 `.changeset/` 变更说明，判断行为变化
3. 若有影响，将改动合并进本目录对应 skill 的 SKILL.md
4. 更新上表「上次对照上游版本」列（填上游 commit）
5. 修改后重新运行 `scripts/link-skills.sh` 使本地链接生效

## 注意

- 本目录的 skill 不加入 `.claude-plugin/plugin.json`、不加入顶层 `README.md`、不加入 `docs/`（与 `misc/`、`personal/` 等非推广 bucket 一致）
- 复制某个 skill 进来时，记得同步更新上表
