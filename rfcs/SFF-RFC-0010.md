---
id: SFF-RFC-0010
title: Factory Evaluation Benchmarks v0
state: Pre-Draft
working_group: WG-10
working_group_slug: continual-improvement
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a starter set of benchmarks for evaluating software factories on completeness, correctness, speed, cost, and safety. Benchmarks are reproducible, public, and version-pinned so claims of "our factory is better at X" can be compared apples-to-apples.

# Motivation

> TODO: working group to draft.

Every factory vendor publishes self-flattering numbers against their own private rubric. The result is that buyers and members can't compare. A small, public, reproducible benchmark suite — owned by the Foundation rather than any one vendor — gives the field a shared frame of reference.

# Detailed Design

> TODO: working group to draft.

## Proposed benchmark axes

- **Completeness** — what fraction of a representative task set the factory can complete unaided.
- **Correctness** — quality of completions on tasks it claims to handle.
- **Speed** — wall-clock and human-minutes to complete.
- **Cost** — measurable resource cost per completed task.
- **Safety** — rate of escapes (data loss, prod-impacting actions without approval).

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- SWE-bench, HumanEval, METR — existing public benchmarks the Foundation might adopt rather than recreate.

# Prior Art

> TODO: working group to draft.

# Open Questions

- Should the Foundation host an official leaderboard, or only the benchmark spec?
- How are results audited? Self-attestation vs third-party run?
