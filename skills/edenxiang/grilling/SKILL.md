---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

> **EdenXiang fork:** invoke `/xhon-workflows` first — its [quality-standards](../xhon-workflows/references/quality-standards.md) reference defines the 6-point decision format this skill uses for every question.

Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a *fact* can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The *decisions*, though, are mine — put each one to me and wait for my answer.

Every question put to the user must include the **6-point decision format** (from quality-standards):

1. **Recommendation** — your recommended choice
2. **Trade-offs** — what we gain and give up vs. the runner-up alternative
3. **Code structure impact** — which modules change shape, which seams move
4. **Production readiness** — error handling, observability, deployment risk
5. **Performance profile** — what gets faster/slower, by roughly how much
6. **Standards alignment** — which enterprise/industry convention this follows

When a decision turns on a fact the environment can't resolve, spin up a `/research` subagent before putting the question to me — research feeds the thinking, it doesn't replace it.

Do not act on it until I confirm we have reached a shared understanding.
