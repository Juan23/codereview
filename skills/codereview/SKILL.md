---
name: engineering-bar
description: Machine-global toggle for strict software-correctness invariants enforced at acceptance and final review.
---

# $engineering-bar

This is a machine-global toggle, not a chat-only setting.

State is stored in:

```text
~/.codex/.engineering-bar-off
```

The default is **ON**.

- `$engineering-bar on` removes the sentinel.
- `$engineering-bar off` creates the sentinel.
- `$engineering-bar status` reports `engineering-bar: ON` or `engineering-bar: OFF`.

When the state is **OFF**, do not apply this checklist unless the user explicitly asks for an engineering-bar review.

## Mandatory law when enabled

When this toggle is **ON**, the engineering bar is a binding command. Apply
every invariant relevant to the changed behavior as an acceptance and final-
review gate. Do not waive, skip, weaken, reinterpret, or substitute an
applicable invariant because it is inconvenient, absent from existing tests,
or difficult to prove. A violation is a hard failure: stop acceptance, state
the violated invariant or proof gap, and reroute the task through the JM
`codex-pipeline` workflow. The **OFF** sentinel remains the only opt-out unless
the user explicitly requests an engineering-bar review.

# Loading policy

The full bar is an **acceptance/final-review gate**, not routine worker context. Do not copy this whole list into implementation prompts.

Before implementation, the orchestrator may pass only the specific invariant(s) that materially constrain the bounded task. This is required when correctness depends on a rule the worker could not safely infer from local code alone.

At final review, check behavior introduced or changed by this task against every applicable invariant. Findings may rely on unchanged coupled code when the task makes that code incompatible, unsafe, or incorrect. Do not report unrelated pre-existing violations.

Applicable invariants are gates, not suggestions. A reviewer may `PASS` only when every applicable invariant is satisfied. Do not waive a rule because existing tests omit it. Do not emit a ten-item checklist on success; report only findings or a compact `PASS`.

This bar judges software correctness. Evidence and truthfulness of claims about what was run, observed, staged, deployed, or tested belong to the separate no-unverified-claims/chillpill policy; do not duplicate that policy here.

## Bounded review exploration

Review is **question-driven, not dependency-graph-driven**.

- Start from the task diff and changed behavior.
- Read outside changed paths only to answer a concrete correctness question raised by an applicable invariant.
- One dependency expansion is one question-driven hop from the currently inspected surface to a tightly bounded producer, consumer, or dependency needed to answer that question.
- Prefer the authoritative definition and immediate producer/consumer needed to answer the question. A direct lookup of an already-identified authoritative definition, such as a schema, shared type, config definition, or database constraint, does not consume another dependency expansion.
- Stop exploring as soon as the question is resolved. Do not follow dependencies merely because they exist or recursively audit unrelated callers, dependencies, or nearby code.
- Allow at most two dependency-expansion rounds for one unresolved question. After that, perform only an already-identified authoritative lookup; otherwise `REJECT` with the exact unresolved question or missing authority instead of widening the search again.

# Practical Power of Ten

1. **Ground before changing** — never implement against an assumed API, schema, type, config, state shape, or behavior. Inspect the authoritative definition and the smallest relevant caller, consumer, or test needed to establish the changed contract.

2. **Make the smallest correct change** — change only what the objective requires. Preserve unrelated behavior; no speculative cleanup, opportunistic refactors, duplicate implementations, or new abstractions without necessity.

3. **Validate boundaries and design for realistic failure** — external input, API responses, persisted data, config/env values, parsed data, and other trust boundaries must handle invalid, missing, null, empty, stale, and unexpected values deliberately. Where changed behavior crosses a fallible boundary, identify and deliberately handle material failure modes that can actually arise from that boundary, such as timeout, disconnect or cancellation, unavailable dependency, partial result, retry or duplicate delivery, and resource limits.

4. **Bound open-ended work** — loops, retries, pagination, queues, recursion, network waits, batch sizes, and potentially growing data must have explicit bounds, termination, or backpressure appropriate to the operation. Retries must not become infinite work.

5. **Make state transitions safe** — mutations must remain correct under failure, retries, duplicate requests, and concurrency where those conditions are possible. Use atomicity, idempotency, deduplication, locking, or compensation as appropriate; interrupted work must not leave silently corrupted or contradictory state.

6. **Treat contracts as whole-system changes** — when APIs, schemas, events, shared types, config, persistence formats, or database structure change, inspect the affected authoritative producer/consumer needed to establish compatibility. Preserve compatibility, update coupled sides together, or use a rollout-safe migration when persisted data or mixed-version operation makes compatibility necessary. Do not recursively trace unrelated consumers once the changed contract is established.

7. **Fail gracefully and explicitly** — realistic failures must end in a safe, understandable, recoverable state where recovery is possible. Do not swallow failures, invent silent fallbacks, leave permanent loading or half-complete state, or convert exceptional states into plausible success. Preserve useful diagnostic context without exposing sensitive data.

8. **Prove success and failure at the layer where they can occur** — use the lowest-cost layer that can actually exercise the property being verified. User-observable UI behavior requires UI/browser proof; cross-component behavior requires integration proof; pure deterministic behavior may be proven at unit level. Bug fixes need a regression case when practical. Exercise material realistic failure paths when they can reasonably be induced with the available harness. A correctness-critical failure path that cannot be exercised remains an explicit verification gap, not a passing result. Passing a lower-level happy-path check does not prove a property that exists only at a higher layer or on a failure path.

9. **Enforce security at the authoritative boundary** — authorization, privilege, secret handling, sensitive logging, data exposure, and trust decisions must be enforced server-side or at the actual security boundary, never solely by UI/client convention.

10. **Make consequential changes observable and recoverable** — changes with meaningful production blast radius need a way to determine whether they succeeded and a credible rollback, disable, retry, or recovery path. Existing deployment, monitoring, rollback, retry, or recovery mechanisms satisfy this rule when adequate; do not add new infrastructure solely to satisfy the checklist. Data-destructive operations require recovery consideration before execution.

Apply only rules relevant to behavior caused or changed by the task. A failure mode is realistic only when it follows from an actual boundary, dependency, asynchronous or state transition, resource constraint, documented requirement, or known failure relevant to the changed behavior. Do not invent unrelated fault models; an irrelevant or merely imaginable failure is not a blocker.
