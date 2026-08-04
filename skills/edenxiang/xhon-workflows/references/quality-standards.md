# Engineering Quality Standards

Every code change — whether produced by `/implement`, a wayfinder decision ticket, or a prototype-to-production handoff — must target these bars.

## Structural clarity

- **Deep modules, small interfaces** — a lot of behaviour behind a small surface. If a module exposes more than a handful of public methods, question whether it's doing one thing.
- **Locality** — related logic lives together; a change to one concern touches one file, not N.
- **Seam discipline** — every testable boundary is a deliberate seam. Prefer existing seams; introduce new ones only at the highest practical point. The `/codebase-design` skill defines the full module vocabulary.

## Performance

- **No silent overhead** — every allocation, blocking call, or loop that scales with input size must be intentional. Flag hot-path decisions with a rough order-of-magnitude estimate.
- **When a performance claim is debatable**, delegate fact-checking to a `/research` subagent before committing to the decision. Primary sources (docs, source code, benchmarks), not blog posts.

## Production readiness

- **Unhappy paths handled** — errors, timeouts, retries, backpressure. A change that only handles the happy path is not done.
- **Observable** — the change ships with the logging, metrics, or tracing needed to debug it in production.
- **Deployable** — migration ordering, rollback risk, and feature-flag gating are considered before merge.

## Enterprise / industry norms

- Follow the established patterns of the language, framework, and domain. A clever solution that fights the ecosystem's grain is usually wrong.
- When introducing a pattern new to the codebase, cite the convention it follows. If no convention exists, justify the departure in a comment or ADR.

## Decision quality (wayfinder / grilling)

When `/wayfinder` grills a decision or `/grilling` runs standalone, every question put to the user must include:

1. **Recommendation** — the agent's recommended choice
2. **Trade-offs** — what we gain and give up vs. the runner-up alternative
3. **Code structure impact** — which modules change shape, which seams move
4. **Production readiness** — error handling, observability, deployment risk
5. **Performance profile** — what gets faster/slower, by roughly how much
6. **Standards alignment** — which enterprise/industry convention this follows

When a decision turns on a fact the environment can't resolve, spin up a `/research` subagent before putting the question to the user — research feeds the thinking, it doesn't replace it.
