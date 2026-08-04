# Edenxiang（个人魔改区）

本目录存放从上游复制出来、由我（EdenXiang）自行修改的 skill，以及自创 skill。**随插件发布**（`.claude-plugin/plugin.json` 的 `skills` 数组全部指向本目录），也通过 `scripts/link-skills.sh` 链接进本地 Claude Code / Codex 使用。

## 为什么需要这个目录

上游仓库（mattpocock/skills）持续更新。直接改原目录里的 skill，每次同步上游（rebase）都会产生冲突；且上游的 `scripts/link-skills.sh` 会把原版 skill 链接到本地，我的修改会被覆盖。

所以魔改一律复制到本目录进行。`link-skills.sh` 已修改为：**本目录下的 skill 优先**，同名时不再链接原版目录。

## 映射关系

| 本目录 skill | 上游来源（复制时的 commit） | 魔改内容摘要 | 魔改原因 | 适配关注点 | 上次对照上游版本 |
|---|---|---|---|---|---|
| wayfinder | `2ab9580` (upstream main) | 本地 `.scratch/` tracker 为固定实现（非 GitHub issues）；chart 时创建 `feature/<map-slug>`（走分支计划）；ticket 用 `Status:`/`Type:` 行、`Blocked by:` 行；引用 xhon-workflows | 本地化 tracker + 分支模型 | 上游若改 tracker 抽象或 chart 流程，需重放 | `2ab9580` |
| to-spec | `2ab9580` | 输出到 `.scratch/<map-slug>/spec.md`；无 triage label | 本地化输出位置 | 上游若改发布位置，需重放 | `2ab9580` |
| to-tickets | `2ab9580` | 输出到 `.scratch/<map-slug>/implements/`（非 issues/）；ticket 模板加 `**Branch:**` 行 | 本地化输出 + 分支链接 | 上游若改 ticket 模板或发布逻辑，需重放 | `2ab9580` |
| implement | `2ab9580` | 增加 worktree 隔离流程（worktree-ticket-<n> 分支 + merge 回 feature）；session start 找分支；引用 xhon-workflows | worktree 隔离 | 上游若改实现流程，需重放 | `2ab9580` |
| setup-matt-pocock-skills | `2ab9580` | 合并 XhonSkills setup：.gitignore（.scratch/**/{prototype,worktrees} 排除）、worktree.bgIsolation none、merge.ff/pull.ff、dev 分支、.scratch 骨架；去掉 CLAUDE.md 指针注入；固定本地 tracker | 消除 CLAUDE.md 注入 + 本地化 | 上游若改 setup 流程或新增配置项，需重放 | `2ab9580` |
| grilling | `2ab9580` | 增加 6 点决策格式（推荐/权衡/代码结构/生产就绪/性能/标准）；引用 xhon-workflows | 决策质量提升 | 上游若改 grilling 协议，需重放 | `2ab9580` |
| grill-with-docs | `2ab9580` | 引用 xhon-workflows 质量层 | 质量层接入 | 上游若改此 skill，需检查引用 | `2ab9580` |
| code-review | `2ab9580` | spec 来源优先 `.scratch/<map-slug>/spec.md`；质量标准引用；去 GitHub 依赖 | 本地化 spec 来源 | 上游若改 spec 查找顺序，需重放 | `2ab9580` |
| resolving-merge-conflicts | `2ab9580` | 融合分支模型（ticket→feature 冲突以 wayfinder 决策记录为唯一依据；feature→dev/dev→main 报告用户）；引用 xhon-workflows | 分支模型融合 | 上游若改冲突解决流程，需重放 | `2ab9580` |
| triage | `2ab9580` | 本地版：`Status:`/`Type:` 行状态机，无 GitHub labels/PRs；`## Triage Notes` 追加到文件 | 本地化状态机 | 上游若改 triage 角色/流程，需重放 | `2ab9580` |
| ask-matt | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| diagnosing-bugs | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| improve-codebase-architecture | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| tdd | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| prototype | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| research | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| domain-modeling | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| codebase-design | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| grill-me | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| handoff | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| teach | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| writing-great-skills | `2ab9580` | 原版复制，未魔改 | 插件独立 | 无 | `2ab9580` |
| xhon-workflows | 自创（无上游来源） | 策略 skill：branching（main←dev←feature/*、分支计划、merge 流程、.scratch 排除）、wayfinder-git（.scratch 布局、implements/ 独立）、quality-standards（6 点格式、质量标准） | 自创策略层 | — | — |
| butler | 自创（重写自 XhonSkills butler） | 单次全自动执行器：`/butler work on <map-slug>` → 决策票波次 fanout（subagent 并行，need-user 阶段后传达）→ 整体审查 → spec → tickets → 生成 workflow.js 跑 implement。不含跨会话 registry | 自创编排层 | — | — |

## 同步适配流程

上游更新后（`git fetch upstream` 并在 main 上 rebase），对新内容做适配：

1. 查看上游改动是否涉及我的魔改 skill：`git log upstream/main -- <原目录>`
2. 对照上游的 `.changeset/` 变更说明，判断行为变化
3. 若有影响，将改动合并进本目录对应 skill 的 SKILL.md
4. 更新上表「上次对照上游版本」列（填上游 commit）
5. 修改后重新运行 `scripts/link-skills.sh` 使本地链接生效

## 注意

- 本目录的 skill 全部加入 `.claude-plugin/plugin.json`（魔改版路径替换原版；自创 skill 直接加入），不加入 `docs/`（与 `misc/`、`personal/` 等非推广 bucket 一致）
- 复制某个 skill 进来时，记得同步更新上表
