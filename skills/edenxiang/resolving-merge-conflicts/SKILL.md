---
name: resolving-merge-conflicts
description: "Use when you need to resolve an in-progress git merge/rebase conflict."
---

> **EdenXiang fork:** invoke `/xhon-workflows` first — its [branching](../xhon-workflows/references/branching.md) reference defines the branch model this skill operates within (feature → dev → main hops; ticket → feature hop for implement worktrees), and its [wayfinder-git](../xhon-workflows/references/wayfinder-git.md) reference points at the wayfinder decision records that are the only basis for judging ticket→feature conflicts.

1. **See the current state** of the merge/rebase. Check git history, and the conflicting files.

2. **Find the primary sources** for each conflict. Understand deeply why each change was made, and what the original intent was. Read the commit messages, check the PRs, check original issues/tickets — and for implement-worktree merges, the effort's wayfinder decision records under `.scratch/<map-slug>/` (map's Decisions-so-far, the ticket files' `## Answer` sections).

3. **Resolve each hunk.** Preserve both intents where possible. Where incompatible, pick the one matching the merge's stated goal and note the trade-off. Do **not** invent new behaviour. Always resolve; never `--abort`.

   Conflict scope matters: in a **ticket → feature** merge, the wayfinder decision records are the only basis for judgement — if a conflict can't be settled from those records, pause and report it to the user rather than guessing. In a **feature → dev** or **dev → main** merge, pause and report each conflict to the user (branching.md's rule).

4. Discover the project's **automated checks** and run them — typically typecheck, then tests, then format. Fix anything the merge broke.

5. **Finish the merge/rebase.** Stage everything and commit. If rebasing, continue the rebase process until all commits are rebased.
