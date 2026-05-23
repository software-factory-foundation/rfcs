---
id: SFF-RFC-0014
title: Context Expiration Policy Schema
state: Draft
working_group: WG-01
working_group_slug: memory-for-coding-agents
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines how an agent declares which pieces of its context expire when, why, and what to do when an expiration is reached (drop, summarize, archive, escalate). Without an expiration policy, agent memory grows unbounded and becomes unsafe.

# Motivation

> TODO: working group to draft.

Long-running agents accumulate context indefinitely. Today every team rolls its own ad-hoc retention rule, mixed in with the agent prompt or in a vendor-specific config. A portable expiration policy is what lets memory be reasoned about across platforms.

Composes with SFF-RFC-0001 (Portable Agent Memory Manifest v0).

## Related Work

Six distinct prior-art traditions inform an expiration policy, none individually sufficient.

[**GDPR Art. 5(1)(e)**](https://www.legiscope.com/blog/storage-limitation.html) — the EU storage-limitation principle — frames retention as a *policy obligation* tied to purpose and lawful basis, not a system parameter. Expiration action is delete OR anonymise OR archive under Art. 89(1). Auditability is required (Art. 5(2) accountability).

[**OpenTelemetry SpanProcessor**](https://opentelemetry.io/docs/specs/otel/trace/sdk/) (Apache-2.0, CNCF) cleanly separates *production-side buffering* (`BatchSpanProcessor` with `maxQueueSize` defaulting to 2048, `scheduledDelayMillis` defaulting to 5000, drop-on-overflow) from *consumption-side retention* (backend policy). Two-stage lifecycle.

[**Apache Kafka topic retention**](https://kafka.apache.org/documentation/#topicconfigs) (Apache-2.0) demonstrates composable policies: `retention.ms` (time) *and* `retention.bytes` (size) *and* `cleanup.policy=delete|compact|delete,compact`. **Tombstones (null payload) are first-class delete events** with their own retention (`delete.retention.ms`) — a precedent for separating logical from physical eviction.

[**Vector DB TTL**](https://docs.weaviate.io/weaviate/manage-collections/time-to-live) — Weaviate (1.36+, BSD-3) has three temporal anchors (creation / last-update / DATE-property offset), background hard-delete with optional query-time soft-filtering. Pinecone, Qdrant, and Chroma have no native TTL — operator-driven delete or time-based sharding patterns. Weaviate's split between *logical expiration* (mask from query) and *physical eviction* (background sweep) is unique.

[**MemGPT / Letta memory pressure**](https://www.letta.com/blog/agent-memory) (Apache-2.0; [paper](https://arxiv.org/abs/2310.08560)) is the LLM-native pattern: three tiers (core / recall / archival), occupancy-threshold-triggered eviction, and crucially — *transform, don't drop*. Recursive summarisation of evicted messages with the summary prepended to context; raw messages remain in recall memory; agent self-pages via function calls.

[**Cache eviction policy literature**](https://www.researchgate.net/publication/2568940_ARC_A_Self-Tuning_Low_Overhead_Replacement_Cache) — LRU, LFU, ARC (Megiddo & Modha, FAST'03), W-TinyLFU ([Einziger et al., arXiv:1512.00727](https://arxiv.org/pdf/1512.00727)) — supplies recency / frequency / hybrid-adaptive trigger primitives. ARC's *ghost lists* track evicted-but-remembered keys to self-tune the policy, separating "evicted from cache" from "forgotten entirely."

The gap SFF-RFC-0014 fills: no single source covers everything. The schema needs to support **time, size / count, access-frequency, pressure, and purpose** as composable triggers (Kafka pattern). It needs **drop / summarise / archive / anonymise / escalate / transform-key-only** as a richer action verb set (only the cache literature defaults to pure drop). It needs a **soft / hard** split (Weaviate / Kafka tombstones / ARC ghost lists). And it needs an **audit trail** — caches have none, OTel SDK has none, but GDPR mandates it and agent contexts arguably need it most.

# Detailed Design

> TODO: working group to draft.

## Proposed policy shape (informed by prior art)

**Identity and scope (selector model):**

- `policy_id` (URN).
- `applies_to` — selector matching memory entries: `{ namespace?, type?, tag_match?, attribute_predicate? }`. Weaviate collection-scope is too narrow; Kafka topic-scope is too narrow; GDPR per-purpose is what we want.
- `priority` — when multiple policies match, ordered by priority (open question: longest-match vs explicit-priority).

**Triggers (Kafka composable pattern):**

- `expires_after.time?` — absolute or relative time (`30d`, `2026-06-01T00:00:00Z`).
- `expires_after.size?` — count or bytes threshold.
- `expires_after.last_access?` — LRU-style recency window.
- `expires_after.pressure?` — context-occupancy threshold (MemGPT pattern).
- `expires_after.purpose_complete?` — declarative "lawful basis ended" (GDPR-style).

Triggers compose with AND / OR semantics; the schema defines combinators.

**Action verbs (richer than cache literature, MemGPT-shaped):**

- `action` — `drop` | `summarize` | `archive` | `anonymise` | `escalate` | `transform`.
- `summarizer?` — when `action == summarize`, specifies the summarisation strategy (MemGPT pattern). The summary is itself subject to a policy (see Open Questions).
- `archive_destination?` — when `action == archive`, URI of the cold store.

**Soft / hard split (Weaviate + Kafka tombstone pattern, novel here):**

- `logical_expiration` — applies first; entry becomes invisible to queries.
- `physical_eviction` — separately scheduled; storage reclaimed. Permits replay of recent logical-expirations and is compatible with ghost-list-style adaptive policies.

**PII and special handling:**

- `pii` — flag indicating special-handling requirements; triggers GDPR-style anonymise-or-delete action.

**Audit (GDPR-mandated, novel for agent memory):**

- `audit_log` — append-only entries `{ policy_id, entry_id, trigger, action, timestamp, actor }`. Caches have no audit; agent memory needs one for provenance, reproducibility, incident review, and GDPR Art. 5(2).

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **TTL fields on individual memory entries** (less expressive). Pinecone / Qdrant / Chroma's pattern — agent must compute the policy at write-time. SFF-RFC-0014 is policy-driven so a single policy can govern many entries.
- **Implicit FIFO eviction** (simpler, loses semantics). The default in many in-memory agent runtimes; loses the GDPR audit trail.
- **[Apache Kafka topic retention](https://kafka.apache.org/documentation/#topicconfigs)**. Composable policies (time + size + compaction) is the directly-borrowed pattern.
- **[Weaviate TTL](https://docs.weaviate.io/weaviate/manage-collections/time-to-live)** (BSD-3). Soft / hard split adopted.
- **[MemGPT memory pressure](https://www.letta.com/blog/agent-memory)** (Apache-2.0). Summarise-and-page action verb adopted as `summarize`.
- **[GDPR Art. 5(1)(e)](https://www.legiscope.com/blog/storage-limitation.html)**. Purpose-bound retention + mandatory audit informs the schema.

# Prior Art

> TODO: working group to draft.

| Source | Trigger type | Action on expiry | Per-record / bulk | Audit trail | Contribution |
|---|---|---|---|---|---|
| [GDPR Art. 5(1)(e)](https://www.legiscope.com/blog/storage-limitation.html) | Purpose fulfilled / lawful-basis end | Delete / anonymise / archive (Art. 89) | Per-record (controller-defined) | Required (Art. 5(2)) | Policy-as-obligation; mandatory audit |
| [OTel SpanProcessor](https://opentelemetry.io/docs/specs/otel/trace/sdk/) | Queue full / batch timer | Drop or export-then-forget | Bulk (batch) | None at SDK | Two-stage SDK / backend split |
| [Kafka](https://kafka.apache.org/documentation/#topicconfigs) | Time + size + key-compaction | Segment delete / log compact / tombstone | Bulk (segment); per-key (compact) | Topic-level config history | Composable triggers; tombstones |
| [Weaviate TTL](https://docs.weaviate.io/weaviate/manage-collections/time-to-live) | Created / updated / DATE-property | Background hard delete; query-time mask | Per-object | Background job logs | Soft / hard split |
| [MemGPT / Letta](https://www.letta.com/blog/agent-memory) | Context occupancy threshold | **Summarise + page out** (reversible) | Bulk (oldest N) | Recall preserves originals | Transform-don't-drop |
| [Cache (ARC / W-TinyLFU)](https://arxiv.org/pdf/1512.00727) | Recency + frequency (adaptive) | Drop | Per-entry | Ghost lists (ARC) | Self-tuning trigger |

# Open Questions

- How are policies merged when multiple sources contribute (system + user + agent)? *Recommendation from synthesis: explicit `priority`, longest-match as a tiebreaker.*
- Are summaries themselves subject to a policy? *Recommendation: yes — summaries inherit a child policy from the original, modifiable.*
- Are policies declared at record-level, namespace / collection-level, or both?
- Who owns escalation when expiration would violate a downstream contract?
- Reversibility guarantees — can summarised context be re-hydrated (MemGPT recall-tier pattern)?
- Composition with [SFF-RFC-0001](./SFF-RFC-0001.md): does the memory manifest reference a policy, or vice-versa?

# References

- *GDPR Art. 5(1)(e) — Storage Limitation.* https://www.legiscope.com/blog/storage-limitation.html
- *GDPR Art. 5 Key Principles.* https://www.exabeam.com/explainers/gdpr-compliance/gdpr-article-5-key-principles-and-6-compliance-best-practices/
- OpenTelemetry. *Tracing SDK specification.* https://opentelemetry.io/docs/specs/otel/trace/sdk/
- Apache Kafka. *Topic configurations.* https://kafka.apache.org/documentation/#topicconfigs
- Confluent. *Kafka log compaction.* https://docs.confluent.io/kafka/design/log_compaction.html
- Weaviate. *Time to live.* https://docs.weaviate.io/weaviate/manage-collections/time-to-live
- Qdrant. *Time-based sharding.* https://qdrant.tech/documentation/tutorials-operations/time-based-sharding/
- Letta. *Agent memory.* https://www.letta.com/blog/agent-memory
- Packer, C. et al. *MemGPT.* arXiv:2310.08560. https://arxiv.org/abs/2310.08560
- Megiddo, N. & Modha, D. S. *ARC: A Self-Tuning Low-Overhead Replacement Cache.* FAST'03. https://www.researchgate.net/publication/2568940_ARC_A_Self-Tuning_Low_Overhead_Replacement_Cache
- Einziger, G., Friedman, R., Manes, B. *TinyLFU.* arXiv:1512.00727. https://arxiv.org/pdf/1512.00727
