---
name: setup-matt-pocock-skills
description: Configure this repo for the EdenXiang engineering skills — local .scratch issue tracker, worktree isolation disabled, merge commit policy, dev branch, and domain doc layout. Run once per repo before first use of the other engineering skills.
disable-model-invocation: true
---

> **EdenXiang fork:** this skill sets up the **local `.scratch/` tracker** (fixed — no GitHub/GitLab), disables background worktree isolation, sets the merge commit policy, creates the `dev` branch, and writes the domain-doc layout. No CLAUDE.md pointer is written — the engineering policies live in the `/xhon-workflows` skill, which the skills reference directly.

# Setup Matt Pocock's Skills (EdenXiang fork)

Scaffold the per-repo configuration that the EdenXiang engineering skills assume:

- **Local issue tracker** — the `.scratch/` local-markdown tracker, fixed (no GitHub issues)
- **Worktree isolation** — background worktree isolation disabled (solo sequential workflows)
- **Merge commit policy** — repo-local `git config merge.ff false` + `pull.ff only`
- **`dev` branch** — the integration branch all feature branches originate from
- **Domain docs** — where `CONTEXT.md` and ADRs live, and the consumer rules for reading them

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `.gitignore` — does it exist? Which of these entries are already present: `.agents/`, `.claude/`, `CLAUDE.md`, `docs/agents/`, `skills-lock.json`? Is `.scratch/` ignored wholesale (old layout — must be migrated)?
- `.claude/settings.local.json` — does it exist? Does it already set `worktree.bgIsolation` to `"none"`?
- `CLAUDE.md` / `AGENTS.md` at the repo root — does either exist? Is there already an `## Agent skills` section in either?
- `CONTEXT.md` and `CONTEXT-MAP.md` at the repo root
- `docs/adr/` and any `src/*/docs/adr/` directories
- `docs/agents/` — does this skill's prior output already exist?
- `.scratch/` — sign that a local-markdown issue tracker convention is already in use
- `git config --get merge.ff` — is the repo-local merge fast-forward policy already `false`?
- `git config --get pull.ff` — is the repo-local pull policy already `only`?
- `git rev-parse --verify dev` — does the `dev` branch already exist?
- `git rev-parse HEAD` — is this a fresh repo with no commits yet? (If so, branch creation is skipped.)
- Monorepo signals — a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. Present only in a genuinely large multi-package repo; their absence means single-context, which is almost every repo.

### 2. Present findings and confirm

Summarise what's already in place and what will be added. One short summary, then ask:

> Proceed with these changes?

Lead with the list of changes so the user can accept in a word. Don't proceed without confirmation.

### 3. Apply

#### .gitignore

Read `.gitignore` if it exists. Append only the entries that aren't already present (exact line match):

```gitignore
# Coding agent artifacts
.agents/
.claude/
.scratch/**/prototype/
.scratch/**/worktrees/
CLAUDE.md
CONTEXT.md
docs/agents/
docs/research/
skills-lock.json
```

Note: `.scratch/` is **not** ignored wholesale — effort state under `.scratch/<map-slug>/` (map, spec, tickets) is tracked on `feature/*` and `dev`, while only `prototype/` and `worktrees/` stay ignored (the `**` covers both active efforts and anything under `.scratch/archive/`). **If a previous run added a `.scratch/` (whole-dir) line, replace it** with the two `.scratch/**/` entries above — otherwise the tracker never reaches git.

If `.gitignore` does not exist, create it with the entries above.

#### .scratch skeleton

Create the local issue-tracker root (idempotent):

```bash
mkdir -p .scratch
```

Per-effort directories (`.scratch/<map-slug>/{issues,implements,prototype,worktrees}`) are materialized by wayfinder at chart time; `.scratch/archive/` appears on the first archive. This step only ensures the root exists so the convention is uniform.

#### Worktree isolation

Read `.claude/settings.local.json` if it exists. If `worktree.bgIsolation` is not already set to `"none"`, merge it into the existing JSON (preserve all other settings, only add or update the `worktree` key):

```json
{
  "worktree": {
    "bgIsolation": "none"
  }
}
```

If `.claude/settings.local.json` does not exist, create it with the content above.

Briefly explain the trade-off to the user: with `bgIsolation: "none"`, parallel background tasks that touch the same files can conflict — but for sequential workflows (wayfinder, spec, tickets, implement one at a time), this risk doesn't apply.

#### dev branch

```bash
git checkout -b dev
```

Skip if `dev` already exists, or if the repo has no commits yet (fresh repo — the first commit will land on `dev`).

#### Merge commit policy

Set the repo-local merge policy so feature and ticket merges always produce a merge commit (topology stays visible in `git log --graph`), while pulls fast-forward without adding noise merge commits:

```bash
git config merge.ff false && git config pull.ff only
```

Skip any flag already set to its target value. **Repo-local only** (no `--global`) — the policy must not leak into unrelated repos. The per-merge message templates are documented in `/xhon-workflows`'s branching reference; this config is the fail-safe that a merge commit is created even if `--no-ff` is forgotten.

The dev→main hop **excludes `.scratch/`** (it is tracked on `feature/*`/`dev` but never reaches `main`) — the exact `--no-commit` + `git restore` + `commit` steps are in the branching reference; this setup only establishes the merge policy, the exclusion is applied at merge time.

#### Domain docs

Default to **single-context** — one `CONTEXT.md` + `docs/adr/` at the repo root. This fits almost every repo; write it without asking.

Offer **multi-context** — a root `CONTEXT-MAP.md` pointing to per-context `CONTEXT.md` files — only when exploration found monorepo signals. Then confirm which layout they want.

Write `docs/agents/domain.md` using the [domain.md](./domain.md) seed template as a starting point.

### 4. Done

Report what was changed:

- `.gitignore` — [created / appended N entries / migrated `.scratch/` to `.scratch/**/{prototype,worktrees}` / already complete]
- Worktree isolation — [created settings.local.json / added bgIsolation key / already set]
- `.scratch/` skeleton — [created / already exists]
- `dev` branch — [created / already existed / skipped — fresh repo]
- Merge commit policy — [set `merge.ff false` + `pull.ff only` / already set]
- Domain docs — [wrote docs/agents/domain.md / already present]

Mention that the agent will now load `/xhon-workflows` before any git branch operation, wayfinder, grilling, implement, to-spec, to-tickets, code-review, and triage. Re-running this skill is safe — each step checks before writing.
