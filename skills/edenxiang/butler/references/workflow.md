# Implement workflow — `.scratch/<map-slug>/workflow.js`

The implement workflow is a **Workflow-tool script** that drives every open implement ticket as a worktree-isolated agent with the implement skill embedded in its prompt. Butler generates it, commits it to `feature/<map-slug>`, and runs it with the Workflow tool.

Workflow scripts are plain JS with two hard constraints that shape the generator:

- `meta` must be a **pure literal** — no variables, function calls, or interpolation. Phases are emitted as a static list by the generator.
- The script body **cannot read files** — so the ticket DAG and the implement-skill text are **baked in** at generation time. (The implement skill is `disable-model-invocation`, and `.agents/` is absent from worktrees, so its text must be inlined rather than referenced.)

## Generation

1. **Read the DAG.** For each open ticket in `.scratch/<map-slug>/implements/*.md`, parse `Blocked by:` (the `NN-<slug>` names it depends on).
2. **Topological waves.** Layer tickets into waves: wave 0 = no blockers; wave k = blocked only by tickets in waves < k. A ticket whose blocker is already done (or references nothing that exists) joins the earliest wave where all its blockers are done. A ticket blocked by an open ticket that never resolves stays out — report it, don't stall the run.
3. **Emit the script** (below), with:
   - `meta.phases` — one `{ title: 'Wave k' }` and one `{ title: 'Merge k' }` entry per wave, as a literal list.
   - `IMPLEMENT_SKILL` — the full text of `skills/edenxiang/implement/SKILL.md` (minus its frontmatter), as a template literal.
   - `WAVES` — the ticket list per wave, each entry `{ path, branch, title }`, `branch` = `worktree-ticket-<n>` numbered from the ticket's `NN-`.

## Script template

```js
export const meta = {
  name: 'implement-<map-slug>',
  description: 'Implement the open tickets for <map-slug> by block DAG',
  // one entry per wave + per merge, emitted literally by the generator
  phases: [
    { title: 'Wave 1' },
    { title: 'Merge 1' },
    { title: 'Wave 2' },
    { title: 'Merge 2' },
  ],
}

// Implement SKILL.md verbatim (disable-model-invocation; `.agents/` absent in worktrees)
const IMPLEMENT_SKILL = `…`

const WAVES = [
  [
    { path: '.scratch/<map-slug>/implements/01-<slug>.md', branch: 'worktree-ticket-1', title: '<title>' },
  ],
  [
    { path: '.scratch/<map-slug>/implements/02-<slug>.md', branch: 'worktree-ticket-2', title: '<title>' },
  ],
]

const implementPrompt = (t) => `你在一个 git worktree 里，分支是 ${t.branch}（local-only，永不 push/merge）。
读取 implement ticket：${t.path}。按它的要求实现。遵循以下 implement skill：

${IMPLEMENT_SKILL}

完成后：把代码 commit 到当前分支；把 ticket 文件（${t.path}）的 Status 置为 done，一并 commit。不要创建或切换分支，不要 merge，不要 push。输出 {ticket: "${t.path}", branch, sha}。`

const mergePrompt = (wave) => `你在主 checkout 的 feature/<map-slug> 分支上。把这波完成的 worktree 分支串行 merge 回当前分支：

${wave.map((t) => `${t.branch} — ${t.title}`).join('\n')}

逐个执行：git merge <branch> -m "Merge ticket-<n>: <title>"。冲突以 wayfinder 决策记录为唯一判断依据；解决不了就报告，绝不猜。合并干净后，确认该 ticket 文件 Status: done（没置就补上并 commit），然后 git worktree remove .scratch/<map-slug>/worktrees/ticket-<n> 和 git branch -d <branch>（未完全合并拒绝 -D）。逐张输出 {ticket, merged, sha}。`

for (let w = 0; w < WAVES.length; w++) {
  phase(`Wave ${w + 1}`)
  await parallel(WAVES[w].map((t) => () =>
    agent(implementPrompt(t), { isolation: 'worktree', label: t.branch })))
  phase(`Merge ${w + 1}`)
  await agent(mergePrompt(WAVES[w]), { label: `merge wave ${w + 1}` })
}
```

The barrier between waves is deliberate: the next wave's implement agents must derive their worktrees from the merged `feature/<map-slug>` HEAD, so each wave waits for the previous wave's merge agent to land.

## Run

Run the workflow with the Workflow tool on the generated script, after one confirmation. Implement agents derive from the current `feature/<map-slug>` HEAD; the merge agent lands each wave in the main checkout, so the next wave starts from the merged tree.

## Recovery

If a run stops mid-way: read `Status:` from `implements/*.md`. Tickets already `done` are skipped on the next generation, so a re-run only dispatches what's left. Unmerged `worktree-ticket-<n>` branches are merged by the merge agent on the next run (or manually per the branching rules).
