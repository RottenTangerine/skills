---
name: xhon-workflows
description: "EdenXiang engineering policies — branching strategy, branch plan format, merge flow, wayfinder-git integration, quality standards. Use BEFORE any git branch operation (checkout -b, merge, rebase, branch -d), /wayfinder, /grilling, /grill-with-docs, /implement, /to-spec, /to-tickets, /code-review, or /triage — load this skill first, then apply its policies when executing the target skill."
---

# Xhon Workflows

## When to load

Invoke this skill before any git branch operation or engineering skill. Identify your target from the table below, then **read every reference file listed in its row**. The table is a map, not the content — the policies live in the reference files.

| Target | Reference files to read |
|---|---|
| `git checkout -b`, `git merge`, `git rebase`, `git branch -d`/`-D` | [branching](references/branching.md) |
| `/wayfinder` | [branching](references/branching.md), [wayfinder-git](references/wayfinder-git.md), [quality](references/quality-standards.md) |
| `/grilling` / `/grill-with-docs` | [quality](references/quality-standards.md) |
| `/implement` | [branching](references/branching.md), [wayfinder-git](references/wayfinder-git.md), [quality](references/quality-standards.md) |
| `/to-spec` | [wayfinder-git](references/wayfinder-git.md) |
| `/to-tickets` | [wayfinder-git](references/wayfinder-git.md) |
| `/code-review` | [quality](references/quality-standards.md) |
| `/triage` | [wayfinder-git](references/wayfinder-git.md) |

## Reference files

- **[branching](references/branching.md)** — Three-tier model (main ← dev ← feature/*), branch naming, when to branch, session start, branch plan format and required commands, exceptions (no plan for read-only), protected branches, merge flow (feature→dev and dev→main, each a separate plan, `.scratch/` excluded at dev→main).
- **[wayfinder-git](references/wayfinder-git.md)** — The local `.scratch/<map-slug>/` effort layout (`issues/` decision tickets vs `implements/` tickets, prototype & implement worktrees), map creation, ticket resolution, research/prototype workflows (worktree commands, naming), the to-spec/to-tickets output locations, the implement workflow, `/implement` session start (finding the correct branch). Merge flow is in [branching](references/branching.md) — this file delegates to it.
- **[quality-standards](references/quality-standards.md)** — Structural clarity (deep modules, locality, seams), performance (no silent overhead, research subagent for debatable claims), production readiness (unhappy paths, observability, deployability), enterprise/industry norms, 6-point decision reference format for wayfinder/grilling.
