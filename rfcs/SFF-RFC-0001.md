---
id: SFF-RFC-0001
title: Portable Agent Memory Manifest v0
state: Discussion
working_group: WG-01
working_group_slug: memory-for-coding-agents
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a portable, vendor-neutral manifest format that describes how a coding agent stores, retrieves, and exchanges memory across runtime boundaries. The goal is for an agent built on one platform to hand off its working context to an agent on a different platform without loss of meaning.

# Motivation

> TODO: working group to draft.

Today every agent platform invents its own memory representation. Handoff between platforms loses structure, attribution, and expiration semantics. A portable manifest creates the interoperability that the Foundation exists to enable.

## Related Work

The agent-memory space has three distinct traditions, none of which alone is sufficient for cross-vendor portability.

**Commercial agent-memory products** range from file-tree memory tools ([Anthropic Memory](https://docs.anthropic.com/), where the agent owns shape via file ops) through conversation-centric persistence ([OpenAI Assistants threads](https://platform.openai.com/docs/assistants), being sunset Aug 2026) to typed-fact stores ([mem0](https://github.com/mem0ai/mem0), Apache-2.0) and bitemporal knowledge graphs ([Zep / Graphiti](https://github.com/getzep/graphiti), Apache-2.0). The closest existing precedent is [Letta's `.af` (Agent File)](https://github.com/letta-ai/agent-file) — a single-file "agent passport" bundling model config, memory blocks, messages, and tool definitions. Anthropic's recent [memory-import](https://www.anthropic.com/news/memory-import) feature is the first vendor-led portability push, but none of these products defines a portable status / lifecycle field.

**Portable knowledge formats** — [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) (W3C Recommendation) and [RDF-star](https://www.w3.org/groups/wg/rdf-star/) (RDF 1.2) — solve cross-system semantic interchange and stable identity (via `@id` IRIs); [schema.org](https://schema.org/) supplies a mature `creativeWorkStatus` lifecycle enum (`Draft` / `Published` / `Obsolete`) and a `version` property; [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core/) models verbs-as-data (Create / Update / Retract) for event-style memory mutations; [Yjs](https://github.com/yjs/yjs) and [Automerge](https://automerge.org/automerge-binary-format-spec/) CRDTs provide mergeable state but are not wire-compatible with each other.

**LLM-session serialization** in [LangGraph](https://reference.langchain.com/python/langgraph/checkpoints) (channel-versioned snapshots, replayable), [AutoGen](https://microsoft.github.io/autogen/) (a clean split between portable config and opaque runtime state), and the OpenAI / Anthropic message-history conventions show that *per-turn* JSON is portable; the *state* layer is not.

The gap SFF-RFC-0001 fills sits between Letta's bundled-agent-file concept (closest precedent, but Letta-internal) and the W3C semantic-web stack (cross-system but never targeted at agents). The manifest should wrap a `.af`-shaped bundle in a JSON-LD envelope with `@id` IRIs, lift schema.org's lifecycle enum to first-class, and adopt Zep's bitemporality so facts can be retracted without deletion.

# Detailed Design

> TODO: working group to draft. See [TEMPLATE.md](./TEMPLATE.md) for expected sections.

## Proposed envelope (informed by prior art)

The manifest SHOULD be a JSON-LD 1.1 document with stable `@id` IRIs for every memory entity — the same convention used by [schema.org](https://schema.org/), [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core/), and RO-Crate.

**Top-level structure (Letta `.af`-shaped):**

- `@context` — the SFF memory-manifest vocabulary IRI plus any extension contexts.
- `@id` — stable IRI for the manifest itself.
- `agent` — `{ model, provider, version, system_prompt }` (config layer; portable per AutoGen split).
- `memory_blocks[]` — typed entries (`persona`, `world`, `task`, `policy`, custom); each block has `@id`, `@type`, `content`, and the lifecycle / bitemporal fields below.
- `messages[]` — optional conversation history projection (per-turn JSON; the only universally portable LLM-session shape today).
- `tool_definitions[]` — references to declared tools (composes with SFF-RFC-0007).

**Per-block lifecycle and bitemporality (new, derived from synthesis):**

- `status` — enum (`draft` | `active` | `obsolete` | `superseded`) modeled on schema.org `creativeWorkStatus`. None of the commercial products has a comparable field.
- `created` / `updated` — when ingested (system time).
- `valid_at` / `invalid_at` — when the fact became true / ceased to be true (Zep / Graphiti pattern). Enables retraction without deletion.
- `version` — monotonic counter for the entry (LangGraph-style replay determinism).
- `supersedes: [@id]` — typed IRI list, not free-text strings.

**Provenance and confidence (new):**

- Each block MAY carry a `provenance` object: `{ source: @id, derived_from: [@id], confidence: 0..1, attestation? }`. RDF-star quoted-triple metadata is an open option for statement-level annotation (see Open Questions).

## Validation and machine-readability

A JSON Schema (`memory-manifest.schema.json`) MUST be published alongside this RFC and SHOULD be referenced via `$schema` from each manifest. The JSON-LD `@context` SHOULD resolve to a stable IRI published by the Foundation.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[Anthropic Memory](https://docs.anthropic.com/)** (proprietary). File-tree memory the agent edits via tools. No schema. Anthropic's memory-import is the first portability push but defines no lifecycle.
- **[OpenAI Memory / Assistants threads](https://platform.openai.com/docs/assistants)** (proprietary, sunset Aug 2026). Largest install base, no portable export.
- **[mem0](https://github.com/mem0ai/mem0)** (Apache-2.0). Tagged fact records with custom Pydantic export. Schema-flexible but no standard vocabulary.
- **[Zep / Graphiti](https://github.com/getzep/graphiti)** (Apache-2.0). Bitemporal knowledge graph. Adopt bitemporality; graph schema is store-tied.
- **[Letta `.af`](https://github.com/letta-ai/agent-file)** (Apache-2.0). Single-file agent passport — closest existing precedent. SFF-RFC-0001 adopts the bundle shape.
- **[MCP](https://modelcontextprotocol.io)** — focused on tools, not memory; complementary, not overlapping.
- **Vector-store interchange formats.** No standardized format exists; each vendor exports a proprietary blob.

# Prior Art

> TODO: working group to draft.

The table below summarises the surveyed prior art and where each contributes to the v0 manifest.

| Source | Stream | Schema type | Portable across vendors? | Status / lifecycle | Contribution to v0 |
|---|---|---|---|---|---|
| [Anthropic Memory](https://docs.anthropic.com/) | Commercial | File-tree + JSON profile | Partial (import) | No | Memory-import as a vendor-led pattern |
| [OpenAI Memory](https://platform.openai.com/docs/assistants) | Commercial | Threads + KV snippets | No (sunsetting) | No | Conversation-history projection shape |
| [mem0](https://github.com/mem0ai/mem0) | Commercial | Tagged fact records | Yes (custom) | `updated_at` only | Fact-record granularity |
| [Zep / Graphiti](https://github.com/getzep/graphiti) | Commercial | Bitemporal KG | Partial | `valid_at` / `invalid_at` | Bitemporality |
| [Letta `.af`](https://github.com/letta-ai/agent-file) | Commercial | Bundled agent file | Yes (design goal) | No | Top-level bundle shape |
| [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) | Portable | RDF in JSON | Yes (W3C Rec) | Vocab-dependent | Envelope; `@id` identity |
| [RDF-star](https://www.w3.org/groups/wg/rdf-star/) | Portable | Quoted-triple graph | Yes (RDF 1.2) | Vocab-dependent | Statement-level provenance metadata |
| [schema.org](https://schema.org/) | Portable | RDF vocab | Yes | `creativeWorkStatus` | Lifecycle enum, `version` |
| [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core/) | Portable | JSON-LD verbs | Yes (Fediverse) | `published` / `updated` | Verbs-as-data for mutations |
| [Yjs](https://github.com/yjs/yjs) / [Automerge](https://automerge.org/automerge-binary-format-spec/) | Portable | Binary CRDT log | Within-lib only | Vector clocks | Mergeability primitive |
| [LangGraph checkpoints](https://reference.langchain.com/python/langgraph/checkpoints) | LLM session | Snapshot + channels | No | `v`, `ts`, `versions_seen` | Replay determinism |
| [AutoGen](https://microsoft.github.io/autogen/) | LLM session | Config + state dicts | Config yes, state no | No | Config / state split |

# Open Questions

- How are credentials and PII tagged inside a memory manifest?
- What is the minimum schema every implementation MUST support?
- How does this compose with SFF-RFC-0014 (Context Expiration Policy Schema)?
- Mergeability — do we require CRDT semantics like Automerge / Yjs, or accept last-writer-wins JSON-LD patches?
- Confidence / provenance — RDF-star quoted-triple metadata, or an inline `provenance` JSON object on each block?
- Tool / MCP scope — does the manifest carry tool definitions (Letta-style) or only memory (mem0-style)? Commercial precedent is split.

# References

- Anthropic. *Memory Import.* https://www.anthropic.com/news/memory-import
- OpenAI. *Assistants API Deep Dive.* https://developers.openai.com/api/docs/assistants/deep-dive
- mem0ai. *mem0: memory layer for AI applications.* https://github.com/mem0ai/mem0
- Zep AI. *Graphiti.* https://github.com/getzep/graphiti
- Rasmussen, S. et al. *Zep: A Temporal Knowledge Graph Architecture for Agent Memory.* arXiv:2501.13956. https://arxiv.org/abs/2501.13956
- Letta. *agent-file (.af) format.* https://github.com/letta-ai/agent-file
- W3C. *JSON-LD 1.1 (Recommendation).* https://www.w3.org/TR/json-ld11/
- W3C RDF-star Working Group. https://www.w3.org/groups/wg/rdf-star/
- schema.org. *creativeWorkStatus.* https://schema.org/creativeWorkStatus
- W3C. *Activity Streams 2.0 Core.* https://www.w3.org/TR/activitystreams-core/
- Yjs. https://github.com/yjs/yjs
- Automerge Binary Format Specification. https://automerge.org/automerge-binary-format-spec/
- LangChain. *LangGraph Checkpoints.* https://reference.langchain.com/python/langgraph/checkpoints
- Microsoft. *AutoGen — Serializing Components.* https://microsoft.github.io/autogen/stable//user-guide/agentchat-user-guide/serialize-components.html
