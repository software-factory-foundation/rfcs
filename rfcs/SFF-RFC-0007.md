---
id: SFF-RFC-0007
title: Agent Capability Manifest v0
state: Discussion
working_group: WG-07
working_group_slug: coding-agent-integration
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines how a coding agent declares its capabilities, tool requirements, supported protocols, and trust boundaries to an orchestrator. Capability negotiation is mandatory; static configuration is OPTIONAL.

# Motivation

> TODO: working group to draft.

Orchestrators today either trust agent self-reports informally or hardcode allowlists per vendor. A capability manifest formalizes what an agent can and cannot do, so the orchestrator can route work, set quotas, and enforce trust boundaries from a single document.

## Related Work

The capability-manifest space spans three traditions, none of which alone provides both the declaration shape and the trust primitives the RFC needs.

**Tool-definition schemas** describe individual tools an agent exposes. [Model Context Protocol (MCP)](https://modelcontextprotocol.io/specification/2025-11-25) (Anthropic, MIT) introduces a JSON-RPC handshake with LSP-inspired symmetric capability negotiation during `initialize`. [OpenAI Responses API](https://developers.openai.com/api/docs/guides/function-calling) contributes strict-mode schemas, namespaced tools, and deferred tool loading. [Anthropic tool_use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use) pins to [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) as a clean minimal parameter dialect. [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0.html) (Apache-2.0, Linux Foundation) supplies HTTP-semantic operations with built-in `securitySchemes` (OAuth2, OIDC, apiKey, mTLS) and is the bridge most agent platforms use to convert HTTP APIs into tools.

**Agent-to-agent protocols** describe how agents *discover and negotiate* with each other. [Google A2A](https://a2a-protocol.org/latest/specification/) (Apache-2.0, donated to the Linux Foundation in June 2025) is the closest sibling to this RFC — its `AgentCard` carries `AgentCapabilities`, `AgentSkill[]`, `AgentInterface[]` (JSON-RPC/gRPC/HTTP bindings), `AgentExtension[]`, `AgentProvider`, `securitySchemes`, and an `AgentCardSignature` for verifiable publication. [IBM ACP](https://agentcommunicationprotocol.dev) introduces **capability tokens** as a wire-level trust primitive. [Magentic-One / AutoGen](https://microsoft.github.io/autogen) shows an orchestrator-led re-planning pattern but declares capabilities only implicitly via an agent class registry.

**Trust and capability theory** anchors the unforgeability of these declarations. Mark Miller's [object-capability model](http://erights.org/talks/thesis/markm-thesis.pdf) (2006) frames authority as unforgeable references — designation equals authorisation, no ambient authority. [Caja](https://github.com/googlearchive/caja) (archived 2021) showed that retrofitting object-capability semantics onto an existing language is brittle (sanitisation vulns drove archival). [SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/) (CNCF graduated) supplies attested, rotating workload identity with no pre-shared secrets — `spiffe://<trust-domain>/<workload-path>` plus short-lived SVIDs. Capability-based OSes ([KeyKOS, EROS, seL4](https://trustworthy.systems/projects/seL4)) set the formal bar for what "enforced at scale" means; seL4 has machine-checked proofs that the capability model upholds integrity, confidentiality, and availability.

The gap SFF-RFC-0007 fills is that **no current tool-definition or agent-protocol schema has a real identity / trust primitive**. A2A's signed AgentCard is the closest, but only describes *discovery*. The RFC's contribution is to weave a tool-definition schema (JSON Schema 2020-12, MCP-style), a discovery shape (A2A AgentCard-shaped), and a trust block (SPIFFE ID and/or ACP-style capability tokens) into a single signed manifest.

# Detailed Design

> TODO: working group to draft.

## Proposed manifest fields (informed by prior art)

The manifest SHOULD be JSON-shaped, with [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) for parameter types and a stable `$schema` URL.

**Identity (A2A-shaped + SPIFFE):**

- `agent_id` — globally unique identifier (URI).
- `agent_version`, `agent_kind`.
- `provider` — `{ name, homepage, contact }` (A2A `AgentProvider`).
- `spiffe_id?` — optional `spiffe://<trust-domain>/<path>` for attested cross-org identity.

**Capabilities and tools (MCP + OpenAPI patterns):**

- `capabilities[]` — declared capabilities, each `{ id, version, description, declared_skills[] }` (A2A `AgentCapabilities` + `AgentSkill`).
- `tools[]` — per-tool definition: `{ name, description, input_schema (JSON Schema 2020-12), output_schema?, namespace?, defer_loading? }` (MCP + OpenAI strict-mode + Anthropic patterns).
- `interfaces[]` — supported wire protocols and bindings: `[{ protocol: "mcp"|"a2a"|"http"|"grpc", endpoint?, version }]`.

**Capability negotiation (MCP-symmetric):**

- The manifest is the *static* declaration; runtime negotiation MUST follow MCP's `initialize` handshake pattern where client and server each advertise feature sets and only mutually-supported features may be used.

**Trust block (novel — RFC's distinctive contribution):**

- `trust` — `{ level: declared_level, boundaries[], capability_tokens?, identity_refs[] }`. Capability tokens follow the ACP / OAuth Token-Exchange pattern (signed, time-bounded, scope-restricted).
- `attestation` — `{ signature (DSSE / JWS), key_id, signing_method, signed_at }` — A2A `AgentCardSignature` pattern.

## Discovery

The manifest SHOULD be published at `/.well-known/sff-agent-capability-manifest.json` for federation by orchestrators across vendors.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[MCP](https://modelcontextprotocol.io/specification/2025-11-25)** (MIT). De-facto agent-tool standard; the v0 manifest layers on top of it rather than replacing the handshake.
- **[Google A2A](https://a2a-protocol.org/latest/specification/)** (Apache-2.0, LF). Closest sibling artefact — signed AgentCard. SFF-RFC-0007 borrows its envelope shape and signature pattern.
- **[IBM ACP](https://agentcommunicationprotocol.dev)** (Apache-2.0). Capability tokens on the wire — adopted as the in-token trust primitive option.
- **[OpenAI Responses API](https://developers.openai.com/api/docs/guides/function-calling)** (proprietary; schema conventions public). Strict-mode + namespace + deferred-loading patterns.
- **[Anthropic tool_use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use)** (proprietary). Minimal clean shape; pin to JSON Schema 2020-12.
- **[OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0.html)** (Apache-2.0). Strong for HTTP API surfaces with `securitySchemes` — useful as a binding profile.
- **[SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/)** (Apache-2.0, CNCF). Drop-in attested workload identity layer.
- **Object-capability theory** ([Miller 2006](http://erights.org/talks/thesis/markm-thesis.pdf)). Theoretical backbone; favour first-class capability tokens over sanitisation (Caja cautionary).

# Prior Art

> TODO: working group to draft.

| Source | Stream | Capability declaration | Trust / identity fields | Negotiation support | Contribution to v0 |
|---|---|---|---|---|---|
| [MCP](https://modelcontextprotocol.io/specification/2025-11-25) | Tool schema | Yes (tools / resources / prompts) | Consent-based, no identity | Yes (symmetric `initialize`) | De-facto standard; handshake pattern |
| [OpenAI Responses](https://developers.openai.com/api/docs/guides/function-calling) | Tool schema | Yes (function / namespace / defer) | No | No wire negotiation | Strict-mode schema conformance |
| [Anthropic tool_use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use) | Tool schema | Yes (`input_schema`) | No | No | Pin to JSON Schema 2020-12 |
| [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0.html) | Tool schema | Yes (Operation) | Yes (`securitySchemes`) | Static (spec-time) | HTTP-semantic with auth |
| [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) | Tool schema | n/a (substrate) | n/a | n/a | Vocabularies for extension |
| [A2A](https://a2a-protocol.org/latest/specification/) | Agent protocol | Yes (AgentCard + skills + interfaces) | Yes (signatures + securitySchemes) | Discovery-time | Closest sibling envelope |
| [IBM ACP](https://agentcommunicationprotocol.dev) | Agent protocol | Yes (short manifest) | Yes (capability tokens) | Yes | Capability tokens on wire |
| [Magentic-One](https://microsoft.github.io/autogen) | Agent protocol | Implicit (class registry) | No | Orchestrator-internal | Orchestrator pattern |
| [Miller, Robust Composition](http://erights.org/talks/thesis/markm-thesis.pdf) | Trust theory | Theoretical | Unforgeable references | Designation = authorisation | Foundational ocap |
| [Caja](https://github.com/googlearchive/caja) | Trust theory | Sanitiser-mediated | Object-capability JS | n/a | Retrofitting failure mode |
| [SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/) | Trust theory | n/a | Yes (SPIFFE ID + SVID) | Federation bundles | Attested identity |
| [seL4](https://trustworthy.systems/projects/seL4) | Trust theory | Kernel cap tables | Yes | n/a | Formally verified ocap |

# Open Questions

- How are capabilities versioned? *Recommendation: SemVer at the manifest level, plus `version` on each capability entry.*
- How does this compose with SFF-RFC-0021 (Cross-Agent Tool Invocation)? *The manifest is the static surface; SFF-RFC-0021 is the invocation envelope; the `interfaces[]` and `tools[]` fields are the bridge.*
- Manifest signing — JWS over canonical JSON (A2A pattern), or COSE / Sigstore?
- Capability revocation — TTL-only (ACP capability tokens), or revocation lists?
- How to express *negative* capabilities (sandbox boundaries, "this agent MUST NOT call X")? Object-capability theory prefers enumeration of positive authority; agent ergonomics may force a hybrid.
- Should the `trust` block require either SPIFFE-style attested identity *or* capability tokens, or both?

# References

- Anthropic. *Model Context Protocol Specification 2025-11-25.* https://modelcontextprotocol.io/specification/2025-11-25
- OpenAI. *Function Calling Guide.* https://developers.openai.com/api/docs/guides/function-calling
- Anthropic. *Define and implement tools.* https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use
- OpenAPI Initiative. *OpenAPI Specification v3.1.0.* https://spec.openapis.org/oas/v3.1.0.html
- JSON Schema. *Draft 2020-12.* https://json-schema.org/draft/2020-12
- a2aproject. *A2A Protocol Specification.* https://a2a-protocol.org/latest/specification/
- Linux Foundation. *Launches Agent2Agent Protocol Project.* https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents
- IBM. *Agent Communication Protocol.* https://agentcommunicationprotocol.dev
- Microsoft Research. *Magentic-One.* https://arxiv.org/abs/2411.04468
- Miller, M.S. *Robust Composition.* https://erights.org/talks/thesis/markm-thesis.pdf
- Google. *Caja (archived).* https://github.com/googlearchive/caja
- SPIFFE. *Concepts.* https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/
- Trustworthy Systems. *seL4.* https://trustworthy.systems/projects/seL4
