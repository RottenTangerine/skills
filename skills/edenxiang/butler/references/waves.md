# Waves — decision-ticket fanout

The heart of butler: resolve every wayfinder decision ticket by **block waves of parallel subagents**. Each subagent owns exactly one ticket; subagents in the same wave run concurrently and never see each other. Tickets blocked on others wait for the wave whose answers unblock them.

## Building the graph

1. Read every open ticket in `.scratch/<map-slug>/issues/NN-<slug>.md`. Extract each `Type:` (`research`/`prototype`/`grilling`/`task`) and `Blocked by:` (the `NN` numbers it depends on).
2. Wave 1 = tickets with no open blockers. Wave k = tickets whose blockers all resolve in waves < k. A ticket whose blocker is `need-user` is deferred until the user answers and it resolves.

## Subagent prompt contract

One subagent per ticket. The prompt must be self-contained:

```
You are resolving wayfinder decision ticket <NN-<slug>> for effort <map-slug>.

- Ticket: .scratch/<map-slug>/issues/<NN>-<slug>.md
- Map (read its Notes + Decisions so far): .scratch/<map-slug>/map.md
- Branch: feature/<map-slug> — you may run in this checkout or a worktree; never merge, never push.

Resolve the ticket per its type:
- research: investigate against primary sources; write findings to .scratch/<map-slug>/research/<topic>.md
- prototype: if the question can be answered by automated verification (a runnable test, a typecheck, a quick benchmark), build it in a throwaway worktree and verify it. If it needs the human to evaluate a design (a UI to react to, a service to sign up for), start the dev server / open the service, and report need_user with the access path and a test checklist.
- grilling: analyse the decision with the six-point format (recommendation, trade-offs, code structure impact, production readiness, performance profile, standards alignment). The six-point analysis almost always converges on one defensible choice — take it, and record it as the answer. Only if there is a genuine fork with no defensible pick (a real either/or with materially different outcomes that only the user can weigh) mark need_user with your recommendation and the options.
- task: do the work if you can (AFK); if it needs the human (credentials, access, manual steps), mark need_user with a precise checklist.

Output STRICT JSON:
{ "answer": "<the decision or finding, one paragraph>", "need_user": <true|false>, "user_question": "<only if need_user: the decision with your recommendation and options>", "new_tickets": [{"title": "...", "type": "grilling|prototype|research|task", "blocked_by": [<NN numbers>]}], "scope_changes": [{"ticket": "<NN>", "out_of_scope": <true|false>}] }
```

## After each wave

1. For each subagent with a non-null answer: append `## Answer` to the ticket file, set `Status: resolved`, append a one-line gist + link to the map's Decisions-so-far. Commit to `feature/<map-slug>`.
2. Create any `new_tickets` as `.scratch/<map-slug>/issues/<NN>-<slug>.md` (next free numbers, blocking edges wired), add to the map's Not-yet-specified→tickets as appropriate. Commit.
3. Apply `scope_changes`: rule out-of-scope tickets per wayfinder rules (close + one line in the map's Out of scope section).
4. Collect `need_user` tickets. If any, surface them to the user **in one batch** (recommendation + options each). On answers: update the tickets, resolve, commit. Then continue with the next wave — including any tickets the user's answers newly unblocked.

## Serial rule

Never dispatch two subagents onto tickets that block each other in the same wave — the graph already prevents this. Within a wave, all tickets run concurrently; a `need_user` return does not pause the other subagents.
