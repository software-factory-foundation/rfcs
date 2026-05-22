---
id: SFF-RFC-0021
title: Cross-Agent Tool Invocation
state: Pre-Draft
working_group: WG-07
working_group_slug: coding-agent-integration
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines how one agent invokes a tool that is owned by another agent, including capability handshake, authorization, argument schema, return envelope, and audit propagation. Composes with the Agent Capability Manifest (SFF-RFC-0007).

# Motivation

> TODO: working group to draft.

Today an "agent calling another agent's tool" devolves into bespoke per-pair integrations. A standard invocation envelope makes multi-agent factories composable: any agent can call any tool any other agent advertises in its Capability Manifest.

## Related Work

Two complementary streams inform cross-agent invocation: agent-to-agent protocols (the wire shape and capability handshake) and auth / capability patterns (the trust primitives).

**Agent-to-agent protocols.** [Google A2A](https://a2a-protocol.org/latest/specification/) (Apache-2.0, donated to LF in June 2025) is the most directly relevant: JSON-RPC 2.0 over HTTP(S), opaque-agent model (remote internals stay private), `Task` lifecycle (`submitted` → `working` → `input-required` → `completed`/`failed`/`canceled`), and an `AgentCard` capability advertisement. [IBM ACP](https://agentcommunicationprotocol.dev) (merged into A2A under LF in 2025) contributed **capability tokens** as a wire-level trust primitive — signed, unforgeable bearer tokens encoding resource + ops + expiry. [MCP server-to-server patterns](https://modelcontextprotocol.io/docs/learn/server-concepts) (MIT) federate via an "Adapter Hub" server that re-exports sub-server tools, but ship no native delegation token — trust is per-server. [Magentic-One / AutoGen](https://microsoft.github.io/autogen) is an *orchestration* pattern (centralized authority, agents-as-tools) rather than a federation protocol — useful as the orchestrator-side reference but not the wire spec.

**Auth and capability patterns.** [OAuth 2.0 Token Exchange (RFC 8693)](https://datatracker.ietf.org/doc/html/rfc8693) is the IETF standards-track answer — `subject_token` + optional `actor_token`, with a nestable `act` claim cryptographically recording the delegation chain in the token itself, plus a `may_act` claim authorising delegation. [SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/) (Apache-2.0, CNCF graduated) supplies *attested workload identity*: `spiffe://<trust-domain>/<workload-path>` URIs and short-lived (~1h) SVIDs delivered via local Workload API — no pre-shared secrets. The [object-capability model](http://erights.org/talks/thesis/markm-thesis.pdf) (Miller, 2006) provides the theoretical primitive: unforgeable references = authority; designation equals authorisation; no ambient authority. [Macaroons](https://research.google/pubs/pub41892) (Google Research) and [Biscuit](https://www.biscuitsec.org) (Eclipse Foundation, Apache-2.0) implement *offline attenuation* — the holder can mint strictly-weaker child tokens without an STS round-trip; Macaroons via HMAC-chained caveats, Biscuit via public-key-signed Datalog blocks. [UCAN](https://ucan.xyz/specification) (Apache-2.0 / MIT) is fully decentralized — principals are DIDs, attenuations are an `att` array, and `prf` is the proof chain of parent UCANs. [Istio / Linkerd](https://istio.io/) (Apache-2.0) enforces mTLS + AuthorizationPolicy in the sidecar — caller identity, not chain.

The gap SFF-RFC-0021 fills: **no current agent-to-agent protocol ships a native delegation chain**. A2A's task IDs correlate calls but don't trace delegation. ACP capability tokens are the closest, but lived inside ACP; with ACP merged into A2A, the delegation primitive is now an open question. The RFC's contribution is to pick *one* delegation primitive (OAuth 8693 `act` claim, Macaroons / Biscuit caveat chain, or UCAN `prf` chain) and define how it rides on the A2A / MCP wire.

# Detailed Design

> TODO: working group to draft.

## Proposed invocation envelope (informed by prior art)

The envelope rides on the A2A JSON-RPC 2.0 wire (with MCP server-to-server as the alternative binding); tool schemas use the SFF-RFC-0007 Agent Capability Manifest definitions.

**Invocation identity:**

- `invocation_id` (UUID).
- `caller_agent_id`, `target_agent_id` — references to Agent Capability Manifests.
- `tool_id` — MUST match a capability advertised in target's Capability Manifest.
- `parent_invocation_id?` — for nested invocations.

**Schema-validated arguments:**

- `args` — validated against the capability's `input_schema` (JSON Schema 2020-12). Validation MUST occur at both caller and target trust boundaries.
- `args_schema_url` — explicit schema reference so version-skew is detectable.

**Authorization (the central design choice):**

- `authorization.token` — signed token authorising the call. The RFC adopts **OAuth 2.0 Token Exchange semantics** (`act` claim nesting) as the recommended path with optional Macaroons / Biscuit / UCAN bindings (see Open Questions).
- `authorization.identity_attestation?` — optional [SPIFFE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/) SVID providing attested workload identity (caller-side).
- `authorization.exp` — REQUIRED. Bearer reuse is rejected.

**Audit chain propagation (novel — RFC's distinct contribution):**

- `audit_chain[]` — ordered list of prior delegators, each `{ agent_id, invocation_id, timestamp, signature }`. Built from the OAuth `act` chain (or caveat / `prf` chain). Every hop appends; never edits prior entries.

**Return envelope:**

- `return` — `{ kind: success | error | partial, payload?, error? }`. Composes with [SFF-RFC-0006](./SFF-RFC-0006.md) (UTRE) `findings[]` when invocation surfaces test-style results.
- `return.audit_chain` — extended with the target's signature on completion (closes the chain).

## Streaming and async

- Synchronous invocation is the default. Streaming (results arrive in chunks before completion) is OPTIONAL — modeled as a sequence of partial-return envelopes sharing one `invocation_id`. Async (caller does not block) uses the [A2A](https://a2a-protocol.org/latest/specification/) `Task` lifecycle.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[A2A](https://a2a-protocol.org/latest/specification/)** (Apache-2.0, LF). Closest existing wire-shape — adopted as the recommended binding.
- **[ACP](https://agentcommunicationprotocol.dev)** (Apache-2.0, merged into A2A). Capability tokens primitive — informed `authorization.token`.
- **[MCP server-to-server](https://modelcontextprotocol.io/docs/learn/server-concepts)** (MIT). Adapter Hub federation; alternative binding for MCP-native ecosystems.
- **[Magentic-One](https://microsoft.github.io/autogen)** (MIT). Orchestrator-led pattern; reference for the orchestrator side, not the wire.
- **[OAuth 2.0 Token Exchange (RFC 8693)](https://datatracker.ietf.org/doc/html/rfc8693)** (IETF). Recommended delegation primitive — `act` chain.
- **[SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/)** (Apache-2.0, CNCF). Optional attested-identity layer.
- **Object-capability model** ([Miller, 2006](http://erights.org/talks/thesis/markm-thesis.pdf)). Theoretical backbone.
- **[Macaroons](https://research.google/pubs/pub41892)**. Offline attenuation; alternative for ecosystems without an STS.
- **[Biscuit](https://www.biscuitsec.org)** (Apache-2.0). Datalog-based authorisation; alternative for richer policy.
- **[UCAN](https://ucan.xyz/specification)** (Apache-2.0 / MIT). Decentralized DID-based; alternative for federated trust.
- **[Istio Authorization](https://istio.io/latest/docs/concepts/security/)** (Apache-2.0). Sidecar-enforced — no in-band delegation chain; complementary at transport layer.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Capability declaration | Delegation chain | Audit log | Time-bounded? | Contribution |
|---|---|---|---|---|---|---|
| [A2A](https://a2a-protocol.org/latest/specification/) | A2A | AgentCard JSON | No | Task IDs | Per-task | Wire shape (JSON-RPC) |
| [ACP](https://agentcommunicationprotocol.dev) | A2A | Agent manifest | No | Run IDs | Per-run | Capability tokens primitive |
| [MCP s2s](https://modelcontextprotocol.io/docs/learn/server-concepts) | A2A | tools / list schemas | No | Host-app side | Session | Federation via proxy servers |
| [Magentic-One](https://microsoft.github.io/autogen) | A2A | Hard-coded agent roster | N/A (in-process) | Task + progress ledgers | N/A | Orchestrator centralizes authority |
| [RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693) | Auth | Scopes / claims | Yes (nested `act`) | STS issuance log | Yes (`exp`) | Standards-track delegation |
| [SPIFFE / SPIRE](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/) | Auth | SPIFFE ID URI | No (identity only) | Attestation logs | Yes (~1h SVID) | Platform-attested identity |
| [Macaroons](https://research.google/pubs/pub41892) | Auth | Caveats | Yes (HMAC chain) | Caveat chain | Yes (caveat) | Offline attenuation |
| [Biscuit](https://www.biscuitsec.org) | Auth | Datalog facts | Yes (signed blocks) | Block chain | Yes (Datalog check) | Logic-language authz |
| [UCAN](https://ucan.xyz/specification) | Auth | `att` array | Yes (`prf` chain) | Proof chain | Yes (`nbf`/`exp`) | DID-based, decentralized |
| [Istio AuthZ](https://istio.io/latest/docs/concepts/security/) | Auth | AuthorizationPolicy | No | Sidecar access log | Cert lifetime | Transparent sidecar enforcement |

# Open Questions

- Is invocation strictly synchronous, or are streaming and async modes in scope? *Recommendation from synthesis: synchronous is default, streaming via partial-returns, async via A2A `Task`.*
- How is back-pressure expressed across agent boundaries? *Open — likely via A2A `Task` `input-required` plus rate-limit headers.*
- Which delegation primitive — OAuth 8693 `act`, Macaroons / Biscuit caveat chain, or UCAN `prf`? *Recommendation: OAuth 8693 for enterprise-aligned deployments, Macaroons / Biscuit for offline-attenuation needs, UCAN for federated decentralized; specify all three bindings.*
- Revocation in offline-attenuated tokens (Macaroons / UCAN both weak here)?
- Schema-version negotiation when caller and callee disagree on a tool's JSON Schema?
- How do audit chains survive when a sub-agent re-delegates to an external SaaS tool outside the trust domain?
- Composition with [SFF-RFC-0007](./SFF-RFC-0007.md): does the caller validate against the latest Capability Manifest at invocation time, or pin to a snapshot?

# References

- a2aproject. *A2A Protocol Specification.* https://a2a-protocol.org/latest/specification/
- IBM. *Agent Communication Protocol.* https://agentcommunicationprotocol.dev
- Anthropic. *Model Context Protocol — Server Concepts.* https://modelcontextprotocol.io/docs/learn/server-concepts
- Microsoft Research. *Magentic-One.* https://arxiv.org/abs/2411.04468
- IETF. *RFC 8693 — OAuth 2.0 Token Exchange.* https://datatracker.ietf.org/doc/html/rfc8693
- SPIFFE. *Concepts.* https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/
- Birgisson, A. et al. *Macaroons: Cookies with Contextual Caveats.* https://research.google/pubs/pub41892
- Eclipse Foundation. *Biscuit specifications.* https://github.com/eclipse-biscuit/biscuit/blob/main/SPECIFICATIONS.md
- UCAN. *Specification.* https://ucan.xyz/specification
- Miller, M.S. *Robust Composition.* http://erights.org/talks/thesis/markm-thesis.pdf
- Istio. *Authentication Policy.* https://istio.io/latest/docs/tasks/security/authentication/authn-policy/
