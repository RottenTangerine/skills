# Review — overall map review before spec

After every decision ticket resolves, review the whole map before the spec is written. This is a **conflict and gap check**, not a re-grilling.

## What to read

- `map.md` — Destination, Notes, Decisions so far, Not yet specified, Out of scope.
- Every ticket's `## Answer` (`.scratch/<map-slug>/issues/*.md`).
- Any research findings (`.scratch/<map-slug>/research/*.md`) and prototype conclusions linked from tickets.

## What to look for

1. **Conflicting decisions** — two answers that contradict each other, or whose implications (module boundaries, interfaces, data shapes) are incompatible. This is the primary risk after parallel resolution: subagents never saw each other, so butler must.
2. **Spec-relevant gaps** — decisions that leave the spec ambiguous: an unstated module boundary, an interface neither ticket pinned down, a seam the testing decisions depend on but no ticket resolved.
3. **Out-of-scope creep** — answers that wandered past the Destination; rule them out of scope per wayfinder rules (close + one line in the map's Out of scope).

## How to resolve

- **Conflicts** — analyse the two answers with a subagent (fresh, untainted by either resolution): which is compatible with more of the map? What's the smallest change to one answer that removes the conflict? Record the resolution in the ticket whose answer changed, update the map's Decisions-so-far gist.
- **Gaps** — fill them from the map's intent where the decision record supports it; otherwise surface to the user with the same one-batch need-user contract as ticket resolution.
- **Out-of-scope** — apply per wayfinder rules; no user call needed.

## Output

A short review note appended to the map's Notes:

```
## Review

- Conflicts: <none | n resolved — what changed>
- Gaps: <none | n filled — what was decided>
- Out of scope: <none | n ruled out>
```

This is the butler's synthesis point: after review, the map is internally consistent and ready to become a spec.
