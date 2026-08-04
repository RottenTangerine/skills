---
name: triage
description: Move issues through a state machine of triage roles on the local .scratch tracker — categorise, verify, grill if needed, and write agent-ready briefs.
disable-model-invocation: true
---

> **EdenXiang fork:** this skill runs on the **local `.scratch/` tracker** — issues are markdown files, and triage state is a `Status:` line in each file (no GitHub labels, no PRs). Invoke `/xhon-workflows` first; its [wayfinder-git](../xhon-workflows/references/wayfinder-git.md) reference defines the tracker layout this skill operates on.

# Triage

Move issues on the local tracker through a small state machine of triage roles. An issue is a markdown file under `.scratch/<map-slug>/` — the triage state is its `Status:` line near the top of the file. This skill has no GitHub/GitLab scope — there are no external PRs, no labels, no comments to post.

## Reference docs

- [AGENT-BRIEF.md](AGENT-BRIEF.md) — how to write durable agent briefs
- [OUT-OF-SCOPE.md](OUT-OF-SCOPE.md) — how the `.out-of-scope/` knowledge base works

## Roles

Two **category** roles:

- `bug` — something is broken
- `enhancement` — new feature or improvement

Five **state** roles:

- `needs-triage` — maintainer needs to evaluate
- `needs-info` — waiting on reporter for more information
- `ready-for-agent` — fully specified, ready for an AFK agent
- `ready-for-human` — needs human implementation
- `wontfix` — will not be actioned

Every triaged issue should carry exactly one category role and one state role. If state roles conflict, flag it and ask the maintainer before doing anything else.

In the local tracker, roles are expressed as lines near the top of the issue file: `Type: bug|enhancement` and `Status: needs-triage|needs-info|ready-for-agent|ready-for-human|wontfix`.

State transitions: an unlabeled issue normally goes to `needs-triage` first; from there it moves to `needs-info`, `ready-for-agent`, `ready-for-human`, or `wontfix`. `needs-info` returns to `needs-triage` once the reporter replies. The maintainer can override at any time — flag transitions that look unusual and ask before proceeding.

## Invocation

The maintainer invokes `/triage` and describes what they want in natural language. Interpret the request and act. Examples:

- "Show me anything that needs my attention"
- "Let's look at ticket 3" (path `.scratch/<map-slug>/issues/03-<slug>.md`)
- "Move ticket 3 to ready-for-agent"
- "What's ready for agents to pick up?"

## Show what needs attention

Scan the tracker and present three buckets, oldest first:

1. **Unlabeled** — never triaged (no `Status:` line).
2. **`needs-triage`** — evaluation in progress.
3. **`needs-info` with reporter activity since the last triage notes** — needs re-evaluation.

Show counts and a one-line summary per item. Let the maintainer pick.

## Triage a specific issue

1. **Gather context.** Read the full issue file (body, `## Comments` history, author, dates). Parse any prior triage notes so you don't re-ask resolved questions. Explore the codebase using the project's domain glossary, respecting ADRs in the area. Run two checks against the codebase: (a) **redundancy** — search for an existing implementation of the requested behavior by domain concept (not just the request's wording), and report where you looked. If found, it's an already-implemented `wontfix` (step 5). (b) **prior rejection** — read `.out-of-scope/*.md` and surface any that resembles this request.

2. **Recommend.** Tell the maintainer your category and state recommendation with reasoning, plus a brief codebase summary relevant to the request — including whether it's already implemented. Wait for direction.

3. **Verify the claim.** Before any grilling, check that the claim holds up. For a bug, reproduce it from the reporter's steps. Report what happened: confirmed (with code path), failed, or insufficient detail (a strong `needs-info` signal). A confirmed verification makes a much stronger agent brief.

4. **Grill (if needed).** If the request needs fleshing out, run the `/grilling` and `/domain-modeling` skills together — grill it into shape one question at a time, sharpening domain terms and updating `CONTEXT.md`/ADRs inline as decisions land.

5. **Apply the outcome** — update the `Status:` line in the issue file (and `Type:` if setting a category):
   - `ready-for-agent` — write an agent brief into the issue file ([AGENT-BRIEF.md](AGENT-BRIEF.md)).
   - `ready-for-human` — same structure as an agent brief, but note why it can't be delegated (judgment calls, external access, design decisions, manual testing).
   - `needs-info` — append triage notes to the file under a `## Triage Notes` heading (template below).
   - `wontfix` — set `Status: wontfix`, with an appended note depending on *why*:
     - **Already implemented** — the change already exists in the codebase. Point to where it lives; do **not** write to `.out-of-scope/` (that KB is for *rejected* requests, not built ones).
     - **Rejected (bug)** — polite explanation appended to the file.
     - **Rejected (enhancement)** — write to `.out-of-scope/`, link to it from the file, then set `wontfix` ([OUT-OF-SCOPE.md](OUT-OF-SCOPE.md)).
   - `needs-triage` — set the status. Optional note if there's partial progress.

## Quick state override

If the maintainer says "move ticket 3 to ready-for-agent", trust them and apply the role directly. Confirm what you're about to do (status change, appended content), then act. Skip grilling. If moving to `ready-for-agent` without a grilling session, ask whether they want to write an agent brief.

## Needs-info template

```markdown
## Triage Notes

**What we've established so far:**

- point 1
- point 2

**What we still need from you (@reporter):**

- question 1
- question 2
```

Capture everything resolved during grilling under "established so far" so the work isn't lost. Questions must be specific and actionable, not "please provide more info".

## Resuming a previous session

If prior triage notes exist in the issue file, read them, check whether the reporter has answered any outstanding questions, and present an updated picture before continuing. Don't re-ask resolved questions.
