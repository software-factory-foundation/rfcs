---
id: SFF-RFC-0014
title: Context Expiration Policy Schema
state: Draft
working_group: WG-01
working_group_slug: memory-for-coding-agents
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines how an agent declares which pieces of its context expire when, why, and what to do when an expiration is reached (drop, summarize, archive, escalate). Without an expiration policy, agent memory grows unbounded and becomes unsafe.

# Motivation

> TODO: working group to draft.

Long-running agents accumulate context indefinitely. Today every team rolls its own ad-hoc retention rule, mixed in with the agent prompt or in a vendor-specific config. A portable expiration policy is what lets memory be reasoned about across platforms.

Composes with SFF-RFC-0001 (Portable Agent Memory Manifest v0).

# Detailed Design

> TODO: working group to draft.

## Proposed policy shape

- `policy_id`, `applies_to` — selector matching memory entries
- `expires_after` — time- or event-based trigger
- `action` — `drop` | `summarize` | `archive` | `escalate`
- `summarizer` — optional, specifies how to summarize on action
- `pii` — flag indicating special-handling requirements
- `audit` — required audit event on every expiration

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- TTL fields on individual memory entries (less expressive).
- Implicit FIFO eviction (simpler but loses semantics).

# Prior Art

> TODO: working group to draft.

# Open Questions

- How are policies merged when multiple sources contribute (system policy + user policy + agent policy)?
- Are summaries themselves subject to a policy?
