---
name: research
description: Investigate a question against high-trust primary sources and capture the findings as a Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork delegated to a background agent.
---

> **EdenXiang fork:** when this research belongs to a wayfinder effort, the findings file lives at `.scratch/<map-slug>/research/<topic>.md` — tracked with the effort (committed to `feature/<map-slug>`, archived with it). See `/xhon-workflows`'s [wayfinder-git](../xhon-workflows/references/wayfinder-git.md) for the effort layout.

Spin up a **background agent** to do the research, so you keep working while it reads.

Its job:

1. Investigate the question against **primary sources** — official docs, source code, specs, first-party APIs — not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Write the findings to a single Markdown file, citing each claim's source.
3. Save it:

   - **Wayfinder research ticket** → `.scratch/<map-slug>/research/<topic>.md` — where `<map-slug>` is the effort's slug and `<topic>` a short kebab-case name (e.g. `<topic>` = `oauth-provider-options`). This location is tracked with the effort: committed to `feature/<map-slug>`, excluded from `main` at the dev→main hop, and archived with the effort. The wayfinder ticket's resolution links the file.
   - **Standalone research (no effort)** → match the repo's existing convention for such notes; if there is none, put it somewhere sensible and say where.
