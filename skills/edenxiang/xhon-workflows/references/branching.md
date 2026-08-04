# Branching Strategy

This repo uses a three-tier branching model:

```
main ─── ● ───── ● ─── (merge-only from dev, never commit directly)
           ↑       ↑
dev  ──── ● ── ● ── ● ──── (integration branch — all features branch from here)
            ↑      ↑
feature/* ── ● ── ●        (feature branches — branched from dev, merged back to dev)
```

All feature branches originate from `dev`.

## Branch naming

- Feature / fix / refactor branches: `feature/<kebab-case-slug>`
- The slug should be a short, English description of what the branch delivers

## When to create a new branch

- **Open a new branch**: new features, refactors, any change touching multiple files
- **Skip branching**: single-line typo fixes, README updates, trivial config tweaks (one commit tells the whole story)
- **When unsure**: ask the user, don't decide unilaterally

## Session start

At the start of every session that involves code changes, evaluate the current branch and the work ahead:
- If this is an independent feature/fix, create a new `feature/<slug>` branch from `dev`
- If continuing work on the current branch, stay put
- If currently on `main` or `dev`, switch to a new branch before making any changes

For any branch operation (checkout -b, merge, rebase, branch -d), output a **branch plan** and wait for confirmation.

## Branch plan

Before executing ANY of these commands, output a branch plan in this format and wait for user confirmation:

```
git checkout -b
git switch -c
git merge
git rebase
git branch -d
git branch -D
```

**Branch plan format:**

```
## Branch Plan

- **Current branch:** <current-branch>
- **Operation:** <create | merge | delete | rebase>
- **Target branch:** <target-branch-name>
- **Source branch (if creating):** dev (default) or <other>
- **Merge path:** <feature-branch> → dev → main
- **Delete source branch after completion:** <yes/no — ask user>
```

Only **one branch operation per plan**. If multiple branches need to be created or merged (e.g., create `dev`, then create `feature/<slug>` from `dev`), present them as separate plans — confirm and execute each one before presenting the next.

Then **wait for the user to confirm** before running any command. Do NOT execute without confirmation.

### Exceptions — no branch plan needed

Skip the branch plan for these read-only commands:

```
git branch          (list branches)
git branch -a       (list all branches including remote)
git branch -r       (list remote branches)
git status
git log
```

Everything else that writes to branches requires the plan.

## Protected branches

**main** and **dev** are protected. On protected branches:
- Never run `git commit`
- Changes only land via merge from a non-protected branch
- If the current session has already opened a worktree on main/dev, switch to a feature branch immediately and re-apply any pending edits there

## Merge flow

The repo is configured (by `/setup-matt-pocock-skills`) with `merge.ff false` and `pull.ff only`: every explicit merge creates a merge commit — preserving each branch line in `git log --graph` — while pulls fast-forward without adding noise merge commits. Per-merge message templates below; a descriptive message matters most where the default is uninformative.

Two hops, each a separate branch plan — never batch them into one plan:

### Feature → dev

When a feature is complete and ready to integrate:

1. Output the branch plan (merge variant)
2. After user confirms:
   - `git checkout dev && git pull origin dev` (ensure dev is current — `pull.ff only` updates without creating a merge commit)
   - `git merge <feature-branch>` — the default `Merge branch 'feature/<slug>'` message carries enough info (the slug identifies the feature)
   - Resolve any conflicts — if conflicts arise, pause and report each one
3. Ask the user: **"Delete `<feature-branch>` now?"** — never delete without asking
4. If yes: `git branch -d <feature-branch>` (use `-d`, not `-D` — refuse to force-delete if the branch isn't fully merged)

### Dev → main

When `dev` has accumulated one or more completed features and is ready to ship:

1. Confirm with the user that `dev` is ready to merge into `main` — describe what's on `dev` that hasn't been shipped yet (`git log main..dev --oneline`). Do NOT proceed without explicit user confirmation.
2. Output a separate branch plan (merge variant, `dev` → `main`)
3. After user confirms:
   - `git checkout main && git pull origin main` (ensure main is current — `pull.ff only`, no noise merge commit)
   - `git merge dev --no-commit -m "Merge dev → main: <release summary>"` — a descriptive message so each release point is identifiable in the log. `--no-commit` stages the merge without committing, so `.scratch/` can be excluded before the commit lands. **No tag** — the merge commit is the marker.
   - **Exclude `.scratch/`** — `git restore --source=HEAD --staged --worktree -- .scratch` (`.scratch/` is tracked on `dev` but must never reach `main`; `--source=HEAD` restores the staged tree and working tree to `main`'s state, which has no `.scratch/`)
   - `git commit` — finalizes the merge commit with the message prepared above
   - Resolve any conflicts — if conflicts arise, pause and report each one
4. Ask the user: **"Push `main`?"** — never push without asking
5. After merging, `git checkout dev` to continue work from the integration branch (`.scratch/` re-materializes on dev)

`.scratch/` is tracked on `feature/*` and `dev` (effort state: map, spec, tickets, workflow.js) but is excluded from `main` at this hop. `prototype/` and `worktrees/` under `.scratch/` are gitignored everywhere and never reach any branch. If this exclusion is ever forgotten, `.scratch/` lands on `main`; clean it up via a `feature/cleanup` branch (remove it, merge to `dev`, then a dev→main merge with the exclusion applied).
