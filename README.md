# EdenXiang Skills Fork

本仓库是 [mattpocock/skills](https://github.com/mattpocock/skills)（Matt Pocock 的 agent skills 集合）的一个 fork，用于个人使用：**跟随上游持续同步，同时保留个人定制**。

## 仓库结构

- `skills/engineering/`、`skills/productivity/` 等 —— **上游原版 skill，一律不改动**（改动会导致同步冲突）
- `skills/edenxiang/` —— **个人魔改区**：需要定制的 skill 从原目录复制一份到这里再修改。详见 [skills/edenxiang/README.md](./skills/edenxiang/README.md)
- 其余文件（`.claude-plugin/`、`docs/`、`.changeset/` 等）保持上游原样

## 分支与同步策略

```
main  ── 与 upstream/main 保持一致（上游快进，只通过 rebase 拉取）
edenxiang  ── 我的工作分支，持有全部魔改（新增文件 + scripts/link-skills.sh 修改 + README 重写）
```

| 分支 | 用途 | 同步方式 |
|---|---|---|
| `main` | 纯上游内容，作为同步的基准 | `git fetch upstream && git rebase upstream/main` |
| `edenxiang` | 我的所有魔改，日常在此工作 | 定期从 `main` rebase，解决冲突 |

### 日常使用

魔改只提交到 `edenxiang` 分支；`main` 分支保持纯上游内容，只被 upstream 更新。

### 同步上游更新

```bash
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main        # 使 fork 的 main 也同步

git checkout edenxiang
git rebase main             # 把魔改重放到新上游之上
# 解决冲突后：
git push origin edenxiang
```

rebase 后按 [skills/edenxiang/README.md](./skills/edenxiang/README.md) 中的「同步适配流程」，对照上游 `.changeset/` 的变更说明适配魔改的 skill。

## 本仓库对上游的改动（全部在 edenxiang 分支）

### 1. `scripts/link-skills.sh`

原脚本把所有 skill 按字母序链接到 `~/.claude/skills` 和 `~/.agents/skills`（后写覆盖先写）。由于 `edenxiang` 目录排在字母序最前，原版会覆盖魔改版。

修改后：**`skills/edenxiang/` 下的 skill 优先**，同名时跳过原目录的链接，无魔改的 skill 仍链接原版。详见脚本内注释。

### 2. `README.md`

完全重写（本文件），说明 fork 的同步与魔改策略。

### 3. `CLAUDE.md`

增加了 `edenxiang` bucket 的项目规则（见下方「给未来修改者的约定」）。

### 4. `skills/edenxiang/`

个人魔改 + 自创 skill 存放目录，见 [skills/edenxiang/README.md](./skills/edenxiang/README.md)。当前包含 24 个 skill：

- **魔改原版**（10 个）：`wayfinder`、`to-spec`、`to-tickets`、`implement`、`setup-matt-pocock-skills`、`grilling`、`grill-with-docs`、`code-review`、`resolving-merge-conflicts`、`triage`
- **原版复制**（12 个，仅为了插件独立）：`ask-matt`、`diagnosing-bugs`、`improve-codebase-architecture`、`tdd`、`prototype`、`research`、`domain-modeling`、`codebase-design`、`grill-me`、`handoff`、`teach`、`writing-great-skills`
- **自创**（2 个）：`xhon-workflows`（策略 skill：分支模型、.scratch 布局、质量标准）、`butler`（单次全自动执行器：`/butler work on <map-slug>` → 决策票波次 fanout → 整体审查 → spec → tickets → implement workflow）

核心变化：本地 `.scratch/` tracker 固定实现（无 GitHub issues）、`main ← dev ← feature/*` 分支模型（所有分支操作走分支计划+确认）、implement 带 worktree 隔离、triage 本地化、butler 全自动编排。

### 5. `.claude-plugin/plugin.json`

插件更名为 **`edenxiang-skills`**（marketplace: `edenxiang`）。`skills` 数组全部指向 `./skills/edenxiang/<name>`：魔改版路径替换原版（原版文件保留供同步对照），原版复制和自创 skill 直接加入。插件安装即魔改版生效。skills.sh 按此清单分组（插件名分组 + General）。

## 给未来修改者的约定

以下规则写入了项目 `CLAUDE.md`，在仓库内工作时会自动生效：

- `skills/edenxiang/` 是魔改 skill 的唯一存放处，直接改原目录 skill 是禁止的
- `edenxiang/` 的全部 skill（魔改版、原版复制、自创）都在 `.claude-plugin/plugin.json` 的 `skills` 数组里：魔改版路径替换为 `./skills/edenxiang/<name>`（原版文件保留），原版复制和自创 skill 直接加入
- 在 `edenxiang/` 新增 skill 时，必须同步更新 [skills/edenxiang/README.md](./skills/edenxiang/README.md) 的映射表
- 同步上游（rebase `edenxiang` 到新 `main`）后，必须按映射表逐个检查上游变更，并更新「上次对照上游版本」列

## 上游技能参考

完整的技能列表与使用说明请参阅上游原始文档（本仓库 `main` 分支的 `README.md` 即上游原版，或 [aihero.dev/skills](https://aihero.dev/skills)）。
