---
name: implement
description: "Implement a piece of work based on a spec or set of tickets, in a worktree-isolated session, merged back to the effort's feature branch."
disable-model-invocation: true
---

> **EdenXiang fork:** invoke `/xhon-workflows` first (its [wayfinder-git](../xhon-workflows/references/wayfinder-git.md) and [branching](../xhon-workflows/references/branching.md) references define the branch model and worktree lifecycle). This skill runs each implement ticket in its own **worktree** on a local-only `worktree-ticket-<n>` branch, then merges back to the effort's `feature/<map-slug>` branch. See "Worktree isolation" below.

## Setup

1. **Find the correct branch** (per wayfinder-git's `/implement` session start): read the ticket body for `**Branch:** feature/<map-slug>`; if present, `git checkout feature/<map-slug>` (no branch plan needed — the branch already exists). If the ticket has no branch field, read `map.md`'s `## Notes` for `Branch: feature/<map-slug>`. If neither is found, treat it as a standalone ticket and follow the normal session-start branching rules (create `feature/<slug>` from `dev` with a branch plan).
2. Ensure the effort branch is current: `git pull origin feature/<map-slug>` (or `git fetch` + check) if a remote exists.

## Worktree isolation

Each implement ticket runs in a **worktree** so parallel tickets don't conflict, and the main checkout stays clean:

1. **Create the worktree** — from the main checkout on `feature/<map-slug>`:

   ```bash
   git worktree add .scratch/<map-slug>/worktrees/ticket-<n> -b worktree-ticket-<n> feature/<map-slug>
   ```

   - Worktree directory: `.scratch/<map-slug>/worktrees/ticket-<n>` (gitignored, tracked nowhere)
   - Branch: `worktree-ticket-<n>` — **local-only, never pushed** (the global worktree rule)
   - **No branch plan needed** — the worktree branch is throwaway by definition and never merges to dev/main directly
   - `prototype/` and `worktrees/` under `.scratch/` are gitignored — the worktree itself is never committed

2. **Implement in the worktree** — `cd .scratch/<map-slug>/worktrees/ticket-<n>`, then:

   - Implement the work described by the ticket (or spec): read the ticket file at `.scratch/<map-slug>/implements/NN-<slug>.md` (or `spec.md` for the whole effort)
   - Use `/tdd` where possible, at pre-agreed seams
   - Run typechecking regularly, single test files regularly, and the full test suite once at the end
   - Commit your work to the current branch (`worktree-ticket-<n>`)
   - Mark the ticket `Status: done` in the ticket file and commit that too

3. **Merge back** — from the main checkout on `feature/<map-slug>`:

   ```bash
   git merge worktree-ticket-<n> -m "Merge ticket-<n>: <title>"
   ```

   - The repo's `merge.ff false` config guarantees a merge commit
   - **Conflicts**: resolve them using the wayfinder decision records as the only basis for judgement. If a conflict can't be settled from those records — a semantic disagreement the records don't cover — pause and report it to the user. Never guess.
   - After a clean merge: `git worktree remove .scratch/<map-slug>/worktrees/ticket-<n>` and `git branch -d worktree-ticket-<n>` (refuse `-D` if not fully merged)

## Review & commit

Once the work is merged to `feature/<map-slug>`, use `/code-review` to review the work (reviewing the diff since the merge-base of the feature branch). The feature branch then merges to `dev` and `main` per the branching model — both hops are always the human's, each with its own branch plan.
