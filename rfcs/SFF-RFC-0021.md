---
id: SFF-RFC-0021
title: Cross-Agent Tool Invocation
state: Pre-Draft
working_group: WG-07
working_group_slug: coding-agent-integration
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines how one agent invokes a tool that is owned by another agent, including capability handshake, authorization, argument schema, return envelope, and audit propagation. Composes with the Agent Capability Manifest (SFF-RFC-0007).

# Motivation

> TODO: working group to draft.

Today an "agent calling another agent's tool" devolves into bespoke per-pair integrations. A standard invocation envelope makes multi-agent factories composable: any agent can call any tool any other agent advertises in its Capability Manifest.

# Detailed Design

> TODO: working group to draft.

## Proposed invocation envelope

- `invocation_id`, `caller_agent_id`, `target_agent_id`
- `tool_id` — must match a capability advertised in target's Capability Manifest
- `args` — schema-validated against the capability declaration
- `authorization` — signed token authorizing the call
- `audit_chain` — propagated audit IDs for full provenance
- `return` — typed envelope (success / error / partial)

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- MCP transport-level invocation (different layer, different concerns).
- gRPC / OpenAPI as the wire format (possible, but doesn't capture agent-specific concerns like audit propagation).

# Prior Art

> TODO: working group to draft.

# Open Questions

- Is invocation strictly synchronous, or are streaming and async modes in scope?
- How is back-pressure expressed across agent boundaries?
