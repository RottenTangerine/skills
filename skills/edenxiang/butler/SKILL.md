---
name: butler
description: "Fully automate a wayfinder effort from charted map to shipped implementation. When the user says 'butler, work on <map-slug>' (or just '/butler work on <map-slug>'), resolve every wayfinder decision ticket by block waves of parallel subagents, surface only the decisions that genuinely need the user, then review, spec, ticket, and run the implement workflow to completion."
argument-hint: "work on <map-slug>"
---

# Butler

A **single-invocation, fully automated executor** for a wayfinder effort. The map is already charted (`.scratch/<map-slug>/map.md` + `issues/` decision tickets on `feature/<map-slug>`) — butler runs the rest: resolve every decision ticket, review the whole map, write the spec, break it into implement tickets, and drive the implementation to completion.

You are the butler. You orchestrate subagents and workflows; you never hold transcripts or long-lived state. There is no cross-session registry — one invocation does the whole effort. If the run is interrupted, re-invoking `/butler work on <map-slug>` resumes from whatever the tracker files say is open.

> **EdenXiang fork:** invoke `/xhon-workflows` first — its [wayfinder-git](../xhon-workflows/references/wayfinder-git.md) reference defines the `.scratch/` effort layout this skill operates on, and [branching](../xhon-workflows/references/branching.md) defines the branch model.

## Principles

- **Never fabricate state.** Read tracker files + git. If a check is ambiguous, ask the user.
- **Decisions are the user's.** Subagents may research, prototype, and analyse — but a decision that genuinely needs the human is surfaced, never invented. Default posture: the six-point analysis (recommendation, trade-offs, code structure impact, production readiness, performance, standards) almost always converges on one defensible choice — take it. Only when it genuinely branches (a real fork with no defensible pick, or a design call only the user can make) mark `need-user`.
- **Never stand in for the human.** A grilling agent that answers its own questions has broken this. But a subagent's *analysis* — researching, weighing options, recommending — is its job.
- **Block waves.** Tickets whose blockers aren't resolved are worked later, by subagents that can see the real answers.

## Invocation

`/butler work on <map-slug>` — where `<map-slug>` is the effort's slug. One invocation runs the whole pipeline:

1. **Resolve decision tickets** (waves, see [waves](references/waves.md)) — every open ticket in `.scratch/<map-slug>/issues/`, including any new tickets opened during resolution.
2. **Surface need-user decisions** — after all waves, present each one with its recommendation and options. Wait for answers; update the tickets; resolve any newly-unblocked tickets.
3. **Overall review** (see [review](references/review.md)) — read every resolved ticket's `## Answer`; look for conflicts between decisions, spec-relevant gaps, and out-of-scope creep.
4. **Spec + tickets** — run the spec synthesis (to-spec content) to `.scratch/<map-slug>/spec.md`, then break it into implement tickets at `.scratch/<map-slug>/implements/` (to-tickets content).
5. **Implement workflow** (see [workflow](references/workflow.md)) — generate `.scratch/<map-slug>/workflow.js` from the implement tickets' `Blocked by:` edges, run it with the **Workflow tool**, wave by wave, worktree-isolated implement agents, merge agent between waves.

## The phases in detail

### 1. Resolve decision tickets

Load the map. Read every open ticket in `issues/`. Build the **block wave graph** from each ticket's `Blocked by:` lines. Then:

- **Wave 1** — tickets with no open blockers. Spawn one subagent per ticket, in parallel (see [waves](references/waves.md) for the subagent prompt contract: structured output `{answer, need_user, new_tickets, scope_changes}`).
- **After each wave** — resolve the tickets the subagents answered: append `## Answer`, set `Status: resolved`, add the Decisions-so-far gist to the map. Create any new tickets the subagents surfaced, wire their blocking edges.
- **need-user tickets** — collect them. After all waves (or as soon as the next wave is blocked on them), surface them to the user **in one batch**, each with its recommendation and options. On answers, update the tickets and continue with any newly-unblocked work.
- **Next wave** — tickets whose blockers are now all resolved. Repeat until no tickets remain.

New tickets opened during resolution join the graph at the earliest wave where their blockers are resolved — the effort is only "done" when every ticket (original and new) is resolved.

### 2. Overall review

After all decision tickets resolve, review the whole map before writing the spec. Read every ticket's answer, the map's Decisions-so-far, and any research findings. Look for:

- **Conflicting decisions** — two answers that contradict each other or have incompatible implications.
- **Spec-relevant gaps** — decisions that leave the spec ambiguous (unstated module boundaries, unstated interfaces).
- **Out-of-scope creep** — answers that wandered past the destination; rule them out of scope per the wayfinder rules.

Resolve conflicts by re-grilling (subagent analysis, not invention); surface to the user only if a conflict genuinely needs their call. Record the review outcome in the map's Notes.

### 3. Spec + tickets

Run the spec synthesis to `.scratch/<map-slug>/spec.md` (the to-spec template: Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope, Further Notes). Then break the spec into implement tickets at `.scratch/<map-slug>/implements/NN-<slug>.md` (the to-tickets template: `**Branch:**`, `**What to build:**`, `**Blocked by:**`, `**Status:** ready-for-agent`, acceptance criteria). No user quiz for the breakdown — butler owns it; the user's involvement happened at decision time.

### 4. Implement workflow

Generate `.scratch/<map-slug>/workflow.js` from the implement tickets' blocking edges (topological waves, worktree-isolated agents, merge agent between waves — see [workflow](references/workflow.md)). Show the wave plan, then run it with the **Workflow tool**. One confirmation before running (it launches worktree-isolated implement agents and spends tokens).

## Interruption / resume

If the run is interrupted (user Ctrl-C, tool failure, session close): re-invoking `/butler work on <map-slug>` resumes from the tracker state — resolved tickets are `Status: resolved`, implemented tickets are `Status: done`, and the workflow re-generates only what's open. Never lose work: commit tracker state to `feature/<map-slug>` as you go (each resolved ticket, each spec, each ticket batch, the generated workflow.js).

## Watch-words

- Subagent prompts must be self-contained: point at the exact ticket path, the map path, and the effort branch. They work in worktrees or the main checkout; they never merge or push.
- `need-user` is the exception, not the rule. Most tickets resolve from analysis.
- The implement workflow runs on the **Workflow tool** — the script is generated, not hand-written. `meta` must be a pure literal.
