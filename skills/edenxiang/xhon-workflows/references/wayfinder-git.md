# Wayfinder-Git Integration

Every wayfinder effort is a **local markdown effort** under `.scratch/<map-slug>/` on a single feature branch `feature/<map-slug>` from `dev`. The local tracker is the single source of truth — no GitHub issues. All work — decision tickets, spec, implementation — happens on this branch.

## Effort layout

```
.scratch/
├── archive/<map-slug>/      # completed efforts (git mv, keeps history)
└── <map-slug>/              # active effort
    ├── issues/              # wayfinder decision tickets: NN-<slug>.md (map's children)
    ├── implements/          # implement tickets from /to-tickets: NN-<slug>.md
    ├── map.md               # the wayfinder map
    ├── spec.md              # the spec (from /to-spec)
    ├── prototype/           # gitignored — prototype worktrees (never committed)
    │   └── proto-<name>/
    └── worktrees/           # gitignored — implement worktrees (never committed)
        └── ticket-<n>/
```

`.scratch/*/prototype/` and `.scratch/*/worktrees/` are gitignored (set by `/setup-matt-pocock-skills`); everything else under `.scratch/<map-slug>/` is **tracked** — on `feature/<map-slug>` and `dev`, and excluded from `main` at the dev→main merge (see [branching](branching.md)).

## Map creation (wayfinder chart)

1. Create `feature/<map-slug>` from `dev` — output a branch plan, wait for confirmation. **If the branch already exists** (e.g. created in an earlier session) and you are already on it, skip creation — no branch plan — and continue from step 2.
2. Ensure the effort directory exists: `mkdir -p .scratch/<map-slug>/{issues,implements,prototype,worktrees}`.
3. Record the branch name in the map's `## Notes`: `Branch: feature/<map-slug>`
4. Chart the map to `.scratch/<map-slug>/map.md`; create each decision ticket as one file under `.scratch/<map-slug>/issues/NN-<slug>.md` (blocking via `Blocked by: NN, NN` lines — see the local tracker doc's "Wayfinding operations" section).

**Done when:** `map.md` exists with `Branch: feature/<map-slug>` in its Notes, and its decision tickets are in `issues/`.

## Decision ticket resolution (wayfinder work)

1. Read `map.md`'s `## Notes` to find the feature branch name
2. Switch to that branch before touching any code (output a branch plan if switching)
3. Resolve the ticket per its type (grilling session, prototype, or research findings), then record `Status: resolved` and an `## Answer` heading in the ticket file, and append a one-line gist + link to the map's Decisions-so-far.

**Done when:** the ticket file is resolved, and the map's Decisions-so-far has the one-line gist + link.

## Research tickets (`wayfinder:research`)

Research branches (`research/<name>`) are throwaway — created during wayfinder charting to stage investigation findings, never merged. The findings are written as a Markdown file; the branch serves only to stage that file.

- **No branch plan needed** for any operation on a `research/<name>` branch — it is throwaway by definition and never merges
- Create: `git checkout -b research/<name> feature/<map-slug>`
- Write findings as a Markdown file
- Commit the findings file to the map's feature branch (NOT to the research branch)
- Delete: `git branch -D research/<name>` after the ticket is resolved (force delete — it was never meant to be merged)

## Prototype tickets (`wayfinder:prototype`)

Prototype worktrees are temporary — created from the feature branch to answer a design question, then deleted once the answer is recorded. They live **inside the effort directory** and are gitignored, throwaway, never merged, and always removed on archive.

### Naming

- **Worktree directory**: `.scratch/<map-slug>/prototype/proto-<short-name>` (e.g. `.scratch/user-auth/prototype/proto-login-ui`)
- **Temporary branch**: `feature/<map-slug>--proto-<short-name>` (e.g. `feature/user-auth--proto-login-ui`)
- Each prototype gets a unique branch name — this enables parallel prototypes from the same feature branch

### Create

```bash
git worktree add .scratch/<map-slug>/prototype/proto-<name> -b feature/<map-slug>--proto-<name> feature/<map-slug>
```

- The temporary branch starts at the current HEAD of the feature branch
- Register the worktree path and branch name in the prototype ticket or map Notes
- **No branch plan needed** — it is throwaway by definition and never merges

### Start dev server

In the worktree, run the project's dev server. Pick a port that doesn't conflict with other running prototypes or the main worktree.

### Clean up

After the prototype ticket is resolved (design decision recorded in the ticket):

```bash
git worktree remove .scratch/<map-slug>/prototype/proto-<name>    # remove worktree directory
git branch -D feature/<map-slug>--proto-<name>                     # delete temporary branch
```

- Use `git worktree remove` (not `rm -rf`) — it cleans up git's internal worktree registry
- Use `git branch -D` (force delete) — the temporary branch was never meant to be merged
- **No branch plan needed** — it's throwaway by definition

### Parallel prototypes

Multiple prototype worktrees can coexist. Each has a different directory, temporary branch, and dev server port. They do not interfere with each other or with the main feature branch worktree.

## Specification & implementation

- `/to-spec` runs on the map's feature branch → writes `.scratch/<map-slug>/spec.md`
- `/to-tickets` runs on the map's feature branch → **overrides the local-tracker default**: implement tickets go to `.scratch/<map-slug>/implements/NN-<slug>.md` (**NOT** `issues/` — that directory is reserved for wayfinder decision tickets). Each ticket body includes `**Branch:** feature/<map-slug>` — the single-source link so any session can find the branch without tracing through the map; the map's Notes remain the canonical reference, the ticket's field is a convenience copy

## Implement worktrees

Each implement ticket runs in its own **worktree** isolated from the main checkout, on a local-only `worktree-ticket-<n>` branch (the global worktree rule: `worktree-*` branches are never pushed). After the ticket is done, the worktree branch merges back into `feature/<map-slug>` and the worktree is removed. See `/implement` for the exact steps — this reference defines the naming and lifecycle:

- Worktree directory: `.scratch/<map-slug>/worktrees/ticket-<n>`
- Branch: `worktree-ticket-<n>` (local-only, never pushed)

## `/implement` session start — finding the correct branch

If an implement ticket is ever worked interactively (outside an automated workflow):

1. Read the ticket body first — look for `**Branch:** feature/<map-slug>`. If present, `git checkout feature/<map-slug>` (no branch plan needed — the branch already exists) and skip the rest of this checklist
2. If the ticket has no branch field but belongs to an effort, read `map.md`'s `## Notes` for `Branch: feature/<map-slug>`, then `git checkout feature/<map-slug>`
3. If neither is found: treat it as a standalone ticket and follow the normal session start branching rules
4. Once on the correct branch, proceed with `/implement` as normal

## Completion & archive

When all decision tickets and implement tickets are resolved and merged into `feature/<map-slug>`, the effort can be archived: remove prototype worktrees, `git mv .scratch/<map-slug> .scratch/archive/<map-slug>`. The feature branch merges to `dev` and then `main` per [branching](branching.md) — both hops are always the human's, and the dev→main hop excludes `.scratch/`.
