---
id: SFF-RFC-0007
title: Agent Capability Manifest v0
state: Discussion
working_group: WG-07
working_group_slug: coding-agent-integration
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines how a coding agent declares its capabilities, tool requirements, supported protocols, and trust boundaries to an orchestrator. Capability negotiation is mandatory; static configuration is OPTIONAL.

# Motivation

> TODO: working group to draft.

Orchestrators today either trust agent self-reports informally or hardcode allowlists per vendor. A capability manifest formalizes what an agent can and cannot do, so the orchestrator can route work, set quotas, and enforce trust boundaries from a single document.

# Detailed Design

> TODO: working group to draft.

## Proposed manifest fields

- `agent_id`, `agent_version`, `agent_kind`
- `capabilities[]` — declared capabilities (read_code, write_code, run_tests, …)
- `tools[]` — tools the agent requires access to
- `protocols[]` — supported communication protocols (MCP, A2A, custom)
- `trust` — declared trust level and required boundaries
- `attestation` — signed by the agent vendor

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- MCP server manifests.
- A2A capability advertisement.

# Prior Art

> TODO: working group to draft.

# Open Questions

- How are capabilities versioned?
- How does this compose with SFF-RFC-0021 (Cross-Agent Tool Invocation)?
